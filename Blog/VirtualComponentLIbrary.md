
# Virtual Component LIbrary

Yesterday, I scrolled down the "[Inventor Ideas](https://forums.autodesk.com/t5/inventor-ideas/idb-p/inventor-ideas-en)" board and found this idea: "[Virtual Component Library](https://forums.autodesk.com/t5/inventor-ideas/virtual-component-library/idi-p/3692736)". It has 112 votes, so I guess that people would like this feature in Inventor. However, this idea was posted in 2012 and is still gathering support. Probably it will not be implemented any time soon. But with a small I logic rule you can implement this functionality yourself.

Just to be clear: **Virtual Components** are placeholders used in assemblies to represent parts or subassemblies that do not exist as modelled files. They are helpful for adding materials to the BOM but are not modelled. Like glue or sticky tape. Creating the "Virtual Components" is not a big deal. But if you have to add them often then it would be more efficient to save them in a library. It would save you the time of creating the "Virtual Components" and if you share the library then it also helps to have consistent data in you BOM.

I want to propose an implementation similar to Autodesk's "Sketched Symbol Library". For those unfamiliar with "Sketched Symbol Library". The library is nothing more than a drawing file with all the sketched symbols that you need. Then in your drawing, you use a tool to select and copy "Sketched symbols" from the library to your drawing. The same thing can also be done for "Virtual Components".

So to start with you need to create an assembly, which will become your library. And [create/add "Virtual](https://www.youtube.com/watch?v=DZpUqZ7XH8k&t=350s) Components" in that assembly. When you are finished you can save the library (assembly file) to a location that you like. (I would suggest to save it along with all other design data. Something like: "C:\Users\Public\Documents\Autodesk\Inventor 2025\Design Data\Virtual Component Library\library.iam".)

The only thing left is to save the following code in an external iLogic rule. when you need a "Virtual Components" in an assembly then run this. The rule lets you select a "Virtual Components" from your library and will copy the selected "Virtual Components"  to your active assembly.

```vb.net
Dim libDoc = ThisApplication.Documents.Open("C:\Users\Public\Documents\Autodesk\Inventor 2025\Design Data\Virtual Component Library\library.iam", False)
Dim libDef As AssemblyComponentDefinition = libDoc.ComponentDefinition

Dim virtualComponents = libDef.Occurrences.Cast(Of ComponentOccurrence).Where(Function(o) TypeOf o.Definition Is VirtualComponentDefinition)
Dim names = virtualComponents.Select(Function(o) o.Name)
Dim selectedName = InputListBox("Select the virtual component to add.", names, names(0), Title := "Virtual component", ListName := "List")

If (String.IsNullOrWhiteSpace(selectedName)) then return

Dim occ = virtualComponents.First(Function(o) o.Name = selectedName)

Dim orign = ThisApplication.TransientGeometry.CreateMatrix()

Dim doc As AssemblyDocument = ThisDoc.Document
Dim def As AssemblyComponentDefinition = doc.ComponentDefinition
Dim newOcc = def.Occurrences.AddByComponentDefinition(occ.Definition, orign)
Try : newOcc.Name = occ.Name : Catch : End Try
	
libDoc.Close(True)
```
As you see this is a simple and small rule. but there is at least 1 known issue. The code will try to use the same name (in your model browser) as used in the library. But if that name is already used in your assembly it will use a generic name like "Component36:1"