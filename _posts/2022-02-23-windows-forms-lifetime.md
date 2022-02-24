---
title: "Windows Forms Lifetime"
author: "Alex Oswald"
date: 2022-02-23
---


This post will go over the Windows Forms Lifetime library I've started writing. The library is a Windows Forms hosting extension for the .NET Generic Host. It enables you to configure the generic host to use the lifetime of Windows Forms, aka when the main Form closes, the host will shut down.

[https://github.com/alex-oswald/WindowsFormsLifetime](https://github.com/alex-oswald/WindowsFormsLifetime)


## Get started with .NET 6's Minimal API

Create a new **Windows Forms App**.

Change the projects SDK to `Microsoft.NET.Sdk.Web` so we can use the `WebApplication` class.

Add `NoDefaultLaunchSettingsFile` to the `csproj` so a `launchSettings.json` file isn't created automatically for us.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <OutputType>WinExe</OutputType>
    <TargetFramework>net6.0-windows</TargetFramework>
    <Nullable>enable</Nullable>
    <UseWindowsForms>true</UseWindowsForms>
    <ImplicitUsings>enable</ImplicitUsings>
    <NoDefaultLaunchSettingsFile>true</NoDefaultLaunchSettingsFile>
  </PropertyGroup>
</Project>
```

Replace the contents of `Program.cs` with the following.

```csharp
using WinFormsApp1;

var builder = WebApplication.CreateBuilder(args);
builder.Host.UseWindowsFormsLifetime<Form1>();
var app = builder.Build();
app.Run();
```


## Instantiating and Showing Forms

Add more forms to the DI container.

```csharp
using WinFormsApp1;

var builder = WebApplication.CreateBuilder(args);
builder.Host.UseWindowsFormsLifetime<Form1>();
builder.Services.AddTransient<Form2>();
var app = builder.Build();
app.Run();
```

To get a form use the `IFormProvider`. The form provider instantiates an instance of the form from the DI container on the GUI thread. `IFormProvider` has one method, `GetFormAsync<T>` used to fetch a form instance.

In this example, we inject `IFormProvider` into the main form, and use that to instantiate a new instance of `Form`, then show the form. Instantiating a Form directly in a nother form will also create it on the GUI thread, but using the `IFormProvider` is best practice when using this library.

```csharp
public partial class Form1 : Form
{
    private readonly ILogger<Form1> _logger;
    private readonly IFormProvider _formProvider;

    public Form1(ILogger<Form1> logger, IFormProvider formProvider)
    {
        InitializeComponent();
        _logger = logger;
        _formProvider = formProvider;
    }

    private async void button1_Click(object sender, EventArgs e)
    {
        _logger.LogInformation("Show Form2");
        var form = await _formProvider.GetFormAsync<Form2>();
        form.Show();
    }
}
```


## Invoking on the GUI thread

Sometimes you need to invoke an action on the GUI thread. Say you want to spawn a form from a background service. Use the `IGuiContext` to invoke
actions on the GUI thread.

In this example, a form is fetched and shown every 5 seconds 5 times, in an action that is invoked on the GUI thread. This example shows how the GUI thread does not lock up during this process.

```csharp
public class HostedService1 : BackgroundService
{
    private readonly IFormProvider _fp;
    private readonly IGuiContext _guiContext;

    public HostedService1(
        IFormProvider formProvider,
        IGuiContext guiContext)
    {
        _fp = formProvider;
        _guiContext = guiContext;
    }

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        int count = 0;
        while (!stoppingToken.IsCancellationRequested)
        {
            await Task.Delay(5000, stoppingToken);
            if (count < 5)
            {
                await _guiContext.InvokeAsync(async () =>
                {
                    var form = await _fp.GetFormAsync<Form2>();
                    form.Show();
                });
            }
            count++;
        }
    }
}
```


## Conclusion

I've explained that the `WindowsFormsLifetime` library lets you use .NET's Generic Host with Windows Forms application, and how it can control the lifetime of the application. I demonstrated how to get started using .NET 6's Minimal API. I also showed the two main services that come with the library, the `IFormProvider` which is used to new up forms on the GUI thread, and the `IGuiContext` which is used to invoke actions on the GUI thread to prevent cross-thread exceptions.

In a future post, I will demonstrate how to use the `WindowsFormsLifetime.Mvp` companion library to develop Model-View-Presenter applications using Windows Forms.

Until then, happy win forming!