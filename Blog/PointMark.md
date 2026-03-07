# Point, click and mark a part with a part number 

Sinds Inventor 2023 we have the new mark-feature. This is a cool feature with lots of possible use cases. But I expect that many users want to use it for laser etching some number on a sheet metal part. For example to etch the part number. Now if you want to do that on all your parts then you're in for a lot of work. Just this week there was a question on the forum. If it would be possible to automate the process. (Link)

In short, this person wanted to select a point on a face. That point should be used to place the mark feature on. The text of the mark feature should contain the part number.

I created the following rule. It mainly consists of 3 main parts.

- Main method. (Lines 3 to19)
- The method to create the mark feature. (Lines 23 to 45)
- The method (class) to help with selecting the point on a face

If you are new to iLogic or if you are just here for the functionality then you only need to have a look at the main method. The first 3 lines of the method contain settings for the rule. To change the mark style used change the "markStyleName" on line 3.
To change the mark text you need to change the "markText" on line 4. In this example, I used iLogic to get the part number but you could use anything here.
To change the text height you need to change the "textHeight" on line 5. Pay attention that this value is in Cm.

The "CreateMarkFeatur(...)" is a nice example of creating a mark feature using the Inventor API.

But the exciting part is the "Selector-class". In most cases when you want a user to select something then there is one simple command:

```vb.net
ThisApplication.CommandManager.Pick(Filter, promptText)
```

However in this case I want the user to select a point on a face. That is not possible with the command above. It will allow your user to choose a face but it can't select points in space. Therefore, I created the "Selector-class". There are 3 main parts.

The "pick-method" will kick off all events needed and hold some settings. (Lines 60 to 75.) Please notice the endless loop (lines 70 to 72). This might seem odd but while the loop is running the selecting is done. The loop is stopped on termination of the command or if something is selected. When the command is terminated the method "oInteractEvents_OnTerminate()" is called. (Lines 88 to 90) That method only sets the variable "_stillSelecting = False". That stops the loop selection loop. The method "oSelectEvents_OnSelect(...)" is called when something is selected. (Lines 111 to 120.) Here we check if the selected entity is a face and if it's a planar face. The ability to also check if a face is planar is another advantage of this way of selecting something. If so then the face and model space point are saved. But whatever is clicked also here the variable "_stillSelecting = False" is set. Which also ends the loop and allows to rule to move on. Then last but not least method "oSelectEvents_OnPreSelect(...)" is there to check everything that is hovered over with the mouse. (Lines 91 to 97.) Also here we check if the entity that is hovered over is a face and if it's a planar face. Except here we set the variable "DoHighlight = True". This will highlight the entity that is hovered over.

