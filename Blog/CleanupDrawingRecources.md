
# Clean up drawing recources.

A user on the [forum](https://forums.autodesk.com/t5/inventor-programming-forum/ilogic-code-to-purge-all-un-used-drawing-recourses/m-p/12846871) posted the code below. He uses it to clean up his drawing resources. He wrote, "It appears to work nicely but I'd be grateful if someone with actual coding knowledge could cast an eye over it". I pretend to have some knowledge. And I started commenting on his code. In the end, I realised, it could benefit everyone.

This was the code by the forum user:
```vb.net
Dim oDrawDoc As DrawingDocument
oDrawDoc = ThisDoc.Document 

'Delete un-used Sheets
Dim oSheets As Sheets = oDrawDoc.Sheets
For Each oSheet As Sheet In oSheets
	If oSheet.DrawingViews.Count = 0 Then
  	oSheet.Delete
	End If  
Next

'Delete Sheet Formats
Dim oSheetFormatDef As SheetFormat
For Each oSheetFormatDef In oDrawDoc.SheetFormats
	 oSheetFormatDef.Delete
Next

'Delete unused TitleBlocks
Dim oTitleBlock As TitleBlockDefinition
For Each oTitleBlock In oDrawDoc.TitleBlockDefinitions
	If oTitleBlock.IsReferenced = False  Then
	oTitleBlock.Delete
	End If
Next 

'Delete unused Borders
Dim oBorder As BorderDefinition 
For Each oBorder In oDrawDoc.BorderDefinitions
	If oBorder.IsReferenced = False  And oBorder.IsDefault = False Then
	oBorder.Delete
	End If
Next 

'Delete un-used Sketch Symbols
Dim oSketchedSymbolDef As SketchedSymbolDefinition
For Each oSketchedSymbolDef In oDrawDoc.SketchedSymbolDefinitions
	If oSketchedSymbolDef.IsReferenced = False Then
  	oSketchedSymbolDef.Delete
	End If  
Next
```
I am not saying this is better but I can do the same in 6 lines of code.
```vb.net
Dim doc As DrawingDocument = ThisDoc.Document
doc.Sheets.Cast(Of Sheet).Where(Function(s) s.DrawingViews.Count = 0).ToList().ForEach(Sub(s) s.Delete())
doc.SheetFormats.Cast(Of SheetFormat).ToList().ForEach(Sub(s) s.Delete())
doc.TitleBlockDefinitions.Cast(Of TitleBlockDefinition).Where(Function(t) Not t.IsReferenced).ToList().ForEach(Sub(t) t.Delete())
doc.BorderDefinitions.Cast(Of BorderDefinition).Where(Function(b) Not b.IsReferenced And Not b.IsDefault).ToList().ForEach(Sub(b) b.Delete())
doc.SketchedSymbolDefinitions.Cast(Of SketchedSymbolDefinition).Where(Function(s) Not s.IsReferenced).ToList().ForEach(Sub(s) s.Delete())
```

In fact, this code is terrible. it's hard to read and very hard to debug but it works! My point: I pretend to have some knowledge but coding styles are mostly based on opinions. And like each opinion, you don't have to agree with me but take from it what you like ;-) Let's have a serious look at the code.

**Lines 1, 2**

