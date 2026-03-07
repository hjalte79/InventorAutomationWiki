# Automatically generate bend notes

I have now 3 rules on this site to automatically generate dimensions. ([Overall dimensions](./generateOverallDimensions.md), [Hole position](./generateHolePosition.md) and [Bend dimensions](./generateBendDimensions.md)) At the moment I can't think of any more types of dimensions I can generate automatically. But there is something left that we can generate.

As you might have guessed from the title I'm talking about generating bend notes. To be honest this is the simplest rule of them all. There is a VBA example in the help files that shows how you can add a bend note to a line. The only thing we need is a list of all bed curves. And use it on the code in the help example file. (Also we need to convert it to VB.net code because you should [not use VBA](./VbaIsObsolete.md).) Nothing I haven't done before. Anyway, This rule is so short that I can't say anything useful about it. I hope the rule speaks for itself.

## Just for fun.

Sometimes I look at code and think I can do this in a shorter way. For example like this:

```vb.net
Dim view As DrawingView = ThisApplication.CommandManager.Pick(
    SelectionFilterEnum.kDrawingViewFilter,
    "Select a drawing view")

If (view Is Nothing) Then Return

view.DrawingCurves.Cast(Of DrawingCurve).
    Where(Function(c) c.EdgeType = DrawingEdgeTypeEnum.kBendDownEdge Or
                      c.EdgeType = DrawingEdgeTypeEnum.kBendUpEdge).ToList().
    ForEach(Function(c) view.Parent.DrawingNotes.BendNotes.Add(c))
```

You could go extreme.

```vb.net
Dim view As DrawingView = ThisApplication.CommandManager.Pick(SelectionFilterEnum.kDrawingViewFilter, "Select a drawing view")
If (view Is Nothing) Then Return
view.DrawingCurves.Cast(Of DrawingCurve).Where(Function(c) c.EdgeType = DrawingEdgeTypeEnum.kBendDownEdge Or c.EdgeType = DrawingEdgeTypeEnum.kBendUpEdge).ToList().ForEach(Function(c) view.Parent.DrawingNotes.BendNotes.Add(c))
```

As you can see it's possible to write 13 lines of code in just 3 lines. But shorter is not always better. Probably you will be having a hard time figuring out what it does. (if you only have a rule with these 3 lines.)

On the other end, it is also possible to write everything out and add comments.

```vb.net
' Let the user select the view that he/she wants to update.
Dim commandManager As CommandManager = ThisApplication.CommandManager
Dim view As DrawingView = commandManager.Pick(
       SelectionFilterEnum.kDrawingViewFilter,
       "Select a drawing view")

' Check if the user selected a view.
If (view Is Nothing) Then
    ' If not stop the rule.
    Return
End If

' Create a list of all bend cures
Dim bendCUrves As List(Of DrawingCurve)
bendCUrves = New List(Of DrawingCurve)
For Each curve As DrawingCurve In view.DrawingCurves
    If (curve.EdgeType = DrawingEdgeTypeEnum.kBendDownEdge) Then
        bendCUrves.Add(curve)
    End If
    If (curve.EdgeType = DrawingEdgeTypeEnum.kBendUpEdge) Then
        bendCUrves.Add(curve)
    End If
Next

' Add bend notes to all sheets.
Dim sheet As Sheet = view.Parent
Dim bendNotes = sheet.DrawingNotes.BendNotes
For Each curve As DrawingCurve In bendCUrves
    bendNotes.Add(curve)
Next
```

 This makes it very long. I'm not so sure that it is better. If all code is this long it will also be hard to read. What do you think is the best?