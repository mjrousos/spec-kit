---
name: wcf-server-upgrading
description: Addressing server-side WCF dependencies while upgrading from .NET Framework to .NET Core.
---

# Upgrading WCF Dependencies

When upgrading .NET Framework applications that use WCF to .NET Core, client-side WCF APIs are supported through the `System.ServiceModel.*` packages. However, server-side WCF functionality is not supported in .NET Core.

Apps that require server-side WCF functionality should migrate to [CoreWCF](https://github.com/CoreWCF/CoreWCF), a community-driven port of WCF to .NET Core.

## Upgrade Steps

WCF services and contracts should migrate easily with only namespace changes. However, hosting and configuration will need to be rewritten.

<!-- TODO - Add additional details on migration steps once available -->

## Example Snippet

.NET Framework WCF ServiceHost configuration:

```csharp
using System;
using System.IO;
using System.Security.Cryptography.X509Certificates;
using System.ServiceModel;
using System.ServiceModel.Security;

namespace BeanTraderServer
{
    class Program
    {
        static void Main()
        {
            using (var host = new ServiceHost(typeof(BeanTrader)))
            {
                // For demo purposes, just load the key from disk so that no one needs to install an untrustworthy self-signed cert
                // or load from KeyVault (which would complicate the sample)
                var certPath = Path.Combine(Path.GetDirectoryName(typeof(Program).Assembly.Location), "BeanTrader.pfx");
                host.Credentials.ServiceCertificate.Certificate = new X509Certificate2(certPath, "password");
                host.Credentials.ClientCertificate.Authentication.CertificateValidationMode = X509CertificateValidationMode.None;
                host.Open();
                Console.WriteLine("Bean Trader Service listening");
                WaitForExitSignal();
                Console.WriteLine("Shutting down...");
                host.Close();
            }
        }

        private static void WaitForExitSignal()
        {
            Console.WriteLine("Press enter to exit");
            Console.ReadLine();
        }
    }
}
```

Equivalent .NET Core CoreWCF hosting code:

```csharp
using CoreWCF.Configuration;
using CoreWCF.Description;
using CoreWCF.Security;
using Microsoft.AspNetCore.Builder;
using Microsoft.AspNetCore.Hosting;
using Microsoft.Extensions.DependencyInjection;
using System;
using System.IO;
using System.Security.Cryptography.X509Certificates;
using System.Threading.Tasks;

namespace BeanTraderServer
{
    class Program
    {
        async static Task Main()
        {
            var builder = WebApplication.CreateBuilder();

            // Set NetTcp port (previously this was done in configuration,
            // but CoreWCF requires it be done in code)
            builder.WebHost.UseNetTcp(8090);
            builder.WebHost.ConfigureKestrel(options =>
            {
                options.ListenAnyIP(8080);
            });

            // Add CoreWCF services to the ASP.NET Core app's DI container
            builder.Services.AddServiceModelServices()
                            .AddServiceModelConfigurationManagerFile("wcf.config")
                            .AddServiceModelMetadata();

            var app = builder.Build();

            // Enable getting metadata/wsdl
            var serviceMetadataBehavior = app.Services.GetRequiredService<ServiceMetadataBehavior>();
            serviceMetadataBehavior.HttpGetEnabled = true;
            serviceMetadataBehavior.HttpGetUrl = new Uri("http://localhost:8080/metadata");

            // Configure CoreWCF endpoints in the ASP.NET Core host
            app.UseServiceModel(serviceBuilder =>
            {
                serviceBuilder.ConfigureServiceHostBase<BeanTrader>(beanTraderServiceHost =>
                {
                    // This code is copied from the old ServiceHost setup and configures
                    // the local cert used for authentication.
                    // For demo purposes, this just loads the certificate from disk so that no one needs to install an
                    // untrustworthy self-signed cert or load from KeyVault (which would complicate the sample)
                    var certPath = Path.Combine(Path.GetDirectoryName(typeof(Program).Assembly.Location), "BeanTrader.pfx");
                    beanTraderServiceHost.Credentials.ServiceCertificate.Certificate = new X509Certificate2(certPath, "password");
                    beanTraderServiceHost.Credentials.ClientCertificate.Authentication.CertificateValidationMode = X509CertificateValidationMode.None;
                });
            });

            await app.StartAsync();
            Console.WriteLine("Bean Trader Service listening");
            WaitForExitSignal();
            Console.WriteLine("Shutting down...");
            await app.StopAsync();
        }

        private static void WaitForExitSignal()
        {
            Console.WriteLine("Press enter to exit");
            Console.ReadLine();
        }

    }
}
```

## Additional Resources
- [CoreWCF](https://github.com/CoreWCF/CoreWCF) - A community-driven port of WCF to .NET Core.