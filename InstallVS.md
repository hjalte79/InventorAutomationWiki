# Installing Visual Studio for Inventor Automation

Most of the tutorials in this wiki assume you have **Visual Studio 2022** installed.

> ⚠️ **Note on Visual Studio vs. Visual Studio Code**  
> Starting with **Inventor 2025**, it's technically possible to create add-ins using **Visual Studio Code** and **C#**, because Inventor now targets **.NET 8 (Core)**. However, **Inventor 2024 and earlier** still use the older **.NET Framework (4.8 or lower)**, which requires the full **Visual Studio IDE**. This wiki focuses exclusively on using **Visual Studio** (the full IDE), which is still required for:
>
> - All **VB.NET** add-ins
> - All **Inventor 2024 and earlier** add-ins (both C# and VB.NET)
> - A streamlined experience when working with .NET-based automation

If you're just getting started or working with older Inventor versions, **Visual Studio 2022** is the recommended and supported environment for everything in this wiki.

This is not a full installation guide, but here are the key things you need to know:


## Download

You can download Visual Studio 2022 from the official site:  
👉 [https://visualstudio.microsoft.com/downloads/](https://visualstudio.microsoft.com/downloads/)

## .NET Framework vs .NET Core (Quick Explanation)

The term **.NET** refers to the software platform used to build and run applications like Inventor add-ins.

There are two main versions you need to know about:

- **.NET Framework** (e.g., version 4.8):  
  This is the **older**, Windows-only version of .NET.  
  Used by **Inventor 2024 and earlier**, and requires **Visual Studio** (the full IDE).

- **.NET Core / .NET 5+ / .NET 8**:  
  This is the **modern**, cross-platform version of .NET.  
  Starting with **Inventor 2025**, Autodesk switched to **.NET 8**, which makes it possible to use tools like **Visual Studio Code**.

> 💡 In short:  
> - If you're working with **Inventor 2024 or earlier**, you must use **.NET Framework 4.8**.  
> - For **Inventor 2025 and later**, you'll use **.NET 8**.  
> - This wiki always uses **Visual Studio** for a consistent development setup.

## Required Workloads

During installation, make sure to select the following:

- ✅ **workloads: .NET Desktop Development**
- ✅ **Individual component: .NET 8.0 Runtime** (For Inventor 2025 and later)
- ✅ **Individual component: .NET Framework 4.8 SDK** (For Inventor 2024 and earlier)

You can modify these settings later using the Visual Studio Installer if needed.

## After Installation

Once Visual Studio is installed, you're ready to begin your Inventor automation journey—whether you're building add-ins, writing external tools, or exploring the Inventor API.

> 📘 Start here: [Getting Started with Inventor Automation](./getting-started.md)