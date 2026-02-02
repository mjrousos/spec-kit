---
name: http-context-upgrading
description: Addressing HttpContext dependency in class libraries while upgrading from ASP.NET (.NET Framework) to ASP.NET Core.
---

# Upgrading HttpContext Dependencies in Class Libraries

To handle scenarios where HttpContext is used in class libraries, install the [Microsoft.AspNetCore.SystemWebAdapters](https://github.com/dotnet/systemweb-adapters) package to make System.Web.HttpContext and other common System.Web APIs available to .NET Core projects. 

Remove the System.Web reference from the project and replace it with the SystemWebAdapters package.

**Do**
- Use NuGet tools to find the latest non-preview version of the `Microsoft.AspNetCore.SystemWebAdapters` and install it.
- Use the SystemWebAdapters package to replace existing System.Web references in the project (since System.Web is not available in .NET Core).

**Do Not**
- Comment out or remove existing HttpContext usage without replacing it with the SystemWebAdapters package.

## Handling HttpContext in ASP.NET Core Projects (as opposed to class libraries)

Web projects should be upgraded side-by-side and will have access to the full ASP.NET Core HttpContext API. Because of this, consider updating code to use 
the ASP.NET Core HttpContext API directly instead of relying on the SystemWebAdapters package.

If the code changes are significant, you can consider using the `Microsoft.AspNetCore.SystemWebAdapters` package temporarily to make the upgrade easier.

If the ASP.NET Core app depends on libraries that use System.Web adapters to access HttpContext, the web project will need to make sure to configure the SystemWebAdapters middleware in `program.cs` or `startup.cs`:

```csharp
var builder = WebApplication.CreateBuilder(args);

builder.Services.AddSystemWebAdapters();

var app = builder.Build();
app.UseRouting();
app.UseSystemWebAdapters();
app.Run();
```

> Note that ASP.NET Core apps using System.Web adapters will need to reference the [Microsoft.AspNetCore.SystemWebAdapters.CoreServices](https://www.nuget.org/packages/Microsoft.AspNetCore.SystemWebAdapters.CoreServices) package rather than just Microsoft.AspNetCore.SystemWebAdapters in order to have access to the necessary services and middleware.

## Additional Resources

- For more information on SystemWebAdapters, see the [Microsoft.AspNetCore.SystemWebAdapters documentation](https://github.com/dotnet/systemweb-adapters#readme).
