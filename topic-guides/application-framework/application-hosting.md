# Application Hosting

## Introduction

Applications in Starcounter are self-hosted. This means that the host is configured and initialized by the developer. Starcounter supports two different configurations for self-hosted apps: injecting Starcounter into the pipeline of ASP.Net Core and using a console-based host.

## Creating the Host

The host is created and configured with the `AppHostBuilder` class. It's a fluent interface where the option methods are chained on the `AppHostBuilder`. The method chain should end with `Build`. We recommend to wrap the initialization of the host to be in a `using` statement so that it's cleaned up if an exception is thrown:

```csharp
using (var appHost = new AppHostBuilder()
    .UseDatabase("myDatabase")
    .AddCommandLine(args)
    .Build());
```

The host can then be started with `appHost.Start()`.

### Specify the Database

A database has to be specified for the app host, otherwise, `Build` will throw `SCERRDATABASENOTFOUND`.

The database can be specified by passing a command line argument or with the method `UseDatabase`. 

#### UseDatabase

With `UseDatabase` it looks like this:

```csharp
using System.IO;
using Starcounter.Core.Options;
using Starcounter.Core.Bluestar;
using Starcounter.Core.Hosting;

public class Program
{
    public static void Main(string[] args)
    {
        const string databaseName = "defaultDatabase";

        if (!StarcounterOptions.TryOpenExisting(databaseName))
        {
            Directory.CreateDirectory(databaseName);
            ScCreateDb.Execute(databaseName);
        }

        using (var appHost = new AppHostBuilder()
            .UseDatabase(databaseName)
            .Build())
        {
            appHost.Start();
        }
    }
} 
```

#### Command Line Argument

To do this with a command line argument, replace `UseDatabase(databaseName)` with `AddCommandLine(args)`.

Starting the app with `dotnet run defaultDatabase` will start the host with `defaultDatabase`. 

### Set the Scheduler Count

The `UseSchedulerCount` method lets you set the number of schedulers:

```csharp
using (var appHost = new AppHostBuilder()
    .UseDatabase(databaseName)
    .UseSchedulerCount(20)
    .Build())
```

The default value is the number of processors times two. The max value is 31 and the min value is 1. If the value passed to `UseSchedulerCount` is outside of that range, it will throw ` ArgumentOutOfRangeException`.

### Set Event Log Directory

`UseEventLogDirectory` lets you determine where the event log is created. By default, it's in the same directory as the database logs. For example, if we run an app that creates a database with the name `defaultDatabase`without `UseEventLogDirectory`, it will create four files in the `defaultDatabase` directory:

* `DEFAULTDATABASE.000000000000.log`
* `DEFAULTDATABASE.000000000001.log`
* `defaultDatabase.cfg`
* `starcounter.0000000000.log`

The last file, starting with `starcounter` is the event log. 

By creating a directory called `eventLog` and adding `UseEventLogDirectory("eventLog")` to the `AppHostBuilder` method chain, the event log file will be created in that directory instead.  

## Starcounter on ASP.Net Core

By injecting Starcounter into the pipeline of ASP.Net Core, it makes it possible to use the Starcounter database while using a familiar framework. The `Starcounter.Core.AspNetCore` assembly has everything needed to setup Starcounter with ASP.Net Core.

### Adding Starcounter as a Service

Starcounter is added as a service to ASP.Net Core the same way ASP.Net MVC is added. It's done in the`ConfigureServices` method when creating the web host or in `Startup.cs`.

`AddStarcounter`is the method to add Starcounter service. It's an extension method for the `IServiceCollection` interface that has three overloads that takes three different arguments:

1. The name of the database. This overload requires the least code since you don't have to create the app host yourself, but It's also the one with least flexibility since you can't set options such as scheduler count or event log directory.
2. The `AppHostBuilder`. It allows you to set the options and then let `AddStarcounter` call `Build` on it and then start the app host.
3.  The `IAppHost`. This means that you configure and call `Build` on the `AppHostBuilder` and then pass along the created app host and let `AddStarcounter` start it.

For example, to use the first overload to add Starcounter as a service, it looks like this:

```csharp
using Microsoft.AspNetCore;
using Microsoft.AspNetCore.Hosting;
using Starcounter.Core.AspNetCore;
using Starcounter.Core.Options;

public class Program
{
    public static void Main(string[] args)
    {
        const string databaseName = "defaultDatabase";

        if (!StarcounterOptions.TryOpenExisting(databaseName))
        {
            System.IO.Directory.CreateDirectory(databaseName);
            Starcounter.Core.Bluestar.ScCreateDb.Execute(databaseName);
        }
        
        WebHost.CreateDefaultBuilder(args)
            .UseStartup<Startup>()
            .ConfigureServices(services => 
                services.AddStarcounter(databaseName))
            .Build()
            .Run();
        }
    }
}
```

The two other overloads look like this:

```csharp
var appHostBuilder = new AppHostBuilder().UseDatabase(databaseName);
WebHost.CreateDefaultBuilder(args)
  .UseStartup<Startup>()
  .ConfigureServices(service => service.AddStarcounter(appHostBuilder))
  .Build()
  .Run();
```

```csharp
var appHost = new AppHostBuilder().UseDatabase(databaseName).Build();
WebHost.CreateDefaultBuilder(args)
  .UseStartup<Startup>()
  .ConfigureServices(service => service.AddStarcounter(appHost))
  .Build()
  .Run();
```

{% hint style="info" %}
For full instructions on how to inject Starcounter into an ASP.Net Core app, read [Create ASP.Net Core App](../../how-to-guides/create-asp.net-core-app.md)
{% endhint %}