```vb.net
Public Class ThisRule
    Sub Main()
        Dim markStyleName = "Mark Surface"
        Dim markText = iProperties.Value("Project", "Part Number")
        Dim textHeight = 2 'Cm

		If (String.IsNullOrWhiteSpace(markText)) Then
			MsgBox("Mark text was not set. Ending this rule now.",, "No mark text")
			Return
		End If
		
        Dim selector As New Selector(ThisApplication)
        selector.Pick()
		
		If (selector.SelectedObject Is Nothing) Then Return
        Dim face As Face = selector.SelectedObject
        Dim pointOnFace As Point = selector.ModelPosition

        CreateMarkFeatur(face, pointOnFace, markStyleName, markText, textHeight)
    End Sub

    Private Sub CreateMarkFeatur(face As Face, pointOnFace As Point, markStyleName As String, markText As String, textHeight As Double)
        Dim doc As PartDocument = ThisDoc.Document
        Dim def As PartComponentDefinition = doc.ComponentDefinition

        Dim sketch = def.Sketches.Add(face)
        Dim modelPoint As Point2d = sketch.ModelToSketchSpace(pointOnFace)

        Dim formatedText = String.Format("<StyleOverride FontSize='{0}'>{1}</StyleOverride>", textHeight, markText)
        Dim sketchText = sketch.TextBoxes.AddFitted(modelPoint, formatedText)

        Dim markGeometry As ObjectCollection = ThisApplication.TransientObjects.CreateObjectCollection()
        markGeometry.Add(sketchText)

        Dim markFeatures As MarkFeatures = def.Features.MarkFeatures
        Dim markStyle As MarkStyle = doc.MarkStyles.Item(markStyleName)
        Dim markDef As MarkDefinition = markFeatures.CreateMarkDefinition(markGeometry, markStyle)
		If (ThisApplication.SoftwareVersion.Major >= 28) Then
			' The following property was introduced in Inventor 2024
			' Therefore we can't set this property before Inventor 2024
			' If this is not set after Inventor 2023 the rule fails most of the time.
			' (Major version 28 = Inventor 2024)
			markDef.Direction = PartFeatureExtentDirectionEnum.kNegativeExtentDirection
		End If        
        markFeatures.Add(markDef)
    End Sub
End Class
Public Class Selector

    Private WithEvents _interactEvents As InteractionEvents
    Private WithEvents _selectEvents As SelectEvents
    Private _stillSelecting As Boolean
    Private _inventor As Inventor.Application

    Public Sub New(ThisApplication As Inventor.Application)
        _inventor = ThisApplication
    End Sub

    Public Sub Pick()
        _stillSelecting = True

        _interactEvents = _inventor.CommandManager.CreateInteractionEvents
        _interactEvents.InteractionDisabled = False
        _interactEvents.StatusBarText = "Select a point on a face."

        _selectEvents = _interactEvents.SelectEvents
        _selectEvents.WindowSelectEnabled = False

        _interactEvents.Start()
        Do While _stillSelecting
            _inventor.UserInterfaceManager.DoEvents()
        Loop
        _interactEvents.Stop()

        _inventor.CommandManager.StopActiveCommand()

    End Sub

    Public Property SelectedObject As Face = Nothing
    Public Property ModelPosition As Point = Nothing

    Private Sub oSelectEvents_OnPreSelect(
            ByRef PreSelectEntity As Object,
            ByRef DoHighlight As Boolean,
            ByRef MorePreSelectEntities As ObjectCollection,
            SelectionDevice As SelectionDeviceEnum,
            ModelPosition As Point,
            ViewPosition As Point2d,
            View As Inventor.View) Handles _selectEvents.OnPreSelect

        DoHighlight = False

        If TypeOf PreSelectEntity Is Face Then
            If TypeOf PreSelectEntity.Geometry Is Plane Then
                DoHighlight = True
            End If
        End If
    End Sub

    Private Sub oInteractEvents_OnTerminate() Handles _interactEvents.OnTerminate
        _stillSelecting = False
    End Sub

    Private Sub oSelectEvents_OnSelect(
            ByVal JustSelectedEntities As ObjectsEnumerator,
            ByVal SelectionDevice As SelectionDeviceEnum,
            ByVal ModelPosition As Point,
            ByVal ViewPosition As Point2d,
            ByVal View As Inventor.View) Handles _selectEvents.OnSelect

        SelectedObject = JustSelectedEntities.Item(1)

        If TypeOf SelectedObject Is Face Then
            If TypeOf SelectedObject.Geometry Is Plane Then
                Me.SelectedObject = SelectedObject
                Me.ModelPosition = ModelPosition
            End If
        End If

        _stillSelecting = False
    End Sub
    ' Copyright 2024
    ' 
    ' This code was written by Jelte de Jong, and published on www.hjalte.nl
    '
    ' Permission Is hereby granted, free of charge, to any person obtaining a copy of this 
    ' software And associated documentation files (the "Software"), to deal in the Software 
    ' without restriction, including without limitation the rights to use, copy, modify, merge, 
    ' publish, distribute, sublicense, And/Or sell copies of the Software, And to permit persons 
    ' to whom the Software Is furnished to do so, subject to the following conditions:
    '
    ' The above copyright notice And this permission notice shall be included In all copies Or
    ' substantial portions Of the Software.
    ' 
    ' THE SOFTWARE Is PROVIDED "AS IS", WITHOUT WARRANTY Of ANY KIND, EXPRESS Or IMPLIED, 
    ' INCLUDING BUT Not LIMITED To THE WARRANTIES Of MERCHANTABILITY, FITNESS For A PARTICULAR 
    ' PURPOSE And NONINFRINGEMENT. In NO Event SHALL THE AUTHORS Or COPYRIGHT HOLDERS BE LIABLE 
    ' For ANY CLAIM, DAMAGES Or OTHER LIABILITY, WHETHER In AN ACTION Of CONTRACT, TORT Or 
    ' OTHERWISE, ARISING FROM, OUT Of Or In CONNECTION With THE SOFTWARE Or THE USE Or OTHER 
    ' DEALINGS In THE SOFTWARE.
End Class
```
