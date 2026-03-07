# Colour parts in assembly by part number

Early 2020 someone started this topic "[Set different colours for each component of Assembly based on the part number](https://forums.autodesk.com/t5/inventor-programming-forum/set-different-colour-for-each-component-of-assembly-based-on-the/td-p/9318077)". In that post, an article by Clint Brown was referenced. In [that article](https://clintbrown.co.uk/2019/05/04/ilogic-set-every-part-to-a-different-colour/), Clint proposed an iLogic rule for colouring each part. I updated Clint's code so it would not colour all parts with a random colour, but to give each part with the same part number the same colour.

The improved iLogic rule was never published. but recently I needed that rule again for some other question on the forum. I  decided that I could (re)publish it after a polish. In this latest version, it's also easier for you to change which parts should get the same colour. For example, if you want parts with the same Stock number to have the same colour then change the function "GetIdentifier()". The only thing that you need to do is change the PropertySet name and property Name. These are all the property(set) names:

- Inventor Summary Information
  - Author, Comments, Keywords, Last Saved By, Revision Number, Subject, Title
- Inventor Document Summary Information
  - Category, Company, Manager
- Design Tracking Properties
  - Authority, Catalog Web Link, Categories, Checked By, Cost, Cost Center, Creation Time, Date Checked, Defer Updates, Description, Design Status, Designer, Document SubType, Document SubType Name, Engineer, Engr Approved By, Engr Date Approved, External Property Revision Id, Language, Manufacturer, Material, Mfg Approved By, Mfg Date Approved, Parameterized Template, Part Icon, Part Number, Part Property Revision Id, Project, Proxy Refresh Date, Size Designation, Standard, Standard Revision, Standards Organization, Stock Number, Template Row, User Status, Vendor, Weld Material
- Inventor User Defined Properties
  - (All custom made iProperty names)

On a side note: This is how I found the site of Clint and he inspired me to start writing. At first, I started writing for his site but when he stopped adding new content early this year I started for myself on this site. [Clint's site](https://clintbrown.co.uk/) is still there. If you like my content then you definitely need to check out his site.

```vb.net
Public Class ThisRule
    Sub Main()
        Dim doc As AssemblyDocument = ThisDoc.Document
        Dim trans As Transaction = ThisApplication.TransactionManager.
                    StartTransaction(doc, "Unique Colors")
        SetDesignViewRepresentation(doc)
        Dim knowOcc As New Dictionary(Of String, Asset)
        For Each occ As ComponentOccurrence In doc.ComponentDefinition.Occurrences
            Dim id As String = GetIdentifier(occ)
            If (knowOcc.ContainsKey(id)) Then
                occ.Appearance = knowOcc.Item(id)
            Else
                Dim asset As Asset = GetColorAsset(doc)
                occ.Appearance = asset
                knowOcc.Add(id, asset)
            End If
        Next
        trans.End()
    End Sub

    Private Function GetColorAsset(doc As AssemblyDocument)
        Dim asset As Asset = doc.Assets.Add(
                    AssetTypeEnum.kAssetTypeAppearance,
                    "Generic",
                    "appearances")
        Dim uColor As ColorAssetValue = asset.Item("generic_diffuse")
        Dim RNG = Math.Round(Rnd() * 255)
        Dim RNG1 = Math.Round(Rnd() * 255)
        Dim RNG2 = Math.Round(Rnd() * 255)
        uColor.Value = ThisApplication.TransientObjects.
                    CreateColor(RNG, RNG1, RNG2)
        Return asset
    End Function

    Private Function GetIdentifier(occ As ComponentOccurrence)
        Dim occDoc As Document = occ.Definition.Document
        Dim occPropSet As PropertySet = occDoc.PropertySets.Item("Design Tracking Properties")
        Dim PartNrProp As [Property] = occPropSet.Item("Part Number")
        Return PartNrProp.Value
    End Function

    Private Sub SetDesignViewRepresentation(doc As AssemblyDocument)
        Dim ucRep As DesignViewRepresentation
        Dim manager As RepresentationsManager = doc.ComponentDefinition.RepresentationsManager
        Try
            ucRep = manager.DesignViewRepresentations("Unique Colors")
        Catch ex As Exception
            ucRep = manager.DesignViewRepresentations.Add("Unique Colors")
        End Try
        ucRep.Activate()
    End Sub

    ' Code based on an Inventor Ideas forum post by Tanner Dant
    ' https://forums.autodesk.com/t5/Inventor-ideas/visual-style-every-part-With-different-color/idc-p/8713666#M34691
    ' Adapted for iLogic by @ClintBrown3D, originally posted at https://clintbrown.co.uk/ilogic-set-every-part-to-a-different-colour
    ' Adapted by Jelte de Jong, and published on www.hjalte.nl

End Class
```