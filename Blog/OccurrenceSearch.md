# Universal occurrence search function

Last week some one on the “Autodesk Inventor customization” forum asked the following:

“I have this code.  It tends to work ok but I'm looking to improve the efficiency of the coding for future endeavours.  I'm want to make a method for traversing the assembly and sub's so that it can be used over and over for all various traversing code I need to write.”

He did create a recursive function that look like this:

```vb.net
Private Sub recursiveSearch(doc As AssemblyDocument)
    Dim Occurrences As ComponentOccurrences
    For Each occ As ComponentOccurrence In Occurrences
        ' Print the name Of the current occurrence.
        If occ.DefinitionDocumentType = DocumentTypeEnum.kPartDocumentObject Then
             'Do stuff with the found Occurrence
        End If
        If occ.DefinitionDocumentType = DocumentTypeEnum.kAssemblyDocumentObject Then
            Call recursiveSearch(occ.Definition.Document)
        End If
    Next
End Sub
```

I have to admit this is how I would solve this, if I need to do something to an occurrence of an (sub)assembly. But as he wrote you can’t reuse this code. For example: The code above will only find part occurrences and do 1 specific task. If you later also need a function that gets all sheetmetal parts, then you need to write a new function for that. That is also true for what the function does. If you need to do something else then you need to write a new function.

So we if we want to improve this function then we need to extract the following:

- The selection of the occurrences (requirements).
- The “Do stuff with the found Occurrence” part of the function.

Making the function do nothing is easy 😉 but in the end we want to do something. There fore I changed the function to returns only the occurrences that meet the requirements. By doing that you could use the same function for multiple actions, as log as the requirements don’t change.

But I also wanted to change the function so that i can change the “selection” part of the function. Without changing the function. For that it would be great if we could use a “function” as a parameter for a function. That is possible with delegates. This is the function that I came up with:

```vb.net
Public Function findAllWhere(doc As AssemblyDocument,
                predicate As Func(Of ComponentOccurrence, Boolean)) As IEnumerable(Of ComponentOccurrence)
    
    Dim list As IEnumerable(Of ComponentOccurrence) = doc.ComponentDefinition.
        Occurrences.Cast(Of ComponentOccurrence).
        Where(predicate)

    Dim assemblyList As IEnumerable(Of ComponentOccurrence) = doc.ComponentDefinition.
        Occurrences.Cast(Of ComponentOccurrence).
        Where(Function(o) o.DefinitionDocumentType = DocumentTypeEnum.kAssemblyDocumentObject)

    For Each assOcc As ComponentOccurrence In assemblyList
        If TypeOf assOcc.Definition Is VirtualComponentDefinition Then Continue For
        Dim assDoc As AssemblyDocument = assOcc.ReferencedDocumentDescriptor.ReferencedDocument
        Dim listToAdd As IEnumerable(Of ComponentOccurrence) = findAllWhere(assDoc, predicate)
        list = list.Concat(listToAdd)
    Next
    Return list
End Function
```

In the usage example below you will see that is call the function in 2 different ways.
In example 1 I use a [lambda expression](https://learn.microsoft.com/en-us/dotnet/visual-basic/programming-guide/language-features/procedures/lambda-expressions) to select all part occurrences in the assembly.

In example 2 I use the function “partNrContains010” as [delegate](https://learn.microsoft.com/en-us/dotnet/visual-basic/programming-guide/language-features/delegates/) function for selection all occurrences that have a part number  that contains “010”. It’s important to notice that this function takes 1 argument. An object of the type ComponentOccurrence (and not of the type ComponentOccurrences). And returns a object of the type Boolean.
In both examples I use the returned list to color the part in the list.

```vb.net
Sub Main()
    Dim doc As AssemblyDocument = ThisDoc.Document

    Dim list As IEnumerable(Of ComponentOccurrence)

    ' Usage example 1
    list = findAllWhere(doc, Function(occ) occ.DefinitionDocumentType = DocumentTypeEnum.kPartDocumentObject)
    colorPartsInList(list, "Gunmetal")
	
    ' Usage example 2
    list = findAllWhere(doc, AddressOf partNrContains010)
    colorPartsInList(list, "Brass - Satin")

    doc.Update()
End Sub

Public Function partNrContains010(occ As ComponentOccurrence) As Boolean
    Dim def As ComponentDefinition = occ.Definition
    Dim doc As Document = def.Document
    Dim propSet As PropertySet = doc.PropertySets.Item("Design Tracking Properties")
    Dim prop As [Property] = propSet.Item("Part Number")
    Dim val As String = prop.Value
    Return (val.Contains("010"))
End Function


Public Sub colorPartsInList(List As IEnumerable(Of ComponentOccurrence), colorName As String)
    For Each occ As ComponentOccurrence In List
        Dim def As ComponentDefinition = occ.Definition
        Dim doc As PartDocument = def.Document
        doc.ActiveRenderStyle = doc.RenderStyles.Item(colorName)
    Next
End Sub
```

As you see with just 1 function its possible to make all different kind of lists.