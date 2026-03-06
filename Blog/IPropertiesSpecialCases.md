# IProperties special cases

Inventor’s iProperties system includes some lesser known behaviors, especially when working with date fields and expression‑driven values, that can affect automation and data consistency. In this post, we’ll look at how Inventor treats the date properties as well as how expressions work behind the scenes in both the UI and the API.

## Working with Date iProperties in Inventor: Understanding the “Zero Date”

In my previous post [1](./AccessingIProperties.md)[2](./ManipulatingCustomIProperties.md) on working with custom iProperties, I covered how to create properties of different data types, including dates. Date properties, however, have one special behavior that’s worth a closer look. When you add a date iProperty in Inventor, you’ll notice a checkbox next to it. This checkbox determines whether the date is considered “set” or not. Here’s the key: Every date iProperty always contains a value, but one specific date is treated as a special case, the zero date.

Inventor’s zero date is: **January 1, 1601**

If a date iProperty is assigned this value, Inventor automatically unchecks the box next to it. This visually indicates that the date has not been set by the user. Below is a simple iLogic example demonstrating this behavior. The code creates 3 custom iProperties, one set to the zero date, one set to some date and one set to today.

```vb.net
' Get the active document.
Dim doc As Document = ThisApplication.ActiveDocument
' Get the PropertySets collection.
Dim propSets As PropertySets = doc.PropertySets
' Get the Design Tracking Properties property set.
Dim propSet As PropertySet = propSets.Item("Inventor User Defined Properties")

Dim zeroDateValue As Date = New Date(1601, 1, 1)
propSet.Add(zeroDateValue, "Zero Date")

Dim someDateValue As Date = New Date(2024, 12, 31)
propSet.Add(someDateValue, "Some Date")

Dim nowDateValue As Date = Now
propSet.Add(nowDateValue, "Now")
```

## Using Expressions in Inventor iProperties

Inventor’s iProperties dialog allows you to create dynamic values by using expressions. With an expression, you can combine multiple iProperty values into one. If you’ve used the method from a previous post to export parameters as iProperties, you can also include parameter values in these expressions.

### Creating an Expression

To create an expression, simply start the iProperty value with an equals sign (=). Inventor will automatically recognize it as an expression. Expressions can contain any text and reference existing iProperties by placing their names in square brackets, like this:

= &lt;Part Number> - &lt;Length> x &lt;Height>

In the example above, the expression uses:

- The built‑in **Part Number** iProperty
- Two custom iProperties, **Length and Height**, created by exporting parameters

Once you press Enter, Inventor evaluates the expression and displays the resulting value. An fx symbol appears next to the field, indicating the value is being driven by an expression. To edit the expression (not the evaluated result), right‑click the iProperty field and choose Edit Expression from the context menu.

### Working With Expressions Through the API

In the Inventor API, expressions are controlled through the Expression property of a Property object. This behaves the same as editing an expression directly in the iProperties dialog.

    Expression stores the raw expression string
    Value always returns the evaluated result
    Setting Value removes the expression and replaces it with a fixed value
    Setting Expression to an empty string ("") also removes the expression

Example: Adding an Expression:
```vb.net
' Get the active document.
Dim doc As Document = ThisApplication.ActiveDocument
' Get the PropertySets collection.
Dim propSets As PropertySets = doc.PropertySets
' Get the Inventor Summary Information property set.
Dim propSet As PropertySet = propSets.Item("Inventor Summary Information")
' Get the property
Dim prop As [Property] = propSet.Item("Comments")
' Set the expression
prop.Expression = "= <Part Number> - <Length> x <Height>"
```