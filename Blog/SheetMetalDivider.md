# Sheetmetal divider

Sometimes a sheet metal part is longer than the stock sheets you have on hand. The fix is to split the flat pattern into multiple pieces, but each cut line has to miss the holes and cutouts in the part. This rule does that automatically. Given a list of available sheet sizes, it finds the combination of cuts with the least waste while keeping every cut clear of the cutouts.

The chosen lengths are written to the parameters `S1`, `S2`, `S3` and `S4` (one length per sheet, in millimetres), plus a total `WastValue`. I drive a layout sketch on the part with these parameters so the cut positions are visible on the flat pattern view.

> 💡 Create `S1`, `S2`, `S3`, `S4` and `WastValue` in the part before running the rule. Set any unused ones to `1` so they stay out of the way in the layout sketch.

## Inputs

At the top of `Main` there are two settings:

- `MinDistanceFromGaps` is a safety margin in cm around every cutout. A cut line will never land closer than this distance to a hole.
- `SheetSizes` is the list of stock sheet lengths you have on hand, in cm. Add as many sizes as you need; the rule explores every combination.

In the example only one size is active (300 cm). Uncomment the others to let the algorithm mix multiple stock lengths in the same part.

## Finding the no-cut zones

The flat pattern's top face has one outer edge loop (the part outline) plus one inner edge loop for each hole or cutout. The rule iterates over the inner loops and turns each one into a `Gap`, which is just an X-range where cuts are forbidden. The gap is the cutout's own X-range expanded by `MinDistanceFromGaps` on each side.

Two extra gaps are added at the very ends of the flat pattern, 25 cm deep on each side. These keep the algorithm from cutting off a thin sliver of material near the start or end. If multiple cutouts produce overlapping gaps, `MergeGaps` joins them, so the algorithm sees one continuous forbidden zone instead of several small ones.

## Trying every split

`GetSheets` is recursive. From the current X position it tries each available sheet size in turn, advancing the cut position and recursing into the remaining length. When a cut reaches or passes the end of the flat pattern, the recursion stops and the current combination of sheet sizes is stored in `Posbilties`.

If a proposed cut lands inside a gap, the sheet is shortened so the cut sits right before the gap, rounded down to the nearest 10 cm for a clean dimension. The difference between the nominal sheet size and this shorter "actual" size is added to a waste counter for that combination. The check repeats in case the shortened cut still lands in another gap further left.

The result is a list of every valid way to divide the part with the available sheet sizes, each tagged with how much material it wastes.

## Picking the winner

After the recursion the rule keeps only the combinations with the lowest `Wast` value. If several combinations tie, the one with the fewest sheets wins, because fewer sheets means fewer cuts and less handling on the shop floor.

The chosen sizes are written to `S1` through `S4` (multiplied by 10 to convert cm to mm), and the total waste goes to `WastValue`.

> ⚠️ The output is limited to four sheets. If your flat pattern is so long that it needs more than four pieces, extend the `SetParameter` calls in `Main` and add matching parameters in the part.

## The full rule

