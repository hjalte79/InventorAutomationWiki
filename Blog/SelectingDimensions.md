# Selecting dimensions 

While selecting dimensions is possible with the easy to use command "ThisApplication.CommandManager.Pick(Filter As SelectionFilterEnum, PromptText As String)" it's impossible to distinguish between the dimension line and extension lines. That is precisely what was asked for in this post.

Answering this post was a bit more complex than the normal post on the forum. The solution has some nice technics that I want to document. First of all, it showcases how to use InteractionEvents to make a custom pick command. see the Selector class. The second thing is the code to check if a point is on a line. Check the function "Selector.DimensionLineIsSelected" Maybe not very impressive but these are the things that I will need someday and can't find it again.

```vb.net
Public Class ThisRule
    Sub Main()

        Dim doc As DrawingDocument = ThisDoc.Document
        Dim sheet As Sheet = doc.ActiveSheet
        Dim symbols As SurfaceTextureSymbols = sheet.SurfaceTextureSymbols

        Dim selector As New Selector(ThisApplication)
        selector.Pick()

        If (selector.SelectedObjectType = SelectedObjectType.NoSelection) Then Return

        Dim style As SurfaceTextureStyle = doc.StylesManager.SurfaceTextureStyles.Item(1)
        Dim definition As SurfaceTextureSymbolDefinition = symbols.CreateDefinition(style)

        Dim intent As GeometryIntent = sheet.CreateGeometryIntent(selector.SelectedObject, selector.PointOnSheet)
        Dim collection As ObjectCollection = ThisApplication.TransientObjects.CreateObjectCollection()
        collection.Add(intent)

        Dim symbol As SurfaceTextureSymbol = symbols.AddByDefinition(collection, definition)

        Select Case selector.SelectedObjectType
            Case SelectedObjectType.DimensionLine
                symbol.ProductionMethod = "DimensionLine selected"
            Case SelectedObjectType.ExtensionLine
                symbol.ProductionMethod = "ExtensionLine selected"
            Case SelectedObjectType.DrawingCurve
                symbol.ProductionMethod = "DrawingCurve selected"
        End Select

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
        _interactEvents.StatusBarText = "Select a LinearGeneralDimension."
        _interactEvents.SetCursor(CursorTypeEnum.kCursorBuiltInLineCursor)

        _selectEvents = _interactEvents.SelectEvents
        _selectEvents.WindowSelectEnabled = False

        _interactEvents.Start()
        Do While _stillSelecting
            _inventor.UserInterfaceManager.DoEvents()
        Loop
        _interactEvents.Stop()

        _inventor.CommandManager.StopActiveCommand()

    End Sub

    Public Property SelectedObject As Object = Nothing
    Public Property PointOnSheet As Point2d = Nothing
    Public Property SelectedObjectType As SelectedObjectType = SelectedObjectType.NoSelection
    Public ReadOnly Property DimensionLineIsSelected As Boolean
        Get
            If (SelectedObject Is Nothing) Then Return False
            If (Not TypeOf SelectedObject Is LinearGeneralDimension) Then Return False

            Dim line As LineSegment2d = SelectedObject.DimensionLine

            Dim x = PointOnSheet.X
            Dim y = PointOnSheet.Y
            Dim x1 = line.StartPoint.X
            Dim y1 = line.StartPoint.Y
            Dim x2 = line.EndPoint.X
            Dim y2 = line.EndPoint.Y

            Dim AB = Math.Sqrt((x2 - x1) ^ 2 + (y2 - y1) ^ 2)
            Dim AP = Math.Sqrt((x - x1) ^ 2 + (y - y1) ^ 2)
            Dim PB = Math.Sqrt((x2 - x) ^ 2 + (y2 - y) ^ 2)

            If (Math.Abs(AB - AP - PB) < 0.000001) Then
                Return True
            Else
                Return False
            End If
        End Get
    End Property

    ''' <summary>
    '''     check in this sub if the user hovers over a object that you are looking for
    ''' </summary>
    ''' <param name="PreSelectEntity">Object that the user is hovering over</param>
    ''' <param name="DoHighlight">Set True/False to highlight the selected objet(s)</param>
    ''' <param name="MorePreSelectEntities">Set to highlight more object</param>
    ''' <param name="ModelPosition">
    '''     Point of mouse in space. 
    '''     In 2D-space (drawings) the Z-value is 0.
    ''' </param>
    Private Sub oSelectEvents_OnPreSelect(
            ByRef PreSelectEntity As Object,
            ByRef DoHighlight As Boolean,
            ByRef MorePreSelectEntities As ObjectCollection,
            SelectionDevice As SelectionDeviceEnum,
            ModelPosition As Point,
            ViewPosition As Point2d,
            View As Inventor.View) Handles _selectEvents.OnPreSelect

        DoHighlight = False

        If TypeOf PreSelectEntity Is LinearGeneralDimension Then
            DoHighlight = True
        End If

        If TypeOf PreSelectEntity Is DrawingCurveSegment Then
            Dim segment As DrawingCurveSegment = PreSelectEntity
            If (segment.GeometryType = Curve2dTypeEnum.kLineSegmentCurve2d) Then
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
        PointOnSheet = _inventor.TransientGeometry.CreatePoint2d(ModelPosition.X, ModelPosition.Y)

        If TypeOf SelectedObject Is DrawingCurveSegment Then
            SelectedObject = SelectedObject.Parent
            SelectedObjectType = SelectedObjectType.DrawingCurve
        Else
            If (DimensionLineIsSelected) Then
                SelectedObjectType = SelectedObjectType.DimensionLine
            Else
                SelectedObjectType = SelectedObjectType.ExtensionLine
            End If
        End If

        _stillSelecting = False
    End Sub

End Class
Public Enum SelectedObjectType
    NoSelection
    DrawingCurve
    DimensionLine
    ExtensionLine
End Enum
```