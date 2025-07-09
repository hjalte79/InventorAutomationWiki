# Debugging Multiple Inventor Versions in One Solution

Developing tools and add-ins for Autodesk Inventor often means supporting multiple versions of the software. This can be challenging when trying to debug, build, and maintain your code—especially when different Inventor versions target different .NET frameworks or have slight differences in their interop assemblies.

This tutorial will guide you through setting up a **single Visual Studio solution** that can:

- Target multiple **.NET frameworks**, specifically:
  - **.NET Framework 4.8** — required for **Inventor 2024 and earlier**
  - **.NET 8.0** — supported by **Inventor 2025 and later**
- Load **Inventor interop assemblies** specific to each Inventor version.
- Use **conditional compilation** to separate version-specific logic.
- Allow you to **debug against multiple versions of Inventor** without duplicating your entire codebase.

By the end of this guide, you’ll be able to create a flexible and maintainable development setup that simplifies support for multiple Inventor releases. This is especially useful for teams maintaining long-term products or automation tools where end users may be working on different Inventor versions.

## Create Debug Profiles

We start by creating debug profiles for each Inventor version you want to debug. This allows you to quickly launch the correct version of Inventor directly from Visual Studio while testing your add-in.

If you're not familiar with setting up debug profiles, see the article: [Set Inventor as a Startup Program for Debugging](./SetupDebugApplication.md). It explains how to configure a basic **Debug profile** in Visual Studio.

For this tutorial, we assume you want to debug both Inventor 2024 and Inventor 2025. This represents the most complex (but also most useful) setup. You will need to create two separate debug profiles: **Inventor 2024** and **Inventor 2025**.

These names are important. We will refer to them later in the tutorial to configure conditional logic and reference loading based on the selected profile.





## Changing the Project File (Manually)

In most cases, you wouldn’t edit the project file directly. Visual Studio provides UI tools for managing references, build settings, and other project properties. However, in this case, we will edit the `.csproj` file manually because it gives us far more flexibility.

> ⚠️ This only works if your project has already been converted to a **.NET Core or SDK-style format**. If your project is still in the older .NET Framework format (non-SDK style), you will first need to convert it. This process is explained in detail on the page: [Migration addins to .NET Core 8 (Inventor 2025 and later)](link-to-your-conversion-page).



### Targeting Multiple Frameworks

To support both Inventor 2024 and Inventor 2025, you can configure your project to target multiple .NET frameworks. This allows you to build the same codebase for both .NET Framework 4.8 (Inventor 2024 and earlier) and .NET Core 8.0 (Inventor 2025 and later).

In your project file (`.csproj`), find the line that looks like this:
```xml
<TargetFramework>net8.0-windows</TargetFramework>
```
You can then change this line to:
```xml
<TargetFrameworks>net8.0-windows;net48</TargetFrameworks>
```
> ⚠️ The XML element is now plural: `<TargetFrameworks>`. Be sure to add an "s" to both the opening and closing tags.

This multi-targeting setup tells MSBuild to compile the project for both frameworks. Later, you can use conditional compilation and framework-specific references to handle differences between Inventor versions.

### Referencing Libraries (DLL) for Different Debug Profiles

In the project file, you’ll find one or more XML nodes called `ItemGroup`. These nodes are used to define references and other build-related items. One of them will contain references to Inventor-specific libraries (DLLs).

To support debugging multiple Inventor versions in the same project, we’ll move those references into two new `ItemGroup` nodes — each with a `Condition` attribute. These conditions will check the active debug profile name and only include the references that match.

The modified structure looks like this:
```xml
<ItemGroup Condition="'$(ActiveDebugProfile)' == 'Inventor 2024'">
    <Reference Include="Autodesk.Inventor.Interop">
        <HintPath>C:\Program Files\Autodesk\Inventor 2024\Bin\Public Assemblies\Autodesk.Inventor.Interop.dll</HintPath>
        <Private>True</Private>
        <EmbedInteropTypes>False</EmbedInteropTypes>
    </Reference>
    <Reference Include="stdole">
        <HintPath>C:\Program Files\Autodesk\Inventor 2024\Bin\stdole.dll</HintPath>
    </Reference>
</ItemGroup>

<ItemGroup Condition="'$(ActiveDebugProfile)' == 'Inventor 2025'">
    <Reference Include="Autodesk.Inventor.Interop">
        <HintPath>C:\Program Files\Autodesk\Inventor 2025\Bin\Autodesk.Inventor.Interop.dll</HintPath>
        <Private>True</Private>
        <EmbedInteropTypes>False</EmbedInteropTypes>
    </Reference>
    <Reference Include="stdole">
        <HintPath>C:\Program Files\Autodesk\Inventor 2025\Bin\stdole.dll</HintPath>
    </Reference>
</ItemGroup>
```
These `ItemGroup` blocks will only be included in the build if the condition is met. The `$(ActiveDebugProfile)` variable holds the name of the currently selected debug profile. That’s why it’s important to give the profiles the correct names (e.g. **Inventor 2024** and **Inventor 2025**) when you create them — they’re referenced directly in these conditions.

