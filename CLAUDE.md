# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This is a community-driven documentation wiki for Autodesk Inventor automation. It contains tutorials, code examples, and reference articles covering:

- **iLogic**: Inventor's built-in rule-based automation
- **Inventor API**: Direct programmatic access via VB.NET/C#
- **Add-ins**: Custom plugins that extend Inventor's functionality
- **External DLLs**: Class libraries called from iLogic rules
- **Power Automate**: Integration with Vault REST API

## Content Structure

- **Root folder**: Main tutorials and reference articles (Markdown files)
- **Blog/**: Additional blog-style posts
- **images/**: Screenshots and diagrams referenced by root articles
- **Blog/images/**: Screenshots and diagrams referenced by blog posts

### Root Articles (Tutorials & Reference)

- **ReadMe.md**: Welcome page and wiki overview
- **InstallVS.md**: Installing Visual Studio for Inventor automation
- **CreatNewProject.md**: Creating a new Visual Studio project
- **AddProjectRefrence.md**: What is a reference and how to add one
- **CreateAddin2025AndLater.md**: Creating an add-in from scratch (Inventor 2025+)
- **SetupDebugApplication.md**: Setting Inventor as startup program for debugging
- **DebugMultiVersions.md**: Debugging multiple Inventor versions in one solution
- **DllForIlogic.md**: Extending iLogic functions with a DLL
- **EventHandler.md**: Event handling in Inventor add-ins
- **HelpMyAddinWontLoad.md**: Troubleshooting add-in loading issues
- **AddinPerformance.md**: Improving add-in performance

### Blog Posts

- **AccessingIProperties.md**: Basics of reading iProperties
- **AddDimension.md**: Adding dimensions to drawings with iLogic
- **AllAppearances.md**: Showing all appearance/texture properties
- **AssignName.md**: Assigning names to faces by code
- **ButtonConstructor.md**: Creating buttons for external iLogic rules (add-on)
- **ChangedDimensions.md**: Highlighting changed dimensions when opening drawings
- **CleanupDrawingRecources.md**: Cleaning up drawing resources
- **ColourParts.md**: Colouring parts in assembly by part number
- **ComparePartGeometry.md**: Comparing part geometry with iLogic
- **DebuggingILogic.md**: Writing and debugging iLogic rules
- **DesignDoctor2.md**: Design doctor second opinion
- **ExportFlatPatterns.md**: Exporting flat patterns (iPart)
- **ExtraLayersDXF.md**: Setting extra layers in DXF exports
- **FilesCheckedOut.md**: Counting checked-out files in Vault with Power Automate
- **FindSheet.md**: Finding sheet objects in a list
- **FlatPatternOrientation.md**: Showing flat pattern orientation
- **GenerateBendNotes.md**: Automatically generating bend notes
- **IPropertiesSpecialCases.md**: iProperties special cases (dates, expressions)
- **LoftFeature.md**: Creating a loft feature with iLogic
- **ManipulatingCustomIProperties.md**: Creating, editing, and managing custom iProperties
- **MiniToolbar.md**: Creating a mini toolbar for user input
- **MiniumRangebox.md**: Minimum range box in any orientation
- **ModelStates.md**: Model states and parameters API deep dive
- **MultiSelectListBox.md**: Multi-select list box based on InputListBox
- **ObjectOrientedDxfExport.md**: Object-oriented DXF flat pattern export
- **OccurrenceSearch.md**: Universal occurrence search function
- **OneRuleToSearchThemAll.md**: One rule to search them all
- **PointMark.md**: Point, click and mark a part with a part number
- **PowerAutomate.md**: Connecting Power Automate to the Vault REST API
- **RetrieveModelAnnotation.md**: Retrieving model annotations for drawing dimensions
- **RuleOnlyOnce.md**: Running "Before Save" rule only once in assemblies
- **SelectingDimensions.md**: Selecting dimensions (dimension line vs extension lines)
- **StopLongRunningRule.md**: Stopping a long-running iLogic rule with progress bar
- **Transactions.md**: Transactions for robust and fast rules
- **UnexpectedResults.md**: Debugging unexpected results in iProperties
- **Units.md**: Parameter, document, and database units
- **VbaIsObsolete.md**: Why VBA in Inventor is obsolete
- **VirtualComponentLIbrary.md**: Virtual component library
- **generateBendDimensions.md**: Automatically generating bend dimensions
- **generateHolePosition.md**: Automatically generating hole position dimensions
- **generateOverallDimensions.md**: Automatically generating overall dimensions

## Writing Guidelines

Content is authored collaboratively with AI assistance. The author provides technical information and code examples; AI helps with structure, formatting, and clarity. All code examples are human-authored.

When editing or adding content:
- Use consistent Markdown formatting
- Never use — in sentences!
- Include relevant screenshots in the `images/` folder
- Code examples should be in VB.NET (aligns with iLogic syntax) unless C# is specifically needed
- Use callout boxes: `> 💡` for tips, `> ⚠️` for warnings, `> ❓` for explanations
