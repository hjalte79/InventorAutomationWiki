
# MultiSelectListBox (based on iLogic's InputListBox)

Just as the title indicates, there is a way to 'sort of' change the functionality of an InputListBox, so that you can select multiple items at once.  No more having to use a regular InputListBox within a loop, and only being able to select a single item each time until you escape the loop.

## How I discovered this:

When you type in "InputListBox" into an iLogic rule, you see that it originates from something called "[RunDialogs](https://help.autodesk.com/view/INVNTOR/2024/ENU/?guid=4e96889c-ad00-ed08-051c-0b707ceef71f)".  If you type "RunDialogs" directly into an iLogic rule, you see that it represents a Module ([Link](https://learn.microsoft.com/en-us/dotnet/visual-basic/language-reference/statements/module-statement), [Link](https://learn.microsoft.com/en-us/dotnet/visual-basic/language-reference/modules)) within "[Autodesk.iLogic.RunTime](https://help.autodesk.com/view/INVNTOR/2024/ENU/?guid=a6157080-2be4-1527-6304-7475a35085dc)".  When you type "Autodesk.iLogic.RunTime" directly into an iLogic rule, you can see that it represents a Namespace ([Link](https://learn.microsoft.com/en-us/dotnet/visual-basic/language-reference/statements/namespace-statement), [Link](https://learn.microsoft.com/en-us/dotnet/visual-basic/programming-guide/program-structure/namespaces)).  With that in mind, I started from that directly typed in "Autodesk.iLogic.RunTime" (unquoted), I could see many other things under it.  By the way, you may also notice "GenericRadioButtonBox" listed in there too, which as you might have guessed, is the similar way to get more control over an iLogic InputRadioBox.  Those are usually created from the ""Autodesk.iLogic.RunTime.RunDialogs" Module, but you can initiate your own, using the similar path mentioned above.  The obvious advantage here is that you can access the actual [System.Windows.Forms.Form](https://learn.microsoft.com/en-us/dotnet/api/system.windows.forms.form?view=windowsdesktop-8.0) Type object that these utilize, and therefore you can further manipulate them, as you wish.  Of course you could just create your own Windows Form directly within the iLogic rule, but that may require a lot more code (and knowledge about that extra code).

Anyways, here is an example of the code used to create a custom Function I have used directly within iLogic rules, that presents an InputListBox, but allows you to select multiple items.  You will notice however, the 'Return' Type being an [IEnumerable(Of Object)](https://learn.microsoft.com/en-us/dotnet/api/system.collections.generic.ienumerable-1?view=net-7.0), since the InputListBox returned a single [Object](https://learn.microsoft.com/en-us/dotnet/visual-basic/language-reference/data-types/object-data-type).  I also did not include the last two options Height & Width, because I have never used them, and have never seen anyone else use them, but feel free to copy this and create your own variations that to include them.  Or maybe don't even use it as a custom Function, and just use the Function's interior code directly if you want, but the input parameters are in a different order, not that it matters.

```vb.net
Sub Main
	Dim oList As New List(Of String) From {"Option 1", "Option 2", "Option 3" }
	Dim Selected As IEnumerable(Of Object) = MultiSelectListBox("Select Some", oList, Nothing, "Multi-Select InputListBox", "My List")
	If Selected.Count = 0 Then
		MessageBox.Show("Nothing was selected from the list.", "None Selected",MessageBoxButtons.OK, MessageBoxIcon.Asterisk)
	Else
		For Each Item In Selected
			MessageBox.Show("You selected:  " & Item.ToString, "Selected", MessageBoxButtons.OK, MessageBoxIcon.Asterisk)
		Next
	End If
End Sub

Function MultiSelectListBox(Optional Instructions As String = vbNullString, Optional Items As IEnumerable = Nothing,
Optional DefaultValue As Object = Nothing, Optional Title As String = vbNullString, Optional ListName As String = vbNullString) As IEnumerable(Of Object)
	Using oILBD As New Autodesk.iLogic.Runtime.InputListBoxDialog(Title, ListName, Instructions, Items, DefaultValue)
		Dim oLB As System.Windows.Forms.ListBox = oILBD.Controls.Item(0).Controls.Item(2)
		oLB.SelectionMode = System.Windows.Forms.SelectionMode.MultiSimple
		Dim oDlgResult As System.Windows.Forms.DialogResult = oILBD.ShowDialog()
		Dim oSelected As IEnumerable(Of Object) = oLB.SelectedItems.Cast(Of Object)
		Return oSelected
	End Using
End Function
```

This is why you could never find an InputListBox or InputRadioBox anywhere else within the general vb.net stuff...because they are unique to the iLogic [ApplicationAddIn](https://help.autodesk.com/view/INVNTOR/2024/ENU/?guid=GUID-ApplicationAddIn), and its associated resources.

(Original text was posted on the Inventor "[iLogic and VB.net Forum](https://forums.autodesk.com/t5/inventor-programming-forum/multiselectlistbox-based-on-ilogic-s-inputlistbox/td-p/12039714)")

## About the Author:
Wesley Crihfield is a design Engineer, design Automation and CNC Programmer. He has been working in engineering since the early 2000's. He primarily uses Autodesk Inventor Professional but has used Autodesk software since AutoCAD R12, back in the mid 90's. He's also very active on the Autodesk forums and even won prices for the number of solutions he contributed to others. More about can be found on his [profile page](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/7812054)

### credit

By the way, just to give credit where credit is due...

Although I had explored into similar aspects & resources before, I was inspired to dig further into these uniquely iLogic forms by a post here on this forum a while back by [@Curtis_Waguespack](https://forums.autodesk.com/t5/user/viewprofilepage/user-id/105031) at this thread that I participated in. [https://forums.autodesk.com/t5/inventor-ilogic...](https://forums.autodesk.com/t5/inventor-programming-forum/delete-forms-with-rule/m-p/11187072/highlight/true#M138420) Kudos to Curtis for being an inspiration to me and many others over the years.