On line 1 the variable "oDrawDoc" is declared. Then on line 2, the variable is set. This can be done on one line and in my opinion easier to read. If something can be done in one line and you use 2 then I consider it clutter. (But it is also possible to exaggerate this. Then you get a situation like the example above.) I also understand why people do this. Most example codes you will find online (and the Autodesk help files) are written voor VBa (not VB.Net/iLogic.) The issue is that combining those 2 lines was not possible in the VBa era. So, most examples you find are outdated. (Just like [VBa is obsolete](http://www.hjalte.nl/44-is-vba-obsolete) and I find it terrible that Autodesk is still creating examples based on VBa. But that's a whole other discussion...) So I can understand why people learned to code like this.

```vb.net
' So instead of this:
'      Dim oDrawDoc As DrawingDocument
'      oDrawDoc = ThisDoc.Document
' I would use:
Dim doc As DrawingDocument = ThisDoc.Document
```

**Lines 5, 6**

only in this "for each" loop items are saved in memory/variable (the "oSheets" variable). So outside of the "for each" loop. In theory, this is better because of "References" calls Instead of "Inline" Calls. This means that only 1 "References" call is made to the Inventor API and the information is saved in a variable. "Inline" calls are used In the other "for each" loop. If you do this you make a call to the inventor api each time you iterate 1 item of the list. That is not as fast as reading from memories using "References" calls. "References" calls can be almost 50% faster. (have a look at this blog post "Improving Your Program’s Performance" chapter: "Use References Instead of Inline Calls".) However for this rule that is a theoretical improvement. I guess most drawings have only a couple of sheets. So the improvement is probably only a few milliseconds. So you could argue that this is clutter because it could be done in 1 line.

```vb.net
' So instead of this:
'      Dim oSheets As Sheets = doc.Sheets
'      For Each oSheet As Sheet In oSheets
' I would use:
For Each oSheet As Sheet In doc.Sheets
```

**Lines 13, 14**

The variable oSheetFormatDef was declared on line 13 and set on line 14. This was (again) the only way in the VBa era... Same problems as I have with lines 1, 2 and I would remove the clutter.
```vb.net
' So instead of this:
'      Dim oSheetFormatDef As SheetFormat
'      For Each oSheetFormatDef In oDrawDoc.SheetFormats
' use this:
For Each oSheetFormatDef As SheetFormat In doc.SheetFormats
```

**Line 21**

Some people would argue that you should not check if a value is "False". (Especially automated coding style check tools don't like this.) You could use the "Not" keyword here. This is a pure readability issue and maybe this is something for you to consider.

```vb.net
' So instead of this:
'      If oTitleBlock.IsReferenced = False Then
' consider using this:
If Not oTitleBlock.IsReferenced Then
```

Lines 21 to 23

The most annoying thing I see in code is wrongly used indentation. I know it is a pure readability thing but... (If it's done wrong, I will not understand the code until I have fixed it.)

```vb.net
' So instead of this:
'      If Not oTitleBlock.IsReferenced Then
'      oTitleBlock.Delete() ' Missing indentation here
'      End If
' do this:
If Not oTitleBlock.IsReferenced Then
	oTitleBlock.Delete()
End If
```

**Line 29**

I'm missing parentheses in the if statements. Also in all other if statements but here is the problem more obvious. (As far as I know, only vb.net allows this.) Using parentheses helps to read what is checked (and in which order).
```vb.net
' So instead of this:
'      If oBorder.IsReferenced = False And oBorder.IsDefault = False Then
' use this:
If ((oBorder.IsReferenced = False) And (oBorder.IsDefault = False)) Then
```

**Variable Naming**
The last point is also an inheritance from the VBa era. In that era, the "Hungarian Notation" was popular for naming variables. (Hungarian Notation: This notation describes the variable type or purpose at the start of the variable name, followed by a descriptor that indicates the variable’s function.) That is why variables in old VBa code usually start with the "o" to indicate that it is an object. That is not the standard any more and because new coders don't know why this is done it's used wrong. For example have a look at this variable name: "**o**ParameterValue". The type of parameter values in inventor is always double. Therefore in correct "Hungarian Notation" would be: "**d**ParameterValue". Doing this worng could confuse readers of your code. Therefore I would suggest complying with industry standards and using camelCasing for variable. (camelCasing: Words are delimited by capital letters, except the initial word.)

Probably over kill but if you would like to look at a good industry standard for coding styles you might want to have a look at the styles from [Microsoft](https://learn.microsoft.com/en-us/dotnet/standard/design-guidelines/naming-guidelines).

**Result**

This is how it looks like in the end after all the changes.

```vb.net
Dim doc As DrawingDocument = ThisDoc.Document

'Delete un-used Sheets
For Each sheet As Sheet In doc.Sheets
	If (sheet.DrawingViews.Count = 0) Then
		sheet.Delete()
	End If
Next

'Delete Sheet Formats
For Each sheetFormat As SheetFormat In doc.SheetFormats
	sheetFormat.Delete()
Next

'Delete unused TitleBlocks
For Each titleBlockDefinition As TitleBlockDefinition In doc.TitleBlockDefinitions
	If (Not titleBlockDefinition.IsReferenced) Then
		titleBlockDefinition.Delete()
	End If
Next

'Delete unused Borders
For Each borderDefinition As BorderDefinition In doc.BorderDefinitions
	If ((Not borderDefinition.IsReferenced) And (Not borderDefinition.IsDefault)) Then
		borderDefinition.Delete()
	End If
Next

'Delete un-used Sketch Symbols
For Each sketchedSymbolDefinition As SketchedSymbolDefinition In doc.SketchedSymbolDefinitions
	If (Not sketchedSymbolDefinition.IsReferenced) Then
		sketchedSymbolDefinition.Delete()
	End If
Next
```