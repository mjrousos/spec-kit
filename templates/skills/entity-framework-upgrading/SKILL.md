---
name: entity-framework-upgrading
description: Addressing Entity Framework dependency while upgrading from .NET Framework to .NET Core.
---

# Upgrading Entity Framework Dependencies

When upgrading .NET Framework applications that use Entity Framework (EF) to .NET Core, it's important to note that recent EF 6 versions support .NET Core. To simplify upgrade and minimize code changes, apps should generally *continue using EF 6* rather than migrating to EF Core.

## Changes required

To upgrade EF 6 apps to .NET Core, the following changes are typically required:

1. **Update NuGet Packages** - Update the EntityFramework NuGet package to version 6.5.1 or later.
1. **Update DB Context Connection String** - If the apps' `DbContext` specifies a connection by name in a call to its base constructor (e.g. `: base("name=MyConnectionString")`), it will need to be updated to use the full connection string instead which should typically be passed in by the caller.
    1. Old way (not supported in .NET Core):
    ```csharp
    public class CatalogDBContext : DbContext
    {
        public CatalogDBContext() : base("name=CatalogDBContext")
        {
        }
        ...
    }
    ```
    2. New way (supported in .NET Core):
    ```csharp
    public class CatalogDBContext : DbContext
    {
        public CatalogDBContext(string connectionString) : base(connectionString)
        {
        }
        ...
    }
    ```
1. **Update Config-based Configuration** - If the app uses an `<entityFramework>` section of `app.config` or `web.config` to configure EF (for example, to specify the database provider), this configuration will need to be moved to code since .NET Core does not support config-based EF configuration. This is described in more detail in [EF 6 documentation](https://learn.microsoft.com/ef/ef6/fundamentals/configuring/config-file).
