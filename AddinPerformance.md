# Improving Your addins Performance
Improving the performance of your add-ins can make a huge difference in how smoothly users experience your tools. Back in 2015, Brian Ekins wrote a helpful article that explored this topic in detail. Although his original post is no longer online, a preserved copy is still available on [blog.autodesk.io](https://blog.autodesk.io/improving-your-programs-performance/).
Unfortunately, that archived version hasn't aged well. The content is outdated in places, and the code samples have collapsed into single unreadable lines. Because of this, the valuable parts of the article are difficult to access today.
With this post I want to make the infromation uptodate and more acessable again, so developers can once again benefit from the practical guidance it offered. 
In this post and [my previous post](http://hjalte.nl/tutorials/70-improvingperformance) you will also find other stratagies for improving preformance that where not in original post. 

# Timing Your Program
Before optimizing your program, it’s important to measure its current performance so you can identify potential improvements and verify whether your changes actually help. In .NET, the Stopwatch class from System.Diagnostics provides an easy and accurate way to measure execution time. Example:

```vb.net
Dim stopwatch As New System.Diagnostics.Stopwatch
stopwatch.Start()

' Do some things here.

MessageBox.Show("Elapsed time: " &
                stopwatch.ElapsedMilliseconds / 1000 &
                " seconds")
```

## Running In-Process vs. Out-of-Process

One factor that significantly impacts performance is whether your program runs inside Inventor's process or outside of it.

- **In-process**: Your code runs within Inventor's own process. Add-ins run in-process.
- **Out-of-process**: Your code runs in a separate process. Standalone executables and programs running from other applications are out-of-process.

When running out-of-process, Windows must package every API call for inter-process communication. This adds overhead to each call, which can add up quickly.

### Measuring the Difference

The test below makes a simple API call that requires no real processing on Inventor's side, it just returns a known value. By repeating this call 100,000 times, we can isolate the overhead of the call itself:

```vb.net
Dim stopwatch As New System.Diagnostics.Stopwatch()
stopwatch.Start()

For i As Integer = 1 To 100000
    Dim code As Integer = ThisApplication.Locale
Next

stopwatch.Stop()
MsgBox("Total time: " & (stopwatch.ElapsedMilliseconds / 1000).ToString("0.00000"))
```


| Mode | Time | Calls per second |
|---|---|---|
| Out-of-process | 16.297 s | ~6136 |
| In-process | 2.341s | ~42716 |

That's roughly **7 times faster** in-process.

For the 'In-process' test I used iLogic.
For the 'Out-of-process' test I used my own Visual Studio project [ILogicRuleTester](https://github.com/hjalte79/ILogicRuleTester) which emulates iLogic.

### Setting Expectations

Don't expect your real-world programs to see a 7x improvement simply by switching to in-process. This test was specifically designed to measure call overhead using a trivial API property. In practice:

- Most API calls require Inventor to do actual work, and that processing time is the same regardless of in-process or out-of-process.
- The overhead is only the cost of packaging the call and returning the result.
- Even in the slow out-of-process test, the program still made over 5,400 calls per second.

That said, in most scenarios there **will** be a noticeable performance difference, so running in-process is preferable whenever possible.

#### Disable User Interaction

Besides the packaging overhead, another reason out-of-process calls are slower is that Inventor gives priority to its user interface. API calls from external processes are handled in between UI updates.

For in-process programs, Inventor dedicates its full processing power to your API calls. You can get closer to this behavior out-of-process by setting the `UserInteractionDisabled` property on the `UserInterfaceManager` object. Setting it to `True` locks the user interface, which:

1. Prevents the user from interfering while your program runs.
2. Allows Inventor to focus on handling API calls instead of watching the UI.

```vb.net
Dim stopwatch As New System.Diagnostics.Stopwatch()
stopwatch.Start()

ThisApplication.UserInterfaceManager.UserInteractionDisabled = True

For i As Integer = 1 To 100000
    Dim code As Integer = ThisApplication.Locale
Next

ThisApplication.UserInterfaceManager.UserInteractionDisabled = False

stopwatch.Stop()
MsgBox("Total time: " & (stopwatch.ElapsedMilliseconds / 1000).ToString("0.00000"))
```

With this single change, the out-of-process time drops from **16.297 seconds to 13.582 seconds**. In more complex programs, the improvement can be even greater.

> ⚠️ Always add error handling when using `UserInteractionDisabled`. If your program crashes without resetting this property to `False`, Inventor's UI will remain frozen and you won't be able to interact with it.

> ⚠️ Because Inventor reacts differently to this setting, it can be used to check if you are running in-process or out-of-process.

## Use References Instead of Inline Calls

Another optimization is to store objects you use multiple times in variables rather than retrieving them repeatedly.

The example below iterates through all occurrences in an assembly and retrieves the name and filename of each one. An iLogic rule in an assembly with 800 occurrences takes **7.3 seconds**. (I used the Jet_Engine_Model from the [Inventor example files](https://www.autodesk.com/support/technical/article/caas/tsarticles/ts/3gnm93P9sPAWE6vndk7fjq.html))

```vb.net
Dim stopwatch As New System.Diagnostics.Stopwatch()
stopwatch.Start()

For index = 1 To 10
    Dim asmDoc As AssemblyDocument = ThisApplication.ActiveDocument

    For i As Integer = 1 To asmDoc.ComponentDefinition.Occurrences.AllLeafOccurrences.Count
        Dim name As String = asmDoc.ComponentDefinition.Occurrences.AllLeafOccurrences.Item(i).Name

        Dim filename As String = asmDoc.ComponentDefinition.Occurrences.AllLeafOccurrences.Item(i).
                    ReferencedDocumentDescriptor.
                    ReferencedFileDescriptor.FullFileName
    Next
Next


stopwatch.Stop()
MsgBox("Total time: " & (stopwatch.ElapsedMilliseconds / 1000).ToString("0.00000"))
```

Here's an improved version. An iLogic rule takes only **1.6 seconds**.

```vb.net
Dim stopwatch As New System.Diagnostics.Stopwatch()
stopwatch.Start()

For index = 1 To 10
    Dim asmDoc As AssemblyDocument = ThisApplication.ActiveDocument
    Dim occs = asmDoc.ComponentDefinition.Occurrences.AllLeafOccurrences

    For i As Integer = 1 To occs.Count
        Dim name As String = occs.Item(i).Name

        Dim filename As String = occs.Item(i).
                    ReferencedDocumentDescriptor.
                    ReferencedFileDescriptor.FullFileName
    Next
Next

stopwatch.Stop()
MsgBox("Total time: " & (stopwatch.ElapsedMilliseconds / 1000).ToString("0.00000"))
```

The difference is that the second version retrieves the `AllLeafOccurrences` object once, then reuses them. This significantly reduces the number of API calls. 

> 💡Also notice that the readabilty difference

A single line like:

```vb.net

asmDoc.ComponentDefinition.Occurrences.AllLeafOccurrences.Item(i).Name
```

results in several API calls, and each call takes time. By making those calls once and reusing the result, you can noticeably improve performance.

## Use For Each Instead of Count and Item

Another optimization is to use `For Each` when iterating over collections. Here's a version of the same test using `For Each`. Besides being more efficient, it's also easier to read.

An iLogic rule takes **1.1 seconds**, a small improvement over the previous example. The actual time saved will vary depending on which collection you're iterating through.

```vb.net
Dim stopwatch As New System.Diagnostics.Stopwatch()
stopwatch.Start()

For index = 1 To 10
    Dim asmDoc As AssemblyDocument = ThisApplication.ActiveDocument
    Dim occs = asmDoc.ComponentDefinition.Occurrences.AllLeafOccurrences

    For Each occ As ComponentOccurrence In occs
        Dim name As String = occ.Name

        Dim filename As String = occ.ReferencedDocumentDescriptor.
                    ReferencedFileDescriptor.FullFileName
    Next
Next

stopwatch.Stop()
MsgBox("Total time: " & (stopwatch.ElapsedMilliseconds / 1000).ToString("0.00000"))
```

`For Each` is faster because Inventor knows you'll be iterating over the entire collection, allowing it to optimize internally. When you use the `Item` property, Inventor doesn't know if you want a single item or plan to iterate through multiple items.

Some collections are already optimized for single-item access, so you won't see big improvements there. Other collections will be significantly faster with `For Each`.

## Get Items By Name

Another optimization is to use the built-in capability of many collections to retrieve an object by name, rather than iterating through the entire collection to find it.

The `Item` property of many collections accepts either an index or the name of the object you want. Some collections also have specialized properties like `ItemById`. Using these is much more efficient than looping through every item looking for a match.

Here's an iLogic rule that searches 100 times for the DXF translator add-in in 726 miliseconds

```vb.net
Dim stopwatch As New System.Diagnostics.Stopwatch()
stopwatch.Start()

Dim addins = ThisApplication.ApplicationAddIns
For index = 1 To 1000
    For Each addin As ApplicationAddIn In addins
        If (addin.ClientId = "{C24E3AC4-122E-11D5-8E91-0010B541CD80}") Then
            Exit For
        End If
    Next
Next

stopwatch.Stop()
MsgBox("Total time: " & stopwatch.ElapsedMilliseconds)
```

Here's the equivalent code using the `ItemById` property of the `ApplicationAddIns` object. It runs in 33 miliseconds

```vb.net
Dim stopwatch As New System.Diagnostics.Stopwatch()
stopwatch.Start()

Dim addins = ThisApplication.ApplicationAddIns
For index = 1 To 1000
    Dim dxfAddIn = addins.ItemById("{C24E3AC4-122E-11D5-8E91-0010B541CD80}")
Next

stopwatch.Stop()
MsgBox("Total time: " & stopwatch.ElapsedMilliseconds)
```

The second version is not only simpler but also much faster. Inventor performs the lookup internally in a highly optimized way, rather than checking objects one by one.

## Avoid Constraints and Defer Updates When Possible

In automated workflows where assemblies are generated programmatically and not meant for manual editing, you can place occurrences without constraints and simply ground them. This avoids the overhead of Inventor’s constraint solver. As the results table at the end of this section shows, skipping constraints dramatically improves performance—often by several seconds, even for small assemblies.
Since your code controls the exact placement, omitting constraints in this context doesn’t reduce stability. What it does reduce is the repeated constraint solving that normally happens after each constraint is added.

### Defer Assembly Updates

If your workflow still requires constraints, you can improve performance by deferring assembly updates. This pauses constraint solving so Inventor doesn’t recompute the entire assembly after each added constraint.
You can toggle this setting in the UI or via the API:

```vb.net
ThisApplication.AssemblyOptions.DeferUpdate = True
```
With updates deferred, constraints are created much faster because the solver does not run constantly.
Performance improves even further if you temporarily disable screen updates:
```vb.net
ThisApplication.ScreenUpdating = False
```
This prevents Inventor from redrawing the viewport during your operation.
> ⚠️ Important: Always use error handling when setting ScreenUpdating = False. If your program exits unexpectedly without restoring it, Inventor’s UI will remain frozen.

|Approach|Time100| 
|---|---|
| occurrences without constraints | 3.39 seconds |
| 100 occurrences with constraints | 17.99 seconds |
| 100 occurrences with constraints and defer updates | 4.64 seconds |
| 100 occurrences with constraints, defer updates, and screen updating disabled | 2.24 second |

### Test Code Used
Below is the exact iLogic rule used to measure performance. (I used the Jet_Engine_Model from the [Inventor example files](https://www.autodesk.com/support/technical/article/caas/tsarticles/ts/3gnm93P9sPAWE6vndk7fjq.html))

```vb.net
Sub Main()

    Dim doc As AssemblyDocument = ThisDoc.Document
    Dim def As AssemblyComponentDefinition = doc.ComponentDefinition
    Dim occs As ComponentOccurrences = def.Occurrences

    Dim firstPart = def.Occurrences.AllLeafOccurrences.Item(1)
    Dim fileName = firstPart.ReferencedFileDescriptor.FullFileName

    Dim orign = ThisApplication.TransientGeometry.CreateMatrix()

    Dim stopwatch As New System.Diagnostics.Stopwatch()
    stopwatch.Start()

    ' Set to 'True' to run with defer updates.
    ThisApplication.AssemblyOptions.DeferUpdate = False
	
	' Set to False to disable screen updating
	ThisApplication.ScreenUpdating = True
    Dim previousOcc As ComponentOccurrence = occs.Add(fileName, orign)
    Dim previousAxis = GetFirstProxyAxis(previousOcc)

    For index = 1 To 100
        Dim newOcc = occs.Add(fileName, orign)
        Dim newAxis = GetFirstProxyAxis(newOcc)

        ' Çomment the next line to run with out constraining
        def.Constraints.AddMateConstraint(newAxis, previousAxis, 0)

        previousAxis = newAxis
    Next

	ThisApplication.ScreenUpdating = True
    stopwatch.Stop()
    MsgBox("Total time: " & (stopwatch.ElapsedMilliseconds / 1000).ToString("0.00"))

End Sub

Public Function GetFirstProxyAxis(occ As ComponentOccurrence)
    Dim doc As PartDocument = occ.Definition.Document
    Dim axis As WorkAxis = doc.ComponentDefinition.WorkAxes.Item(1)
    Dim proxy As WorkAxisProxy
    occ.CreateGeometryProxy(axis, proxy)
    Return proxy
End Function
```

## Use Defer When Creating Sketches

The same principle applies to sketches. Like assemblies, sketches solve constraints as you add entities. However, unlike assemblies, you can't skip constraints entirely. Constraints connect sketch entities together, so they're required. You can minimize the number of constraints, but you can never eliminate them completely.

Sketches support deferring updates through the `Sketch.DeferUpdates` property:

```vb.net
Dim sketch As Sketch = ...

sketch.DeferUpdates = True
```

Setting this to **True** before adding sketch geometry prevents the sketch from solving after each entity is added. For sketches with many entities, this results in a significant speedup.

## Transactions

In Inventor every operations like add, create and edit are wrapped in a transaction. In the user interface each transaction is shown as an Undo steps. The API lets you explicitly manage that behavior with:
```vb.net
Dim trans As Transaction =  ThisApplication.TransactionManager.StartTransaction(myDoc, "My Command")

' Your actions here

trans.End()
```
This function wraps multiple transactions in one transaction so they become one Undo step. This keeps your add‑in’s actions user-friendly and reduces per‑action bookkeeping. 

That is convenient for the user but it doesn’t improve performance much. There is another type of transaction that will block the individual transactions from being created. This capability is accessed by using the hidden StartGlobalTransaction method. This is used in exactly the same as the standard StartTransaction. The downside of using this, and the reason it’s hidden, is that you need to be very careful to make sure you always end a global transaction or you can leave Inventor in a bad state that will most likely result in a crash as you or the user continues to do work. You’ll want to wrap the code that’s doing the work in an error handler so that if an error occurs you can either end, or more typically, abort the transaction.

This are my stats after running the benchmark iLogic rule below.
|Approach|Time| 
|---|---|
| Without managing transactions | 16.86 seconds|
| With StartTransaction() | 14.19 seconds |
| With StartGlobalTransaction | 11.42 seconds |
| With StartGlobalTransaction, DeferUpdate = true, ScreenUpdating = false    | 1.5 seconds |

```vb.net
Sub Main()

    Dim doc As AssemblyDocument = ThisDoc.Document
    Dim def As AssemblyComponentDefinition = doc.ComponentDefinition
    Dim occs As ComponentOccurrences = def.Occurrences

    Dim firstPart = def.Occurrences.AllLeafOccurrences.Item(1)
    Dim fileName = firstPart.ReferencedFileDescriptor.FullFileName

    Dim orign = ThisApplication.TransientGeometry.CreateMatrix()

    Dim stopwatch As New System.Diagnostics.Stopwatch()
    stopwatch.Start()

    Dim previousOcc As ComponentOccurrence = occs.Add(fileName, orign)
    Dim previousAxis = GetFirstProxyAxis(previousOcc)

	' On the next line try StartGlobalTransaction or remove the transaction 
	Dim trans As Transaction = ThisApplication.TransactionManager.StartTransaction(doc, "My Command")
		
	    For index = 1 To 100
	        Dim newOcc = occs.Add(fileName, orign)
	        Dim newAxis = GetFirstProxyAxis(newOcc)
	
	        ' Çomment the next line to run with out constraining
	        def.Constraints.AddMateConstraint(newAxis, previousAxis, 0)
	
	        previousAxis = newAxis
	    Next
	trans.End()

    stopwatch.Stop()
    MsgBox("Total time: " & (stopwatch.ElapsedMilliseconds / 1000).ToString("0.00"))

End Sub

Public Function GetFirstProxyAxis(occ As ComponentOccurrence)
    Dim doc As PartDocument = occ.Definition.Document
    Dim axis As WorkAxis = doc.ComponentDefinition.WorkAxes.Item(1)
    Dim proxy As WorkAxisProxy
    occ.CreateGeometryProxy(axis, proxy)
    Return proxy
End Function
```

## Execution Is Faster When the Mouse Is Moving
One of the strangest performance quirks you may encounter while developing Inventor add-ins is that your code appears to run faster when you move the mouse over the Inventor window. As odd as it sounds, this behavior is real and stems from how Inventor handles external API calls. 
Inventor processes both UI events and COM API calls on a single main thread, meaning that when the UI is idle, Inventor polls for incoming API requests less frequently. Moving the mouse artificially increases UI activity, which forces Inventor to process its message queue more often—resulting in your automation code executing noticeably faster. According to [some sources](https://forums.autodesk.com/t5/inventor-programming-forum/why-execution-is-faster-when-mouse-moving-on-screen/td-p/8792842), the solution is to temporarily disable UI interaction, which allows Inventor to give full attention to API calls and removes the dependency on accidental mouse movement. This can be accomplished by setting these options:

```vb.net
ThisApplication.UserInterfaceManager.UserInteractionDisabled = True

ThisApplication.DrawingOptions.EnableBackgroundUpdates = False
```