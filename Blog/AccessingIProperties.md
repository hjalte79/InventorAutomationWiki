
# Accessing IProperties

Working with iProperties is a common task when automating Autodesk Inventor: managing metadata, preparing BOM exports, or enforcing naming rules. Developers typically access these properties through one of two interfaces: the **Inventor API** or the **iLogic API**. Although both expose the same underlying document data, they do so in noticeably different ways. The object models differ, the available property names are not always identical, and some operations are simpler in one API than the other.

This post outlines the practical differences between the two APIs when working with iProperties and serves as a reference for developers who want a clear overview of **all iProperties accessible through both APIs**.

The goal is to provide a straightforward technical comparison, plus a complete list of property names you can rely on when writing rules, add‑ins, scripts, or external automation.

## Working with iProperties: Inventor API vs iLogic

iProperties are one of the most frequently accessed parts of Inventor automation, and because of that they tend to generate many questions. This section gives a short overview of how to work with iProperties using both the Inventor API and iLogic, shown side‑by‑side so you can see the differences clearly.

Every Inventor document exposes its iProperties through a PropertySets collection. In the Inventor API, you must navigate through:

- Document
- PropertySets
- Individual PropertySet
- Individual Property

iLogic provides a much simpler interface for the same underlying data. Below are examples for reading and writing the Part Number property.

# Getting a Reference to the Part Number Property

**Inventor API (VB.NET)**
```vb.net
' Get the active document.
Dim doc As Document = ThisApplication.ActiveDocument

' Get the PropertySets collection.
Dim propSets As PropertySets = doc.PropertySets

' Get the Design Tracking Properties property set.
Dim propSet As PropertySet = propSets.Item("Design Tracking Properties")

' Get the Part Number property.
Dim partNumberProp As [Property] = propSet.Item("Part Number")
```

**iLogic**
```vb.net
Dim partNumberValue = iProperties.Value("Project", "Part Number")
```

iLogic uses category names from the iProperties dialog (“Summary”, “Project”, “Status”, “Custom”), not the internal Inventor API property‑set names (Inventor Summary Information, Inventor Document Summary Information, Design Tracking Properties, Inventor User Defined Properties ).

## Reading the Part Number

**Inventor API (VB.NET)**
```vb.net
MsgBox("Part Number: " & partNumberProp.Value)
```
**iLogic**
```vb.net
MessageBox.Show("Part Number: " & partNumberValue) 
```

## Setting the Part Number

```vb.net
partNumberProp.Value = "SamplePart001"
```
**iLogic**
```vb.net
iProperties.Value("Project", "Part Number") = "SamplePart001" 
```

## Refrence table

The table below provides a unified reference for all standard Inventor iProperties, showing how each property is exposed in both the ***Inventor API** and the **iProperties dialog** inside Inventor. Although Inventor presents iProperties to users through four tabs (Summary, Project, Status, and Custom) the underlying API organizes these properties into **PropertySets** with different internal names. This often leads to confusion because the names visible in the UI do not match the names required in code.  To clarify this relationship, each row in the table includes:

### Name

The display name of the iProperty as shown in the Inventor UI and commonly referenced in documentation, title blocks, and user workflows.

### PropertySet name (iLogic)

This is the tab where users can locate the property inside Inventor’s iProperties dialog (Summary, Project, Status, or Custom).
iLogic uses these tab names as the first argument in calls such as:
```vb.net
Properties.Value("Project", "Part Number") 
```

### PropertySet name (Inventor Api)

