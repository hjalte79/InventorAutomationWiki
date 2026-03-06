# Manipulating custom IProperties
In my [last post](./AccessingIProperties.md), Accessing iProperties, we explored the basics of reading iProperties in Inventor. Now it’s time to go a level deeper. In this article, we’ll look at how to create, edit, and manage **custom** iProperties — both with iLogic and with the more powerful Inventor API.

iLogic makes simple tasks quick and easy, but when you need full control, the Inventor API offers far more flexibility. Let’s explore the differences and see how to work effectively with both.

## Using iLogic

In the previous post, we looked at how to **read and write custom iProperties using iLogic**, and the same approach can also be used to create new custom iProperties. In fact, iLogic will automatically generate a custom iProperty the moment you write a value to a property name that doesn’t already exist. While this makes iLogic very easy and flexible to use, it also introduces a subtle risk: if you accidentally mistype a property name or assume a property already exists, Inventor won’t warn you, it will simply create a new custom iProperty. This silent behavior can lead to unintended or duplicate properties. By contrast, when using the **Inventor API**, this situation cannot occur, because the API requires you to explicitly create new custom iProperties before assigning values to them.

## Creating Custom iProperties Using the Inventor API

When you use the Inventor API, creating a custom iProperty it’s an explicit, controlled action: you first get the property set ‘Inventor User Defined Properties’ and then add a property. This prevents the silent creation issues you can encounter with iLogic. If a property doesn’t exist, the API won’t let you “accidentally” write to it. In practice, you’ll need to get the property set. But then you can add a property like this:

```vb.net
' Get the active document.
Dim doc As Document = ThisApplication.ActiveDocument
' Get the PropertySets collection.
Dim propSets As PropertySets = doc.PropertySets
' Get the Design Tracking Properties property set.
Dim propSet As PropertySet = propSets.Item("Inventor User Defined Properties")
' Create property 'Website' with value 'www.hjalte.nl'
propSet.Add("www.hjalte.nl", "Website")
```

The value can be any one of the types: String (text), date, double (number), Boolean (yes/no). once set they can not change any more. From the moment you created the property, you can safely read and write the property by name, confident you’re working with a single, well-defined iProperty rather than spawning duplicates due to typos or assumptions.

## Check if Exists
Inventor will throw exceptions if you try to **create, write to, read from, or delete** an iProperty in the wrong state (e.g., creating one that already exists, or reading one that doesn’t). To avoid this, always **check existence first**. There are two common ways to do that:

Attempt to get the property and handle the exception
Try to access the property by name and wrap it in a try…catch. If it isn’t there, you’ll get an exception and can handle the “create” or “skip” path in the catch.

### Attempt to get the property and handle the exception
Try to access the property by name and wrap it in a try…catch. If it isn’t there, you’ll get an exception and can handle the “create” or “skip” path in the catch.

```vb.net
' Get the active document.
Dim doc As Document = ThisApplication.ActiveDocument
' Get the PropertySets collection.
Dim propSets As PropertySets = doc.PropertySets
' Get the Design Tracking Properties property set.
Dim propSet As PropertySet = propSets.Item("Inventor User Defined Properties")

Dim prop As [Property]
Try
    ' try getting a handle on the property
    prop = propSet.Item("RequestedProperty")
Catch ex As Exception
    ' if the code gets here than the Property does not exist.
    ' Handle this however you like
    ' For example show a messagebox and exit. Like this:
    MsgBox("Property does not exists")
    Exit Sub
End Try
' you can use the property from here
MsgBox("Property exists")
```

### Loop over the property set and search by name
Enumerate the User Defined property set, compare each property’s Name to your target, and decide based on whether you find a match.

```vb.net
' Get the active document.
Dim doc As Document = ThisApplication.ActiveDocument
' Get the PropertySets collection.
Dim propSets As PropertySets = doc.PropertySets
' Get the Design Tracking Properties property set.
Dim propSet As PropertySet = propSets.Item("Inventor User Defined Properties")

Dim prop As [Property] = propSet.Cast(Of [Property]).FirstOrDefault(Function(p) p.Name.Equals("RequestedProperty"))

If (prop Is Nothing) Then
    ' if the code gets here than the Property does not exist.
    ' Handle this however you like
    ' For example show a messagebox and exit. Like this:
    MsgBox("Property does not exists")
    Exit Sub
End If
' you can use the property from here
MsgBox("Property exists")
```

### Which is best?
For clarity and performance, prefer searching the set (option 2) over using exceptions (option 1) for control flow. Exceptions are relatively expensive and can obscure the “happy path” logic. Iterating the set (or caching the names into a lookup/dictionary once per document) keeps your intent explicit and avoids exception overhead in normal operation. Reserve the try/catch approach for truly exceptional cases, e.g., when you expect the property to exist and you want to fail fast if it doesn’t.

## Deleting Custom iProperties

Deleting a custom iProperty is straightforward when using the Inventor API, you can simply call the Delete() method on that property like this:
```vb.net
' Get the active document.
Dim doc As Document = ThisApplication.ActiveDocument
' Get the PropertySets collection.
Dim propSets As PropertySets = doc.PropertySets
' Get the Design Tracking Properties property set.
Dim propSet As PropertySet = propSets.Item("Inventor User Defined Properties")

Dim prop As [Property] = propSet.Item("RequestedProperty")
prop.Delete()
```