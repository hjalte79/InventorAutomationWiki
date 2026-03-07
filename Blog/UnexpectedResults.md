# Unexpected Results

If you have read any of my blogs post before, you’ll know that I like to challenge myself to solve other Inventor user’s problems. I usually find these problems on the Inventor Customization Forum. A while back, I came across an interesting problem. This user wanted to change the iProperties of a part, based on it’s thickness and material.

The rule looked something like this.

```vb.net
Dim thickness As Double = Parameter.Value("Thickness")
Dim material As String = iProperties.Material

If (material = "Stainless steel") Then
	Select Case thickness
		Case 2
			iProperties.Value("Project", "Part Number") = 100
		Case 3
			iProperties.Value("Project", "Part Number") = 101
		Case Else
			MsgBox("Couldn’t set part number for stainless steel.")
	End Select
Else If (material = "Copper") Then
	Select Case thickness
		Case 2
			iProperties.Value("Project", "Part Number") = 103
		Case 3
			iProperties.Value("Project", "Part Number") = 104
		Case Else
			MsgBox("Couldn’t set part number for copper")
	End Select
Else
	MsgBox("Unknown material: " & material)
End If
```

It’s a very straightforward rule, at first glance, there’s nothing wrong with the code, but there are actually 2 problems.

1. If the user selects Stainless Steel (as the material) an error message pops up “Unknown material: stainless Steel”
2. If the user selects Copper (as the material) and sets the thickness to 3mm, an error message pops up ”Couldn’t set part number for Copper 3mm”
The first problem was an easy fix ,“steel” in “Stainless Steel” should have been written with a capital letter. Not something that I would like to write about but there is an elegant solution to these kinds of problems that I would like to share. Vb.net has a built in function for comparing strings and ignoring capital cases. Therefore I don’t think the solution here is to write the title correctly, but use this function. The if-statement will look like this.

```
If (material.Equals("Stainless steel", StringComparison.InvariantCultureIgnoreCase)) Then
```

It is a bit longer to write, but if you teach yourself to use this function, then you would never have to worry about incorrect capitalisation..

The second problem is more interesting. Inventor (in fact the computer) thinks that the variable “Thickness” which is 3 is not the same as the 3 in the case statement. It’s important to notice the 3 in the case statement is not declared therefore the computer assumes its an integer. (if it was written as 3.0 the computer would have assumed it’s a double.) And the variable “Thickness” is declared as double. Floating point numbers (like Double) can lead to unexpected behavior if you try to compare them with other numbers. This is caused by the fact that a floating point numbers don’t have a precise representation in memory. They are stored as binary fractions. For example the number “3” can be represented in memory as “3.00000000000000000000000001”. If you compare that with the integer 3 (which has an exact representation in memory) the computer will evaluate this as **not** the same number.

This problem is more common than you would expect and is something that you should look for when writing iLogic rules. Try creating a part with a parameter called Thickness and run the following rule.

```vb.net
Dim falseResults = 0
For i = 1 To 100
	' change model parameter to the Double ‘i’
	Parameter("Thickness") = i	

	' compare parameter against the integer ‘i’
	If (Parameter("Thickness") <> i) Then
		Dim msg = String.Format("{0} is not {1} ?!?", 
			i, Parameter("Thickness"))
		MsgBox(msg)
		falseResults = falseResults + 1
	End If
Next i
MsgBox("Number of false results: " & falseResults)
```

This rule sets the parameter “Thickness” a 100 times. Then compares the set value with the value of the parameter. (Parameters are always of the type Double.) That should always be the same but I get 14 false results. (With your computer/settings the result may be different)

Because of this imprecision, there is always the risk that you get unexpected results when comparing floating points. The Microsoft documentation has suggestions on [how you can handle this kind of problem](https://docs.microsoft.com/en-us/dotnet/visual-basic/programming-guide/language-features/data-types/troubleshooting-data-types).  But conveniently enough iLogic has a [built in function](https://help.autodesk.com/cloudhelp/2019/RUS/Inventor-iLogic/iLogic_API/html/66b38abb-950a-3396-44be-6001911188e1.htm) that can help you to overcome this problem. “**DoubleUtil.DoublesAreEqual(value1, value2)**”. This function will tests if two numbers are equal within 6 decimal digits. If we use this function in the previous rule then you will see all results are correct. Try it.

```vb.net
Dim falseResults = 0
For i = 1 To 100
	' change model parameter to the Double ‘i’
	Parameter("Thickness") = i	

	' compare parameter against the integer i
	If DoubleUtil.DoublesAreEqual(Parameter("Thickness"), i) = False Then
		Dim msg = String.Format("{0} is not {1} ?!?", 
			i, Parameter("Thickness"))
		MsgBox(msg)
		falseResults = falseResults + 1
	End If
Next i
MsgBox("Number of false results: " & falseResults)
```