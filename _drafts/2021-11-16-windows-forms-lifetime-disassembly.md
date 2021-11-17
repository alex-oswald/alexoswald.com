---
title: "Windows Forms Lifetime Disassembly"
author: "Alex Oswald"
date: 2021-08-08
---


In this post I'm going to disassemble the Windows Forms Lifetime library I've started writing. The library is a Windows Forms hosting extension for the .NET Generic Host. It enables you to configure the generic host to use the lifetime of Windows Forms.

https://github.com/alex-oswald/WindowsFormsLifetime


## Understading how Windows Forms works

To get started with this project, we really need to understand how Windows Forms works in a standard application.

With VS 2022 and .NET 6, creating a new Windows Forms application using the template generates a `Program.cs` like the following.

```csharp
namespace WinFormsApp1
{
    internal static class Program
    {
        /// <summary>
        ///  The main entry point for the application.
        /// </summary>
        [STAThread]
        static void Main()
        {
            ApplicationConfiguration.Initialize();
            Application.Run(new Form1());
        }
    }
}
```

**Going over what we have here**

The [`STAThreadAttribute`](https://docs.microsoft.com/en-us/dotnet/api/system.stathreadattribute?view=net-5.0) says that the COM threading model for an application is single-threaded apartment (STA).

This is required because C# applications COM threading model is multithreaded apartment by default. Windows Forms must use single-threaded apartment for specific functions to work.

`ApplicationConfiguration.Initialize()` is just a source-generated bootstrap of what was in the previous Windows Forms template. It includes a few things to set on the `Application` object before running.

Next, we invoke `Application.Run()`. Create a new instance of `Form1`. Then we run the application, passing in the instance we want to be set as the main form. The main form is the first form shown and controls when the application exits. The `Application` object is static, and controls all aspects of the Windows Forms application. We will use this object quite a bit.

### Take away

- The thread we spawn to run Windows Forms needs to have its COM threading model set to single-threaded apartment
- Run the application bootstrap code before running.
- The application should be run in its o








https://docs.microsoft.com/en-us/dotnet/core/extensions/generic-host

https://stackoverflow.com/questions/1361033/what-does-stathread-do

https://web.archive.org/web/20090611124052/http://www.sellsbrothers.com/askthewonk/Secure/WhatdoestheSTAThreadattri.htm