
# Event Handling in Autodesk Inventor Add-Ins

Event handling in Autodesk Inventor add-ins allows you to respond to changes in the application, such as document events, user actions, or updates to geometry and parameters. The Inventor API provides a wide range of events that can be used to monitor and extend behavior at both the application and document level.

## Event handler class

When developing add-ins for Autodesk Inventor, For this example we will override the default "Save As" dialog. For example to provide a more tailored experience.

The process begins during initialization (in the sub new(...)). When an instance of InventorEventHandler is created, it receives the main Inventor.Application object. From this, it retrieves the FileUIEvents interface and stores it in a private class-level variable. This step is important: if the event object isn't held in memory, the .NET garbage collector may dispose of it, and the subscribed event will silently stop firing. With the reference safely stored, the add-in subscribes to the OnFileSaveAsDialog event, preparing to intercept save operations.

Once the user activates the "Save As" command within Inventor, the subscribed event handler method OnFileSaveAsDialog is triggered. The method receives several parameters: a list of allowed file types, context information (such as the top-level document and suggested directory), and an editable FileName. Before doing anything else, the method explicitly sets HandlingCode to kEventNotHandled, which tells Inventor to proceed with its normal behavior unless this value is changed later.

The handler then creates a standard Windows Save File Dialog (SaveFileDialog), populating it with details extracted from the current document and Inventor context. If the user confirms the dialog by selecting a file and clicking Save, the method assigns the chosen path to the FileName parameter and sets HandlingCode to kEventHandled. This signals to Inventor that the event has been fully handled and that it should not display its own dialog or continue the default save operation.

If the user cancels the dialog or closes it, the handling code remains unchanged. In that case, Inventor continues as if the add-in never intervened, ensuring the built-in behavior is preserved. This workflow override specific parts of Inventor’s user interface while respecting its internal event model.

Have a look at the following class

```vb
Imports Inventor
Imports Microsoft.Win32

Public Class InventorEventHandler

    Private _inventor As Inventor.Application
    Private _fileUiEvents As FileUIEvents

    Public Sub New(inventor As Inventor.Application)
        _inventor = inventor

        ' Get an instance of the desired event-object and store it in a class-level variable.
        ' This is required — if the event object is not held in memory, it may be garbage
        ' collected, and the event handler will not be triggered.
        _fileUiEvents = _inventor.FileUIEvents

        ' subscribe to the desired event 
        AddHandler _fileUiEvents.OnFileSaveAsDialog, AddressOf OnFileSaveAsDialog
    End Sub

    Private Sub OnFileSaveAsDialog(ByRef FileTypes() As String, SaveCopyAs As Boolean, ParentHWND As Integer, ByRef FileName As String, Context As NameValueMap, ByRef HandlingCode As HandlingCodeEnum)


        ' Always set the HandlingCode to indicate how the event was processed.
        ' By default, set it to kEventNotHandled.
        ' If you fully handle the event and want to suppress Inventor’s default behavior,
        ' update it to kEventHandled instead.
        HandlingCode = HandlingCodeEnum.kEventNotHandled

        ' This method should only run *before* the document is saved.
        ' If that's not the case, exit early.
        ' If (BeforeOrAfter <> EventTimingEnum.kBefore) Then Return

        Dim topLevelDocument As Document = Context.Value("TopLevelDocument")
        Dim fileInfo As New System.IO.FileInfo(topLevelDocument.FullFileName)

        Dim saveDialog As New SaveFileDialog()
        saveDialog.Title = "My replacement 'Save Document' dialog!"
        saveDialog.InitialDirectory = Context.Value("InitialDirectory")
        saveDialog.Filter = String.Join("|", FileTypes)
        saveDialog.FileName = fileInfo.Name


        If (saveDialog.ShowDialog() = True) Then
            FileName = saveDialog.FileName

            
        End If


        ' We've handled the event, so we want to prevent Inventor from continuing with its default behavior.
        ' Setting this flag tells Inventor not to show the standard dialog or perform the default action.
        HandlingCode = HandlingCodeEnum.kEventHandled
    End Sub
End Class
```

The OnFileSaveAsDialog method — like most event-handling methods in the Inventor API — includes several parameters that provide context and allow for customization. One common parameter across many events is BeforeOrAfter, which indicates whether the event is being triggered before or after Inventor performs its default action. This is especially useful when you only want to respond at a specific point in the workflow. For example, if your logic should run only before the file is saved, you can check the timing like this:
```vb
If (BeforeOrAfter <> EventTimingEnum.kBefore) Then Return
```
This allows you to exit the handler early and avoid running code when it's no longer relevant or even worse running the code multiple times.

## Initializing InventorEventHandler

The InventorEventHandler class won’t work automatically — it needs to be explicitly initialized. Typically, this is done within the StandardAddInServer class, inside the Activate(...) method. If you've followed my other tutorials, your Activate method probably looks something like this:

```vb
' You also need to add this as it is set in the activate sub
Private _eventHandler As InventorEventHandler 

Public Sub Activate(AddInSiteObject As ApplicationAddInSite, FirstTime As Boolean) Implements ApplicationAddInServer.Activate
    Try

        ' initialize the rule class
        _myButton = New MyButton(AddInSiteObject.Application)

        ' Here, we initialize the InventorEventHandler and begin listening for events.
        _eventHandler = New InventorEventHandler(AddInSiteObject.Application)

    Catch ex As Exception

        ' Show a message if any thing goes wrong.
        MessageBox.Show(ex.Message)

    End Try
End Sub
```