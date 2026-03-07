# Assign name by code

On the "Inventor iLogic, API & VBA Forum" someone started the topic "[Bug report when setting Face Name](https://forums.autodesk.com/t5/inventor-programming-forum/bug-report-when-setting-face-name/m-p/10674756/highlight/false#M129786)". I could reproduce a problem and I got a nice exception "**The method or operation is not implemented**". I expected that implementing the function "**SetName**" myself would not be that complicated.

But I was wrong on a couple of points. After I did all my research and I was writing this article I discovered that I made a typo in my ilogic code and that resulted in the exception. I should have looked better at my code I just assumed this was the "bug". (You have to admit that it is a strange exception description....). But that was not the case there is something else. You can use the code below to set a name to a face. But it will only work when you run this from iLogic.

```vb.net
Dim doc As PartDocument = ThisDoc.Document
Dim face As Face = ThisApplication.CommandManager.Pick(
            SelectionFilterEnum.kPartFaceFilter, "Select a face")
Dim name As String = InputBox("Give a name", "Name", "Face ")

'----------------------------------------------------
' First option to get the iLogic automation object
' that should also work in an external applications
Const iLogicAddinGuid As String = "{3BDD8D79-2179-4B11-8A5A-257B1C0263AC}"
Dim addin As ApplicationAddIn = ThisApplication.ApplicationAddIns.ItemById(iLogicAddinGuid)
Dim iLogicAuto As Object = addin.Automation
'----------------------------------------------------
' Second option that only works within iLogic.
' Dim iLogicAuto = iLogicVb.Automation
'----------------------------------------------------

Dim namedEntities As Object = iLogicAuto.GetNamedEntities(doc)
namedEntities.SetName(face, name)
```

When you run this code in an external application you will get a **System.ArgumentException: 'Controls created on one thread cannot be parented to a control on a different thread.'** That is the exact same exception as the topic starter got on the forum. (At least when you assign the first name to a face.) Because this exception is only thrown when you use the code above "Outside-of-process" I'm not sure if you should call it a bug. (For more information about Running In-Process vs. Out-of-Process I recommend looking at the article "[Improving Your Program’s Performance](../AddinPerformance.md)") iLogic and addin code are usually run In-Process and then you will not have this problem (but better performance). So it's not convenient but maybe Autodesk gets better performance this way. If so I would agree on making such a function not available "Out-of-process".

The second thing I was wrong about: It's that it was more complicated than I thought when I started. To name a face you need to create/change at least 3 (4 if you also want to make the named face visible with a label). The first is easy just add an attribute to the face and set the name. The second is a bit more complicated you will need to add/change an attribute in the document. That attribute stores the (order of) named entities and you need to add the new name to the list. If you don't it will not be shown in the "Geometry" tab. The third is the most complicated. It's value looks like this: "EAIAABAAAABZAQAAAAAAAIAAAAAAAAAA". It looks like some random letters. But I remembered a post by Brian Ekins called "[Understanding Reference Keys in Inventor](https://modthemachine.typepad.com/my_weblog/2015/09/understanding-reference-keys-in-inventor.html)".

Some time ago I played around with those "Reference Keys" and remembered that they looked similar. But when I create the "Reference Key" it was way longer than the one that was created by iLogic. After some digging around I found that the "Reference Key" is not linked to the face itself but to the feature that created the face. With all this information I could write the iLogic rule below. There is only 1 problem with assigning a name to a face with your own code. Your newly create named face will not show up in the "Geometry tab" until you close and reopen the document. That is why this rule closes and reopens your document but you can leave that part of the code away.

```vb.net
Sub Main()
    Dim doc As PartDocument = ThisDoc.Document

    Dim face As Face = ThisApplication.CommandManager.Pick(SelectionFilterEnum.kPartFaceFilter, "Select a face")
    Dim name As String = InputBox("Give a name", "Name", "Face ")

    addNameToFace(doc, face, name)

    Dim fileName = doc.FullFileName
    doc.Save()
    doc.Close()
    ThisApplication.Documents.Open(fileName)

End Sub

Private Sub addNameToFace(doc As PartDocument, face As Face, name As String)

    Dim attSetName As String = "iLogicEntityNameSet"
    Dim attName As String = "iLogicEntityName"
    Dim att As Attribute = getAttribute(face, attSetName, attName)
    att.Value = name

    Dim refKey() As Byte = New Byte() {}
    face.CreatedByFeature.GetReferenceKey(refKey)
    Dim refKeyString = doc.ReferenceKeyManager.KeyToString(refKey)

    Dim attFeatureName As String = "iLogicEntityNameFeatureName"
    Dim attFeature As Attribute = getAttribute(face, attSetName, attFeatureName)
    attFeature.Value = refKeyString


    Dim attSetNameDoc = "iLogicEntityNameSet"
    Dim attNameDoc = "iLogicEntityNamesOrdered"
    Dim attDoc As Attribute = getAttribute(doc, attSetNameDoc, attNameDoc)

    If (attDoc.Value.contains(name) = False) Then

        Dim atts As AttributesEnumerator = doc.AttributeManager.FindAttributes(attSetName, attName)

        Dim entities As String = ""
        For Each itemAtt As Attribute In atts
            entities = entities & itemAtt.Value & Constants.vbTab
        Next
        attDoc.Value = entities
    End If
End Sub

Public Function getAttribute(objectWithAttributes As Object, attSetName As String, attName As String) As Attribute
    Dim attSet As AttributeSet

    If (objectWithAttributes.AttributeSets.NameIsUsed(attSetName)) Then
        attSet = objectWithAttributes.AttributeSets.Item(attSetName)
    Else
        attSet = objectWithAttributes.AttributeSets.Add(attSetName)
    End If

    Dim att As Attribute
    If attSet.NameIsUsed(attName) Then
        att = attSet.Item(attName)
    Else
        att = attSet.Add(attName, ValueTypeEnum.kStringType, "")
    End If
    Return att
End Function
```