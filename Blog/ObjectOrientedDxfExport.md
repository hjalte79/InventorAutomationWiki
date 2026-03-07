# Dxf export made easy
Just a random post about an object-oriented way of exporting flat patterns to dxf. if you want to use the code you only need to change the stuff in the main-sub. All other stuff is just there to help you (and not to be touched).

```vb.net
Public Class ThisRule
	Sub Main()
		Dim doc As PartDocument = ThisDoc.Document

		Dim newFileName As String = ThisDoc.PathAndFileName(False) & ".dxf"

		Dim red = ThisApplication.TransientObjects.CreateColor(255, 0, 0)
		Dim green = ThisApplication.TransientObjects.CreateColor(0, 255, 0)
		Dim blue = ThisApplication.TransientObjects.CreateColor(0, 0, 255)
		
		Dim Exporter As New FlatPatternDxfExporterService()
		' Basic options
		Exporter.AcadVersion = "2018"
		Exporter.RebaseGeometry = True	
		' Layer options
		Exporter.Layers.OuterProfileLayer.Name = "Outer profile"
		Exporter.Layers.OuterProfileLayer.Color = red		
		Exporter.Layers.InteriorProfilesLayer.LineWeight = 0.01		
		Exporter.Layers.BendUpLayer.Name = "Bend up"
		Exporter.Layers.BendUpLayer.Color = green
		Exporter.Layers.BendUpLayer.LineWeight = 0.01		
		Exporter.Layers.BendDownLayer.Name = "Bend down"
		Exporter.Layers.BendDownLayer.Color = blue
		Exporter.Layers.BendDownLayer.LineWeight = 0.01		
		Exporter.Layers.TangentLayer.Hidden = True		
		Exporter.Export(doc, newFileName)
		
	End Sub
End Class

Public Class FlatPatternDxfExporterService

	''' <summary>Only use one of the following values: 2018, 2013, 2010, 2007, 2004, 2000, or R12</summary>
	Public Property AcadVersion As String = "2018"

	''' <summary>Enable spline replacement(by linear segments or arcs).</summary>
	Public Property SimplifySplines As Boolean = True

	''' <summary>Chord tolerance for spline replacement.</summary>
	Public Property SplineTolerance As Double = 0.01

	Public Property AdvancedLegacyExport As Boolean = True

	''' <summary>Build a polyline Of the exterior profiles</summary>
	Public Property MergeProfilesIntoPolyline As Boolean = False

	''' <summary>Trim the centerlines at contour.</summary>
	Public Property TrimCenterlinesAtContour As Boolean = False

	''' <summary>Move geometry to 1st quadrant.</summary>
	Public Property RebaseGeometry As Boolean = False

	''' <summary>
	''' True: Replace splines by tangent arcs. 
	''' False: Replace splines by line segments. 
	''' The SimplifySplines should be specified To True otherwise this Option Is ignored.
	''' </summary>
	Public Property SimplifyAsTangentArcs As Boolean = False

	Public Property Layers As OverallLayerSettings = New OverallLayerSettings()

	Public Sub Export(doc As Inventor.PartDocument, newFileName As String)
		If doc Is Nothing Then Return

		If (String.IsNullOrWhiteSpace(newFileName)) Then
			Throw New Exception("No DXF FullFileName specified.")
		End If

		If (doc.SubType <> "{9C464203-9BAE-11D3-8BAD-0060B0CE6BB4}") Then
			Throw New Exception("Document is not a Sheet Metal Part.")
		End If

		Dim def As SheetMetalComponentDefinition = doc.ComponentDefinition

		If (System.IO.File.Exists(newFileName)) Then
			Dim nl = System.Environment.NewLine
			Dim msg = String.Format("The following DXF file already exists.{1}{0}{1}Do you want to overwrite it?", newFileName, nl)
			Dim result As MsgBoxResult = MsgBox(msg, vbYesNo + vbQuestion + vbDefaultButton2, "Dxf already exists")
			If result = MsgBoxResult.No Then Return
		End If

		If (Not def.HasFlatPattern) Then
			Try
				def.Unfold()
				def.FlatPattern.ExitEdit()
			Catch
				Throw New Exception("Error unfolding " & doc.FullDocumentName)
			End Try

		End If
		Dim flatpattern As FlatPattern = def.FlatPattern

		If flatpattern Is Nothing Then
			Throw New Exception("Sheet Metal FlatPattern was not attainable.")
		End If

		Try
			flatpattern.DataIO.WriteDataToFile(ToString(), newFileName)
		Catch ex As Exception
			Throw New Exception("Failed to export to dxf.")
		End Try
	End Sub

	Public Overrides Function ToString() As String
		Dim settingsString As String = "FLAT PATTERN DXF?"

		Dim settings As New List(Of String)
		settings.Add("AcadVersion=" & AcadVersion)
		settings.Add("SimplifySplines=" & SimplifySplines)
		settings.Add("SplineTolerance=" & SplineTolerance)
		settings.Add("AdvancedLegacyExport=" & AdvancedLegacyExport)
		settings.Add("MergeProfilesIntoPolyline=" & MergeProfilesIntoPolyline)
		settings.Add("TrimCenterlinesAtContour=" & TrimCenterlinesAtContour)
		settings.Add("RebaseGeometry=" & RebaseGeometry)
		settings.Add("SimplifyAsTangentArcs=" & SimplifyAsTangentArcs)
		settings.Add(Layers.ToString())
		Return "FLAT PATTERN DXF?" & String.Join("&", settings)
	End Function
End Class

Public Class OverallLayerSettings
	Public Property TangentLayer() As LayerSettings = New LayerSettings("TangentLayer", "IV_TANGENT")
	Public Property OuterProfileLayer() As LayerSettings = New LayerSettings("OuterProfileLayer", "IV_OUTER_PROFILE")
	Public Property ArcCentersLayer() As LayerSettings = New LayerSettings("ArcCentersLayer", "IV_ARC_CENTERS")
	Public Property InteriorProfilesLayer() As LayerSettings = New LayerSettings("InteriorProfilesLayer", "IV_INTERIOR_PROFILES")
	Public Property BendUpLayer() As LayerSettings = New LayerSettings("BendUpLayer", "IV_BEND")
	Public Property BendDownLayer() As LayerSettings = New LayerSettings("BendDownLayer", "IV_BEND_DOWN")
	Public Property ToolCenterUpLayer() As LayerSettings = New LayerSettings("ToolCenterUpLayer", "IV_TOOL_CENTER")
	Public Property ToolCenterDownLayer() As LayerSettings = New LayerSettings("ToolCenterDownLayer", "IV_TOOL_CENTER_DOWN")
	Public Property FeatureProfilesUpLayer() As LayerSettings = New LayerSettings("FeatureProfilesUpLayer", "IV_FEATURE_PROFILES")
	Public Property FeatureProfilesDownLayer() As LayerSettings = New LayerSettings("FeatureProfilesDownLayer", "IV_FEATURE_PROFILES_DOWN")
	Public Property AltRepFrontLayer() As LayerSettings = New LayerSettings("AltRepFrontLayer", "IV_ALTREP_FRONT")
	Public Property AltRepBackLayer() As LayerSettings = New LayerSettings("AltRepBackLayer", "IV_ALTREP_BACK")
	Public Property UnconsumedSketchesLayer() As LayerSettings = New LayerSettings("UnconsumedSketchesLayer", "IV_UNCONSUMED_SKETCHES")
	Public Property TangentRollLinesLayer() As LayerSettings = New LayerSettings("TangentRollLinesLayer", "IV_ROLL_TANGENT")
	Public Property RollLinesLayer() As LayerSettings = New LayerSettings("RollLinesLayer", "IV_ROLL")

	Public Overrides Function ToString() As String
		Dim settings As New List(Of String)
		Dim hiddenLayers As New List(Of String)
		AddSettingOrDefault(settings, hiddenLayers, TangentLayer)
		AddSettingOrDefault(settings, hiddenLayers, OuterProfileLayer)
		AddSettingOrDefault(settings, hiddenLayers, ArcCentersLayer)
		AddSettingOrDefault(settings, hiddenLayers, InteriorProfilesLayer)
		AddSettingOrDefault(settings, hiddenLayers, BendUpLayer)
		AddSettingOrDefault(settings, hiddenLayers, BendDownLayer)
		AddSettingOrDefault(settings, hiddenLayers, ToolCenterUpLayer)
		AddSettingOrDefault(settings, hiddenLayers, ToolCenterDownLayer)
		AddSettingOrDefault(settings, hiddenLayers, FeatureProfilesUpLayer)
		AddSettingOrDefault(settings, hiddenLayers, FeatureProfilesDownLayer)
		AddSettingOrDefault(settings, hiddenLayers, AltRepFrontLayer)
		AddSettingOrDefault(settings, hiddenLayers, AltRepBackLayer)
		AddSettingOrDefault(settings, hiddenLayers, UnconsumedSketchesLayer)
		AddSettingOrDefault(settings, hiddenLayers, TangentRollLinesLayer)
		AddSettingOrDefault(settings, hiddenLayers, RollLinesLayer)
		Dim settingsString = ""
		If (settings.Count > 0) Then
			settingsString = String.Join("&", settings)
		End If
		If (hiddenLayers.Count > 0) Then
			If (settings.Count > 0) Then settingsString += "&"

			settingsString += "InvisibleLayers=" + String.Join(";", hiddenLayers)
		End If
		Return settingsString
	End Function

	Private Sub AddSettingOrDefault(settings As List(Of String), hiddenLayers As List(Of String), value As LayerSettings)
		If (Not String.IsNullOrWhiteSpace(value.ToString())) Then
			settings.Add(value.ToString())
		End If
		If (value.Hidden) Then
			hiddenLayers.Add(value.Name)
		End If
	End Sub
End Class

Public Class LayerSettings
	Public Sub New(inventorName As String, defaultLayerName As String)

		Me.InventorName = inventorName
		Me.Name = defaultLayerName
		Me.NameOrg = defaultLayerName
	End Sub

	Private Property InventorName As String
	Public Property Name As String = String.Empty
	Private Property NameOrg As String = String.Empty

	Public Property Color As Color = Nothing
	Public Property LineType As LineTypeEnum = LineTypeEnum.kContinuousLineType
	Public Property LineWeight As Double = -1
	Public Property Hidden As Boolean = False

	Public Overrides Function ToString() As String
		Dim settings As New List(Of String)
		If (Name <> NameOrg) Then
			settings.Add(InventorName & "=" & Name)
		End If
		If (Color IsNot Nothing) Then
			settings.Add(String.Format("{0}Color={1};{2};{3}", InventorName, Color.Red, Color.Green, Color.Blue))
		End If
		If (LineType <> LineTypeEnum.kContinuousLineType) Then
			settings.Add(String.Format("{0}LineType={1}", InventorName, LineType.ToString()))
		End If
		If (LineWeight <> -1) Then
			settings.Add(String.Format("{0}LineWeight={1}", InventorName, LineWeight.ToString()))
		End If
		Return String.Join("&", settings)
	End Function
End Class
```