This approach allows Visual Studio to automatically load the correct references depending on which version of Inventor you're debugging. It avoids the need to change references manually when switching versions, and helps keep your solution flexible and maintainable.

> 💡 You could also use conditions based on `$(TargetFramework)` (e.g. `'net48'` or `'net8.0-windows'`) instead of `$(ActiveDebugProfile)` if you prefer switching based on the build target rather than the selected debug profile.

## Managing Output Files

Under normal circumstances, you know exactly which files need to be copied and where they should go. However, when supporting multiple Inventor versions in a single project, you may want to copy different build outputs to different target folders — depending on the active debug profile.

In your project file (`.csproj`), you may already have one or more `Target` nodes. These are used to define **build events**, like post-build copy commands. A typical example might look like this:
```xml
<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  <Exec Command="xcopy &quot;...&quot; &quot;...&quot; /Y" />
</Target>
```
We’ll now modify this to support both Inventor 2024 and 2025 by using conditions and variables.

In this example, we assume the add-in is version-dependent. That means the compiled `.dll` must be copied to different folders depending on the Inventor version you're targeting. The output folder of the build also differs, since Visual Studio creates a separate subfolder for each framework target (e.g. `net48`, `net8.0-windows`).

To manage this, we define conditional `PropertyGroup` nodes based on the `$(ActiveDebugProfile)` variable. These set the correct source and destination paths:
```xml
<Target Name="PostBuild" AfterTargets="PostBuildEvent">
  
  <!-- PropertyGroup structure
  <PropertyGroup Condition="'$(ActiveDebugProfile)' == '[Debug profile name]'">
    <DebugFolder>[Build path]</DebugFolder>
    <InstallPath>[Addin Path for Inventor VERSION]</InstallPath>
  </PropertyGroup>
  -->
  
  <PropertyGroup Condition="'$(ActiveDebugProfile)' == 'Inventor 2024'">
    <DebugFolder>$(ProjectDir)bin\Debug\net48\</DebugFolder>
    <InstallPath>[Path for Inventor 2024]</InstallPath>
  </PropertyGroup>

  <PropertyGroup Condition="'$(ActiveDebugProfile)' == 'Inventor 2025'">
    <DebugFolder>$(ProjectDir)bin\Debug\net8.0-windows\</DebugFolder>
    <InstallPath>[Path for Inventor 2025]</InstallPath>
  </PropertyGroup>

  <Message Importance="high" Text="####### Installing addin from debug profile: $(ActiveDebugProfile) #######" />
  <Message Importance="high" Text="DebugFolder: $(DebugFolder)" />
  <Message Importance="high" Text="InstallPath: $(InstallPath)" />

  <Exec Command="xcopy /y &quot;$(DebugFolder)&quot; &quot;$(InstallPath)&quot;" />
</Target>
```
This configuration ensures that MSBuild uses the correct paths depending on which debug profile is active. The `<Message>` lines provide useful output during the build to confirm which paths are being used.

> 💡 Make sure the profile names exactly match those used in your debug configuration. If they don’t match, the condition won’t apply, and the file copy will not happen.

## Handling Code Differences Between Frameworks
When supporting both .NET Framework 4.8 and .NET 8.0, you may run into cases where certain code only works under one specific framework. To deal with this, you can use conditional compilation. This allows you to include or exclude code at compile-time, based on which framework is being targeted. For example:
```vb
#if NETFRAMEWORK
    System.Windows.MessageBox.Show("This message will only be shown if the target framework is .Net Framework")
#else
    System.Windows.MessageBox.Show("This message will only be shown if the target framework is .Net Core 8.0")
#endif
```
When building for .NET Framework 4.8, the NETFRAMEWORK symbol is automatically defined. When targeting .NET 8.0, it is not, so the code in the #else block will run instead.

> 💡 You can define additional symbols in your project file if you want to create more custom build conditions.























