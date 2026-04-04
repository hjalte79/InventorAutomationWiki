# A Simple April Fools iLogic Prank
Last week was April 1st, and I created a small iLogic prank to surprise a few colleagues during their daily workflow. The rule is completely harmless and automatically restores the model to its original state. You can attach this rule to any of Inventor’s iLogic triggers, such as On Parameter Change, Before Save, After Open, or After Document Change.
This prank also highlights something important about security: Inventor will show a warning when a rule uses features such as a random number generator. That warning exists to remind users to stay alert and not automatically trust every iLogic rule they come across.

# How the Prank Works
The rule has a 1‑in‑10 chance of running. It generates a random number, and only when the result is zero does the prank activate. This allows it to trigger rarely and unexpectedly, which makes it more surprising.
If the rule triggers, and the active document is an assembly, the script will:

- Disable user interaction
- Start a transaction
- Animate an “explosion” by pushing components outward
- Show a couple of message boxes for added drama
- Roll back everything so the model returns to normal

If the random number is not zero, the rule ends immediately and does nothing.

# Choosing the Right Trigger
Inventor provides several trigger options for iLogic rules. You can attach this rule to any of them depending on how often you want the prank to run. Examples include:

## On Parameter Change
Runs whenever a parameter value is edited. This makes the prank feel tied to a normal modeling action.
## After Open
Triggers when the document is opened. Good for a sudden surprise at the start of a workday.
## Before Save
Runs just before saving the model. Creates a moment of panic before the rollback restores everything.
## After Document Change
Executes whenever the document updates due to a change in the design.

Any of these will work, but be mindful that a trigger that fires often may give the prank away unless the randomizer remains enabled.

# A Useful Security Reminder
Because this rule uses a Random object, Inventor will display a security warning before running it. Autodesk uses these warnings to prevent users from blindly trusting scripts that the find on the internet (like this one ;-).
In the context of an April Fools prank, the warning adds just a little extra tension, perfect for the moment before the harmless explosion animation begins.

If you dont like the security warning then you can remove the randomizer part.

# The Rule
```vb.next
imports System.Windows.Forms
Sub Main()
	
	' =============== This part trigers security warning ===============
	' =========== Remove if you dont want a security warning ===========
	' Create a random number generator
    Dim rng As New Random()

    ' Generate a number from zero to 10
    Dim roll As Integer = rng.Next(0, 11)

    ' Only run the prank when roll equals zero
    If roll <> 0 Then
        Exit Sub
    End If
	' ==================================================================
	' ==================================================================
	
    ' Check if the active document is an assembly document
    If (ThisDoc.Document.DocumentType = DocumentTypeEnum.kAssemblyDocumentObject) Then
        Dim doc As AssemblyDocument = ThisDoc.Document
        Dim def = doc.ComponentDefinition

        ' Disable user interaction so the user cannot interrupt the process
        ThisApplication.UserInterfaceManager.UserInteractionDisabled = True

        ' Start a transaction so the changes can be reverted
        Dim transaction = ThisApplication.TransactionManager.StartTransaction(doc, "Explode")
		MessageBox.Show("Are you sure you want to explode your model?", "MisChief", MessageBoxButtons.YesNo, MessageBoxIcon.Question)
        Try
            ' Get the active camera and make it fit the full assembly in view
            Dim camera = ThisApplication.ActiveView.Camera
            camera.Fit()
            camera.Apply()

            ' Call the explode routine on the top level occurrences
            Explode(def.Occurrences.Cast(Of ComponentOccurrence)())

            ' Show a message to the user
			MessageBox.Show("Is this what you wanted?", "AprilFools", MessageBoxButtons.YesNo, MessageBoxIcon.Question)

        Catch ex As Exception
            ' Errors are ignored on purpose
        Finally

            ' Abort the transaction so the exploded view is rolled back
            transaction.Abort()

            ' Re enable user interface interaction
            ThisApplication.UserInterfaceManager.UserInteractionDisabled = False
        End Try
    End If
End Sub

Private Sub Explode(occs As IEnumerable(Of ComponentOccurrence))

    ' Loop through all occurrences that are passed into this routine
    For Each occ As ComponentOccurrence In occs
        Try
            ' Get current transformation matrix of the component
            Dim matrix = occ.Transformation

            ' Extract its translation values
            Dim translation = matrix.Translation

            ' Create a new matrix that only contains this translation vector
            Dim newMatrix = ThisApplication.TransientGeometry.CreateMatrix()
            newMatrix.SetTranslation(translation)

            ' Apply the translation to the original transformation
            matrix.TransformBy(newMatrix)

            ' Update the component position without considering constraints
            occ.SetTransformWithoutConstraints(matrix)

            ' If this component has sub occurrences, explode them recursively
            If (occ.SubOccurrences.Count > 0) Then
                Explode(occ.SubOccurrences.Cast(Of ComponentOccurrence)())
            End If

        Catch ex As Exception
            ' Errors inside individual occurrences are ignored
        End Try
    Next

End Sub
```