This is the internal Inventor API name of the PropertySet object that contains the property. When using the Inventor API (VB.NET, C#, add‑ins), you must reference the exact PropertySet name, for example:

```vb.net
Dim ps = doc.PropertySets.Item("Design Tracking Properties") 
```

### Type

The data type of the property.
This is especially important because Inventor expects the correct COM type when assigning values. For example:

- String values must be plain text,
- Date values must be valid filetime‑compatible dates,
- Currency properties require numeric or currency‑formatted input,
- IPictureDisp values represent internal picture objects (thumbnails, part icons).

### Custom properties

Custom iProperties do not exist until created by the user or by automation.
In the table they are represented as:

- Name: (custom property name)
- Tab: Custom
- PropertySet: Inventor User Defined Properties
- Type: depends on creation (String, Number, Yes/No, Date)

### How developers can use this table

- **iLogic authors** can quickly determine which tab and name to reference using iProperties.Value(tab, propertyName).
- **Inventor API developers** can map UI names to actual PropertySet names and avoid guessing or inspecting properties at runtime.
- **Add‑in developers** can use the Type column to properly cast values before assignment and avoid COM type issues.


| Name | PropertySet name (iLogic) | PropertySet name (Inventor Api) | Type |
|------|----------------------------|----------------------------------|------|
| Author | Summary | Inventor Summary Information | String |
| Authority | Project | Design Tracking Properties | String |
| Catalog Web Link | Project | Design Tracking Properties | String |
| Categories | Project | Design Tracking Properties | String |
| Category | Summary | Inventor Document Summary Information | String |
| Checked By | Project | Design Tracking Properties | String |
| Comments | Summary | Inventor Summary Information | String |
| Company | Summary | Inventor Document Summary Information | String |
| Cost | Project | Design Tracking Properties | Currency |
| Cost Center | Project | Design Tracking Properties | String |
| Creation Time | Project | Design Tracking Properties | Date |
| Date Checked | Project | Design Tracking Properties | Date |
| Defer Updates | Project | Design Tracking Properties | Boolean |
| Description | Project | Design Tracking Properties | String |
| Design Status | Project | Design Tracking Properties | Long |
| Designer | Project | Design Tracking Properties | String |
| Document SubType | Project | Design Tracking Properties | String |
| Document SubType Name | Project | Design Tracking Properties | String |
| Engineer | Project | Design Tracking Properties | String |
| Engr Approved By | Project | Design Tracking Properties | String |
| Engr Date Approved | Project | Design Tracking Properties | Date |
| External Property Revision Id | Project | Design Tracking Properties | String |
| Keywords | Summary | Inventor Summary Information | String |
| Language | Project | Design Tracking Properties | String |
| Last Saved By | Summary | Inventor Summary Information | String |
| Manager | Summary | Inventor Document Summary Information | String |
| Manufacturer | Project | Design Tracking Properties | String |
| Material | Project | Design Tracking Properties | String |
| Mfg Approved By | Project | Design Tracking Properties | String |
| Mfg Date Approved | Project | Design Tracking Properties | Date |
| Parameterized Template | Project | Design Tracking Properties | Boolean |
| Part Icon | Project | Design Tracking Properties | IPictureDisp |
| Part Number | Project | Design Tracking Properties | String |
| Part Property Revision Id | Project | Design Tracking Properties | String |
| Project | Project | Design Tracking Properties | String |
| Proxy Refresh Date | Project | Design Tracking Properties | Date |
| Revision Number | Summary | Inventor Summary Information | String |
| Size Designation | Project | Design Tracking Properties | String |
| Standard | Project | Design Tracking Properties | String |
| Standard Revision | Project | Design Tracking Properties | String |
| Standards Organization | Project | Design Tracking Properties | String |
| Stock Number | Project | Design Tracking Properties | String |
| Subject | Summary | Inventor Summary Information | String |
| Template Row | Project | Design Tracking Properties | String |
| Thumbnail | Summary | Inventor Summary Information | IPictureDisp |
| Title | Summary | Inventor Summary Information | String |
| User Status | Project | Design Tracking Properties | String |
| Vendor | Project | Design Tracking Properties | String |
| Weld Material | Project | Design Tracking Properties | String |
| [custom property] | Custom | Inventor User Defined Properties | [Custom] |

## Summary
Accessing an iProperty is simply a matter of navigating the Inventor object model (Inventor API) or using the simplified iProperties.Value(category, name) call (iLogic). The main challenge in both cases is knowing:

- Which property set the property belongs to, and
- The exact name of the property, which must match Inventor’s internal name.
