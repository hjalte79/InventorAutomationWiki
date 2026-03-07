# Minium rangebox in any orientation

I came across this cool piece of code that draws a bounding box around any part in any orientation. I did improve it a bit but credits should go to the topic starter of this post "[a dynamic box?](https://forums.autodesk.com/t5/inventor-programming-forum/a-dynamic-box/m-p/10786550/highlight/true#M131805)".

```vb.net
Sub Main()
    Dim margin1 = 1 ' cm
    Dim margin2 = 1 ' cm
    Dim margin3 = 1 ' cm

    ' Get the current Part document.
    Dim partDoc As PartDocument = ThisDoc.Document

    ' Get the TransientBRep and TransientGeometry objects.
    Dim transBRep As TransientBRep = ThisApplication.TransientBRep
    Dim transGeom As TransientGeometry = ThisApplication.TransientGeometry

    ' Combine all bodies in Part into a single transient Surface Body.
    Dim combinedBodies As SurfaceBody = Nothing
    For Each surfBody As SurfaceBody In partDoc.ComponentDefinition.SurfaceBodies
        If combinedBodies Is Nothing Then
            combinedBodies = transBRep.Copy(surfBody)
        Else
            transBRep.DoBoolean(combinedBodies, surfBody, BooleanTypeEnum.kBooleanTypeUnion)
        End If
    Next

    ' Get the oriented mininum range box of all bodies in Part.
    ' NOTE: "OrientedMinimumRangeBox" was added in Inventor 2020.3/2021.
    Dim minBox As OrientedBox = combinedBodies.OrientedMinimumRangeBox

    ' Create starting box with dimensions of oriented rangebox.
    Dim startBox As Box = transGeom.CreateBox()
    Dim boxMaxPoint As Point = transGeom.CreatePoint(
        minBox.DirectionOne.Length + margin1,
        minBox.DirectionTwo.Length + margin2,
        minBox.DirectionThree.Length + margin3)
    startBox.Extend(boxMaxPoint)
    Dim boxMinPoint As Point = transGeom.CreatePoint(-margin1, -margin2, -margin3)
    startBox.Extend(boxMinPoint)

    ' Create surface body for the range box.
    Dim minBoxSurface As SurfaceBody = ThisApplication.TransientBRep.CreateSolidBlock(startBox)

    ' Create transformation matrix to move box to correct location/orientation.
    Dim transMatrix As Matrix = ThisApplication.TransientGeometry.CreateMatrix
    transMatrix.SetCoordinateSystem(
        minBox.CornerPoint,
        minBox.DirectionOne.AsUnitVector.AsVector,
        minBox.DirectionTwo.AsUnitVector.AsVector,
        minBox.DirectionThree.AsUnitVector.AsVector)

    ' Transform range box surface body to the correct location/orientation.
    ThisApplication.TransientBRep.Transform(minBoxSurface, transMatrix)


    Dim cGraphics As ClientGraphics = CreateBox(partDoc, minBoxSurface)

    ShowMinRangeDimensions(partDoc, minBox)

    cGraphics.Delete()

End Sub

Private Sub ShowMinRangeDimensions(partDoc As PartDocument, minBox As OrientedBox)
    ' Get length of each side of mininum range box.
    Dim dir1 As Double = minBox.DirectionOne.Length
    Dim dir2 As Double = minBox.DirectionTwo.Length
    Dim dir3 As Double = minBox.DirectionThree.Length

    ' Convert lengths to document's length units.
    Dim uom As UnitsOfMeasure = partDoc.UnitsOfMeasure

    dir1 = uom.ConvertUnits(dir1, UnitsTypeEnum.kDatabaseLengthUnits, uom.LengthUnits)
    dir2 = uom.ConvertUnits(dir2, UnitsTypeEnum.kDatabaseLengthUnits, uom.LengthUnits)
    dir3 = uom.ConvertUnits(dir3, UnitsTypeEnum.kDatabaseLengthUnits, uom.LengthUnits)

    ' Sort lengths from smallest to largest.
    Dim lengths As New List(Of Double) From {dir1, dir2, dir3}
    lengths.Sort()

    Dim minLength As Integer = lengths(0)
    Dim midLength As Integer = lengths(1)
    Dim maxLength As Integer = lengths(2)

    ' Display message with minimum rangebox size.
    MessageBox.Show("Oriented Minimum Rangebox Size: " &
        minLength.ToString("#.###") & " x " & midLength.ToString("#.###") & " x " & maxLength.ToString("#.###"),
        "Oriented Minimum Rangebox", MessageBoxButtons.OK, MessageBoxIcon.Information)
End Sub

Private Function CreateBox(partDoc As Document, surface As Object) As ClientGraphics
    Dim cGraphics As ClientGraphics

    cGraphics = partDoc.ComponentDefinition.ClientGraphicsCollection.Add("OrientedRangeBox")
    Dim surfacesNode As GraphicsNode = cGraphics.AddNode(1)
    Dim surfGraphics As SurfaceGraphics = surfacesNode.AddSurfaceGraphics(surface)
    Dim targetAppearance As Asset

    Try
        targetAppearance = partDoc.Assets.Item("Clear - Blue")
    Catch
        Dim sourceAppearance As Asset = ThisApplication.AssetLibraries.Item("314DE259-5443-4621-BFBD-1730C6CC9AE9").AppearanceAssets.Item("InvGen-001-1-2") ' "Clear - Blue"
        targetAppearance = sourceAppearance.CopyTo(partDoc)
    End Try

    surfacesNode.Appearance = targetAppearance

    ThisApplication.ActiveView.Update()

    Return cGraphics
End Function
```