```vb.net
Public Class ThisRule

    Sub Main()
        MinDistanceFromGaps = 2.5 ' in Cm
        SheetSizes = New List(Of Double)
        SheetSizes.Add(300) ' in Cm
        'SheetSizes.Add(250) ' in Cm
        'SheetSizes.Add(125) ' in Cm

        Dim doc As PartDocument = ThisDoc.Document
        Dim def As SheetMetalComponentDefinition = doc.ComponentDefinition
        Dim flat = def.FlatPattern
        Dim top = flat.TopFace

        Dim flatMinX = top.Evaluator.RangeBox.MinPoint.X
        Dim flatMaxX = top.Evaluator.RangeBox.MaxPoint.X

        Gaps = New List(Of Gap)
        Gaps.Add(New Gap(flatMinX, flatMinX + 25))
        Gaps.Add(New Gap(flatMaxX - 25, flatMaxX))
        For Each edgeLoop As EdgeLoop In top.EdgeLoops

            If (edgeLoop.IsOuterEdgeLoop) Then Continue For

            Dim newGap As New Gap(edgeLoop.RangeBox.MinPoint.X - MinDistanceFromGaps, edgeLoop.RangeBox.MaxPoint.X + MinDistanceFromGaps)

            Dim isMerged = Gaps.Where(Function(g) g.TryMergGap(newGap))

            If (isMerged.Count = 0) Then
                Gaps.Add(newGap)
            End If
        Next

        Gaps = MergeGaps(Gaps)

        GetSheets(flatMinX, flatMaxX, New FlatPatternSet)

        Dim leftOverPosbilties As New List(Of FlatPatternSet)
        Dim flatLength = flatMaxX - flatMinX
        Dim minWast = Double.MaxValue
        For Each curentSet As FlatPatternSet In Posbilties

            If (curentSet.Wast < minWast) Then
                leftOverPosbilties = New List(Of FlatPatternSet)
                leftOverPosbilties.Add(curentSet)
                minWast = curentSet.Wast
            ElseIf (Math.Abs(curentSet.Wast - minWast) < 0.01) Then
                leftOverPosbilties.Add(curentSet)
            End If
        Next

        Dim minCount = leftOverPosbilties.Min(Function(p) p.Sizes.Count)
        Dim useSheets = leftOverPosbilties.First(Function(p) p.Sizes.Count = minCount)


        SetParameter("S1", useSheets.Sizes, 0)
        SetParameter("S2", useSheets.Sizes, 1)
        SetParameter("S3", useSheets.Sizes, 2)
        SetParameter("S4", useSheets.Sizes, 3)
        Parameter("WastValue") = useSheets.Wast * 10
        ThisDoc.Document.Update()
    End Sub

    Private Property SheetSizes As List(Of Double)
    Private Property Gaps As List(Of Gap)
    Private Property MinDistanceFromGaps As Double
    Public Property Posbilties As New List(Of FlatPatternSet)

    Public Sub SetParameter(name As String, sheets As List(Of Double), value As Integer)
        Try
            Parameter(name) = sheets(value) * 10
        Catch ex As Exception
            Parameter(name) = 1
        End Try
    End Sub

    Public Sub GetSheets(min As Double, max As Double, curentSet As FlatPatternSet)

        For Each sheetSize As Double In SheetSizes
            Dim cutDistance = min + sheetSize
            Dim actualSheetSize = sheetSize
            Dim newSet = curentSet.Clone()

            Dim problamaticGap = Gaps.FirstOrDefault(Function(g) g.InGap(cutDistance))
            While (problamaticGap IsNot Nothing)
                actualSheetSize = problamaticGap.Min - min
                actualSheetSize = Math.Floor(actualSheetSize / 10) * 10
                cutDistance = min + actualSheetSize
                problamaticGap = Gaps.FirstOrDefault(Function(g) g.InGap(cutDistance))
            End While
            newSet.Wast += sheetSize - actualSheetSize

            newSet.Sizes.Add(actualSheetSize)
            If (max <= cutDistance) Then
                newSet.Wast += cutDistance - max
                Posbilties.Add(newSet)
            Else
                GetSheets(min + actualSheetSize, max, newSet)
            End If
        Next
    End Sub


    Public Function MergeGaps(gaps As List(Of Gap)) As List(Of Gap)
        If gaps Is Nothing OrElse gaps.Count = 0 Then Return gaps

        ' sort first
        Dim ordered = gaps.OrderBy(Function(g) g.Min).ToList()

        Dim result As New List(Of Gap)
        Dim current = New Gap(ordered(0).Min, ordered(0).Max)

        For i = 1 To ordered.Count - 1
            Dim nextGap = ordered(i)

            If current.TryMergGap(nextGap) Then
                ' already merged into current
            Else
                result.Add(current)
                current = New Gap(nextGap.Min, nextGap.Max)
            End If
        Next

        result.Add(current)

        Return result
    End Function


    Public Class FlatPatternSet

        Public Property Sizes As List(Of Double) = New List(Of Double)
        Public Property Wast As Double = 0.0

        Public Function Clone() As FlatPatternSet
            Dim flatSet As New FlatPatternSet
            flatSet.Sizes.AddRange(Sizes)
            flatSet.Wast = Wast
            Return flatSet
        End Function

    End Class

    Public Class Gap
        Private _floatingPointTollerance As Double = 0.01
        Public Sub New(minValue As Double, maxValue As Double)
            Min = minValue
            Max = maxValue
        End Sub
        Public Property Min As Double
        Public Property Max As Double

        Public Function InGap(value As Double) As Boolean
            Return (Min + _floatingPointTollerance < value And value < Max - _floatingPointTollerance)
        End Function

        Public Function TryMergGap(other As Gap) As Boolean
            If other.Min <= Max AndAlso other.Max >= Min Then
                Min = Math.Min(Min, other.Min)
                Max = Math.Max(Max, other.Max)
                Return True
            End If
            Return False
        End Function
    End Class

End Class
```

