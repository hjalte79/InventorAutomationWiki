# Links

Top Accepted Solutions Authors in 
 - [Inventor Programming Forum](https://forums.autodesk.com/t5/solutions/acceptedsolutionsleaderboardpage/node-display-id/board%3Ainventor-programming-ilogic-forum-en/timerange/all)
  - [Inventor Forums](https://forums.autodesk.com/t5/solutions/acceptedsolutionsleaderboardpage/node-display-id/category%3Ainventor-en/timerange/all) 

 [Search Autodesk Inventor Help](https://www.google.com/search?q=site%3Ahelp.autodesk.com%2Fview%2FINVNTOR%2F2027%2FENU%2F+)

[Get the factory document from iLogic](https://forums.autodesk.com/t5/inventor-ilogic-api-vba-forum/retreaving-custom-iproperty/m-p/10955051/highlight/true#M134730)

```vb.net
Dim oPropertySetName As String = "Design Tracking Properties"
Dim oPropertyName As String = "Description"

Dim doc = ThisDoc.Document
If doc.ReferencedDocuments.Count > 0 Then
	Dim refDoc = doc.ReferencedDocuments(1)
	Dim stdObjects = StandardObjectFactory.Create(refDoc)
	Dim desc = stdObjects.iProperties.Value(oPropertySetName, oPropertyName)
	MessageBox.Show(String.Format("Description in {0} = {1}", refDoc.DisplayName, desc))
End If
```

[Creating forms with iLogic](https://forums.autodesk.com/t5/inventor-ilogic-and-vb-net-forum/vba-object-with-configurations/m-p/12177742/highlight/true#M156581)

# Autodesk University papers

[How Deep is the Rabbit Hole? Examing the Matrix and other Inventor Math and Geometry Objects.](./files/Autodesk%20Math%20Geometry.pdf)

# Model state stuff

[The Component.IsActive function in a delegated BOM model state does not update the BOM in Inventor 2022](https://knowledge.autodesk.com/support/inventor/troubleshooting/caas/sfdcarticles/sfdcarticles/The-Component-IsActive-function-in-a-delegated-BOM-model-state-does-not-update-the-BOM-in-Inventor-2022.html)

[Create and Activate Model States in the assembly and subassemblies](https://forums.autodesk.com/t5/inventor-ilogic-api-vba-forum/example-create-and-activate-model-states-in-the-assembly-and/td-p/10391500)

[Can't suppress sub-parts in Model States](https://forums.autodesk.com/t5/inventor-forum/can-t-suppress-sub-parts-in-model-states/m-p/10383812/highlight/true#M830851)

[Working with Model State API](https://help.autodesk.com/view/INVNTOR/2022/ENU/?guid=GUID-045E78DB-DBE3-4220-8EBC-29EB53890E1A)

[Working with PropertySets and Model States](https://blog.autodesk.io/working-with-propertysets-and-model-states)

[Porting guide from Level of Details to Model states.](https://blog.autodesk.io/porting-guide-from-level-of-details-to-model-states)

# Vault

[Get a Vault connection in iLogic (and LifecycleInfo)](https://forums.autodesk.com/t5/inventor-programming-ilogic/get-vault-folder-properties/m-p/12440442/highlight/true#M161489)

# Tools that might get handy sometime..

[Process monitor. (Find registry values set by Inventor.)](https://forums.autodesk.com/t5/inventor-programming-ilogic/set-dockablewindows-dock-and-size/m-p/12291116/highlight/true#M158839)

Some settings are not availible by the API. But might be availible thrue the registry:
**Computer\HKEY_CURRENT_USER\Software\Autodesk\Inventor\RegistryVersion30.0\System\Preferences**