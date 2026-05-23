# Auto balloon

```vb.net
Public Class ThisRule

    Private Enum Edge
        Top
        Bottom
        Left
        Right
    End Enum

    Sub Main()
        Dim doc As DrawingDocument = ThisDoc.Document
        Dim sheet As Sheet = doc.ActiveSheet
        Dim view As DrawingView = sheet.DrawingViews.Item(1)

        Dim assemblyDocument As AssemblyDocument = view.ReferencedDocumentDescriptor.ReferencedDocument
        Dim assemblyFileName = assemblyDocument.FullFileName

        Dim tagCurves = CollectLargestCurvePerFile(view)
        PlaceBalloons(doc, sheet, view, tagCurves)
    End Sub

    Private Function CollectLargestCurvePerFile(view As DrawingView) As List(Of TagCurve)
        Dim byFile As New Dictionary(Of String, TagCurve)

        For Each curve As DrawingCurve In view.DrawingCurves.Cast(Of DrawingCurve)()
            'This only works with edges
            If TypeOf curve.ModelGeometry IsNot EdgeProxy Then Continue For

            Dim tagCurve As New TagCurve(curve)
            If tagCurve.Occ.OccurrencePath.Count > 2 Then Continue For

            Dim existing As TagCurve = Nothing
            If Not byFile.TryGetValue(tagCurve.FileName, existing) OrElse tagCurve.SquaredSize > existing.SquaredSize Then
                byFile(tagCurve.FileName) = tagCurve
            End If
        Next

        Return byFile.Values.ToList()
    End Function

    Private Sub PlaceBalloons(doc As DrawingDocument, sheet As Sheet, view As DrawingView, tagCurves As List(Of TagCurve))
        Dim transientGeom = ThisApplication.TransientGeometry
        Dim balloonDiameter = doc.StylesManager.ActiveStandardStyle.ActiveObjectDefaults.BalloonStyle.BalloonDiameter
        Dim minimalDistanceBetweenBaloons = balloonDiameter

        Dim viewTop = view.Top
        Dim viewBottom = view.Top - view.Height
        Dim viewLeft = view.Left
        Dim viewRight = view.Left + view.Width

        ' Tracks already-placed balloon centers along each view edge so we can
        ' nudge new balloons sideways and avoid overlap. Top/Bottom store X
        ' coordinates; Left/Right store Y coordinates.
        Dim placed As New Dictionary(Of Edge, List(Of Double)) From {
            {Edge.Top, New List(Of Double)},
            {Edge.Bottom, New List(Of Double)},
            {Edge.Left, New List(Of Double)},
            {Edge.Right, New List(Of Double)}
        }

        For Each tagCurve As TagCurve In tagCurves
            ' Closed curves (circles, ellipses) have no midpoint — fall back to the center.
            Dim midPoint As Point2d = tagCurve.Curve.MidPoint
            If midPoint Is Nothing Then midPoint = tagCurve.Curve.CenterPoint

            Dim chosenEdge = ChooseClosestEdge(midPoint, viewTop, viewBottom, viewLeft, viewRight)
            Dim leader = ComputeLeaderPoint(chosenEdge, midPoint, viewTop, viewBottom, viewLeft, viewRight, balloonDiameter)

            Dim alongEdge = placed(chosenEdge)
            Dim leaderX = leader.X
            Dim leaderY = leader.Y
            If chosenEdge = Edge.Top OrElse chosenEdge = Edge.Bottom Then
                leaderX = FindFreeSlot(leaderX, alongEdge, minimalDistanceBetweenBaloons)
                alongEdge.Add(leaderX)
            Else
                leaderY = FindFreeSlot(leaderY, alongEdge, minimalDistanceBetweenBaloons)
                alongEdge.Add(leaderY)
            End If

            Dim leaderPoints = ThisApplication.TransientObjects.CreateObjectCollection()
            leaderPoints.Add(transientGeom.CreatePoint2d(leaderX, leaderY))
            leaderPoints.Add(sheet.CreateGeometryIntent(tagCurve.Curve, midPoint))
            sheet.Balloons.Add(leaderPoints)
        Next
    End Sub

    Private Shared Function ChooseClosestEdge(midPoint As Point2d, top As Double, bottom As Double, left As Double, right As Double) As Edge
        Dim distTop = Math.Abs(top - midPoint.Y)
        Dim distBottom = Math.Abs(bottom - midPoint.Y)
        Dim distLeft = Math.Abs(left - midPoint.X)
        Dim distRight = Math.Abs(right - midPoint.X)

        Dim horizontalEdge As Edge = If(distTop <= distBottom, Edge.Top, Edge.Bottom)
        Dim horizontalDist = Math.Min(distTop, distBottom)
        Dim verticalEdge As Edge = If(distLeft <= distRight, Edge.Left, Edge.Right)
        Dim verticalDist = Math.Min(distLeft, distRight)

        ' Tie goes to vertical edge to preserve original behavior
        Return If(horizontalDist < verticalDist, horizontalEdge, verticalEdge)
    End Function

    Private Shared Function ComputeLeaderPoint(edge As Edge, midPoint As Point2d, top As Double, bottom As Double, left As Double, right As Double, balloonDiameter As Double) As (X As Double, Y As Double)
        Select Case edge
            Case Edge.Top
                Return (midPoint.X + balloonDiameter, top + balloonDiameter)
            Case Edge.Bottom
                Return (midPoint.X + balloonDiameter, bottom - balloonDiameter)
            Case Edge.Left
                Return (left - balloonDiameter, midPoint.Y + balloonDiameter)
            Case Else ' Edge.Right
                Return (right + balloonDiameter, midPoint.Y + balloonDiameter)
        End Select
    End Function

    ' Returns a position along an edge that is at least minDist away from every
    ' already-placed balloon on that edge. Starts at the desired position and
    ' spirals outward in both directions.
    Private Shared Function FindFreeSlot(desired As Double, placed As List(Of Double), minDist As Double) As Double
        If Not placed.Any(Function(p) Math.Abs(p - desired) < minDist) Then Return desired

        Dim k As Integer = 1
        Do
            Dim candidatePos As Double = desired + k * minDist
            If Not placed.Any(Function(p) Math.Abs(p - candidatePos) < minDist) Then Return candidatePos

            Dim candidateNeg As Double = desired - k * minDist
            If Not placed.Any(Function(p) Math.Abs(p - candidateNeg) < minDist) Then Return candidateNeg

            k += 1
        Loop
    End Function

    Public Class TagCurve

        Public Sub New(drawingCurve As DrawingCurve)
            Curve = drawingCurve

            EdgeProxy = drawingCurve.ModelGeometry
            Occ = EdgeProxy.ContainingOccurrence.OccurrencePath.Item(1)
            FileName = Occ.ReferencedFileDescriptor.FullFileName

            CalculateSize()
        End Sub

        Public Property FileName As String
        Public ReadOnly Property Curve As DrawingCurve
        Public Property SquaredSize As Double

        Public Sub CalculateSize()
            Dim rangeBox = Curve.Evaluator2D.RangeBox
            Dim p1 = rangeBox.MinPoint
            Dim p2 = rangeBox.MaxPoint
            SquaredSize = (p1.X - p2.X) ^ 2 + (p1.Y - p2.Y) ^ 2
        End Sub

        Private ReadOnly EdgeProxy As EdgeProxy
        Public Occ As ComponentOccurrence
    End Class

End Class
```
