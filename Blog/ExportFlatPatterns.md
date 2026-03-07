# Export Flat Patterns iPart

I’ve recently seen several questions from people looking to save or export files automatically.  Questions like “How to perform ‘save a copy as’ PDF for a list of Inventor drawings from an excel?” and “Automate iPart flat pattern export to DXF”. That is why I wrote this post, explaining how you can write your own iLogic export rules.

In this post I will try to explain my thoughts while writing an export rule. As an example I will use the question: “[Automate iPart flat pattern export to DXF](https://forums.autodesk.com/t5/inventor-programming-forum/automate-ipart-flat-pattern-export-to-dxf/m-p/9928972#M118994)”

These questions have 3 things in common. There is a list of drawings or models. 1 or more items in the list must be saved or exported. The name of the new file must depend on 1 or more properties of the drawings or models.

Generally, 3 functions are needed to create an export rule. The main function with a loop. The second function for creating the file name. The third function that will do the saving/exporting of the file(s). The basic rule will look something like this:

```vb.net
Sub main()
	For Each item As Object In collectionOfObjects
		Dim newFileName As String = createFileName()
		export(newFileName)
	Next
End Sub

Function createFileName() As String
	' here we will create a file name
End Function

Function export(newFileName As String)
	' here we will export stuff
End Function
```

Let’s start with the easy bit. There are lots example codes on how you export/save a file (or sheet). On this site alone there are several posts about exporting files. A quick google with search terms like “Inventor API export DXF” and you will find what you need. I happened to find [a post on this site](https://clintbrown.co.uk/2018/12/09/dxf-export/) by Clint. After I had a look, I found the part that does the exporting. With that I could finish the export function. For testing the function, I wrote this rule:

```vb.net
Sub main()
	export(ThisDoc.Document, "c:\temp\exportTest.dxf")
End Sub

Function export(doc As PartDocument, newFileName As String)
	Dim oCompDef As SheetMetalComponentDefinition = doc.ComponentDefinition
	If oCompDef.HasFlatPattern = False Then
		oCompDef.Unfold()
 	Else
 		oCompDef.FlatPattern.Edit
 	End If
	
	Dim sOut As String = "FLAT PATTERN DXF?AcadVersion=2000&OuterProfileLayer=IV_INTERIOR_PROFILES"
	oCompDef.DataIO.WriteDataToFile(sOut, newFileName)
    	oCompDef.FlatPattern.ExitEdit()
End Function
```

For the next function “createName()”. The iLogic GUI gives us lots of examples/snippets about extracting properties. But often the original filename should be a part of the new file name. The “System.IO” namespace of windows has some great functions for that. Also, we need a way to create the full path. Often, I see code like this:

```
" ... " & someVariable & " - " & someOtherVariable & ".dxf"
```

That is not wrong and will work. But there is a cleaner way to write this. The “string”-object has a static/shared function for doing that. If I put it all together, I get this test rule:

```vb.net
Sub main()
	MsgBox(createFileName(ThisDoc.Document))
End Sub

Function createFileName(doc As PartDocument) As String
	Dim dirName As String = IO.Path.GetDirectoryName(doc.FullFileName)
	Dim fileNameWithoutExtension As String = IO.Path.GetFileNameWithoutExtension(doc.FullFileName)
	Dim material As String = iProperties.Material
	Dim thickness As Double = Parameter("Thickness") 
	' points in filenames can give problems
	' therefore I did round it to a hole number
	' by multiplying it by 10 a sheet of 1.5mm 
	' will Not Get same file name as a sheet of 1mm
	thickness = Math.Round(thickness * 10)
	Dim thicknessString As String = thickness.ToString()
	Return String.Format("{0}\{1} - {2}.dxf", 
		dirName, fileNameWithoutExtension, thicknessString)		
End Function
```

The main function is all about looping over a list and filtering out what you don’t need.
Typically, you would get for IDW files something like this:

```vb.net
Sub main
	Dim doc As DrawingDocument = ThisDoc.Document
	' if you wish to export each sheet
	For Each sheet As Sheet In doc.Sheets
		Dim newFileName As String = createFileName(Sheet)
		export(Sheet, newFileName)
	Next

	' if you wish to export the associated document
	For Each sheet As Sheet In doc.Sheets
		Dim refrenceDoc As Document = Sheet.DrawingViews.Item(1).ReferencedDocumentDescriptor.ReferencedDocument
		Dim newFileName As String = createFileName(refrenceDoc)
		export(refrenceDoc, newFileName)
	Next
End Sub
```

Typically, you would get for IAM files something like this (with some typical filters):

```vb.net
Sub Main()
	Dim doc As AssemblyDocument = ThisDoc.Document

	For Each refDoc As Document In doc.AllReferencedDocuments
		' filter out files is in librarie folders 
		If (refDoc.FullFileName.Contains("\librarie\")) Then Continue For
		' filter sheet metal parts
		If (refDoc.SubType <> "{9C464203-9BAE-11D3-8BAD-0060B0CE6BB4}") Then Continue For

		Dim newFileName As String = createFileName(refDoc)
		export(refDoc, newFileName)
	Next
End Sub
```

But for this example, we are going to loop over all rows in an iPart table. The code below is also the complete rule for exporting all flat patterns of an iPart.

```vb.net
Sub Main()
	Dim doc As AssemblyDocument = ThisDoc.Document
	If (doc.ComponentDefinition.IsiPartFactory = False) Then
		MsgBox("This is not an iPart factory")
	End If
		
	Dim fact As iPartFactory = doc.ComponentDefinition.iPartFactory
	For Each row As iPartTableRow In fact.TableRows
		fact.DefaultRow = Row
		doc.Update()
		doc.Update2(False)

		Dim newFileName As String = createFileName(doc)
		export(doc, newFileName)
	Next
End Sub

Function createFileName(doc As PartDocument) As String
	Dim dirName As String = IO.Path.GetDirectoryName(doc.FullFileName)
	Dim fileNameWithoutExtension As String = IO.Path.GetFileNameWithoutExtension(doc.FullFileName)
	Dim material As String = iProperties.Material
	Dim thickness As Double = Parameter("Thickness") 
	' points in filenames can give problems
	' therefore I did round it to a hole number
	' by multiplying it by 10 a sheet of 1.5mm 
	' will Not Get same file name as a sheet of 1mm
	thickness = Math.Round(thickness * 10)
	Dim thicknessString As String = thickness.ToString()
	Return String.Format("{0}\{1} - {2}.dxf", 
		dirName, fileNameWithoutExtension, thicknessString)		
End Function

Function export(doc As PartDocument, newFileName As String)
	Dim oCompDef As SheetMetalComponentDefinition = doc.ComponentDefinition
	If oCompDef.HasFlatPattern = False Then
		oCompDef.Unfold()
 	Else
 		oCompDef.FlatPattern.Edit
 	End If
	
	Dim sOut As String = "FLAT PATTERN DXF?AcadVersion=2000&OuterProfileLayer=IV_INTERIOR_PROFILES"
	oCompDef.DataIO.WriteDataToFile(sOut, newFileName)
    	oCompDef.FlatPattern.ExitEdit()
End Function
```