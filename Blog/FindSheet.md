# Finding a sheet objects in a list. 

While writing iLogic rules I find my self often in the situation that I need to find a object in a list. Often I can use a standard Inventor api functions. But in some cases they do not work or the function just does not exists. For example I have the following situation. While generating drawings with our product configurator. An iLogic rule adds dimensions to an idw with many sheets. Ofcourse in that ILogic rule I need to select the correct sheet (in a list of sheets).

I have 2 options. Use the iLogic api and call the function:

```vb.net
Dim sheets As IManagedSheets = ThisDrawing.Sheets
Dim sheet As IManagedSheet = sheets.ItemByName("MySheetName:15")
```
_(Notic the types start with IManaged… that is a sign we use the iLogic api)_

As you probably can guess this number will also change when a sheet is removed. Therefor this also doesn’t work for me. The most obvious solution would be to loop over all sheets and check each name. something like this:

```vb.net
Dim sheets As Sheets = ThisDrawing.Document.Sheets
Dim sheet As Sheet = Nothing
For Each currentSheetInLoop As sheet In sheets
    If (currentSheetInLoop.Name.Contains("MySheetName")) Then
        sheet = currentSheetInLoop
        Exit For
    End If
Next
```

But there is a more **elegant solution** that does exactly the same thing:

```vb.net
Dim sheet As Sheet = ThisDrawing.Document.Sheets.
	Cast(Of Sheet).
	Where(Function(s) s.Name.Contains("MySheetName")).
	FirstOrDefault()
```

 I think this is more readable then the whole loop also it’s more flexible. In this 4 lines I have chained a lot of functions/properties one after the other.  Usaly i would not advice to do this because its dificult to debug, but in this case there isn't much that can go wrong. (I could have done this in 1 line but that would have been less readable.) Lets have a look what is going on here and breaking it down in separate lines. 

 ```vb.net
 'Get the list of sheets Inventor api style 
Dim sheets As Sheets = ThisDrawing.Document.Sheets

'Cast the list to the type IEnumerable list. 
'IEnumerable list is a standard vb.Net type. With the some greate (extension) functions like "Where" 
Dim allSheets As IEnumerable(Of Sheet) = sheets.Cast(Of Sheet)

'Here we use the function "Where" to find all sheets that 
'comply with the following statement: Sheet._DisplayName.Contains("MySheetName")
'think of this as the statement in an "If" function.
Dim sheetsSubSet As IEnumerable(Of Sheet) = allSheets.Where(Function(s) (s.Name.Contains("MySheetName")))
'Notice that this is also an IEnumerable list. 
'The "Where" function is like a filter on a list that returns a list. 
'If you want you could use multiple filters here ;-)

'If your filter is correct you will end up with a list of 1 item.
'You will get the first item of the list like this. 
'That has to be the item/sheet that you where looking for
Dim sheet As Sheet = sheetsSubSet.FirstOrDefault()
 ```

 If you want to test this function, I would suggest the following code:

 ```vb.net
 Dim sheet As Sheet = ThisDrawing.Document.Sheets.
	Cast(Of Sheet).
	Where(Function(s) s.Name.Contains("MySheetName")).
	FirstOrDefault()

If (sheet Is Nothing) Then
    MsgBox("Sheet not found")
Else 
    MsgBox(sheet.Name)
End If
 ```

