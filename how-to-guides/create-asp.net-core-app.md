# Create ASP.Net Core App

To create an ASP.Net Core application with Starcounter as a database, follow these steps:

## 1. Create Project

With .NET Core 2.0 SDK installed, run:

```text
dotnet new mvc -o WebApp
```

This will create a new ASP.Net Core MVC app in the directory `WebApp`.

## 2. Add Starcounter as a Dependency

### Create NuGet.Config

To add Starcounter as a dependency, we have to specify from where Starcounter can be retrieved. 

Create a NuGet.Config file in the root of the `WebApp` directory created above and insert this:

```markup
<?xml version="1.0" encoding="utf-8"?>
<configuration>
  <packageSources>
    <add key="Starcounter" value="https://www.myget.org/F/starcounter/api/v2" />
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
  </packageSources>
</configuration>
```

Now, we can add the Starcounter assemblies:

```text
dotnet add package runtime.native.Starcounter.Bluestar -v 2.4.0-*
dotnet add package Starcounter.Core.AspNetCore
```

The `WebApp.csproj` should now look like this:  


```markup
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>netcoreapp2.0</TargetFramework>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Microsoft.AspNetCore.All" Version="2.0.0" />
    <PackageReference Include="runtime.native.Starcounter.Bluestar" Version="2.4.0-*" />
    <PackageReference Include="Starcounter.Core.AspNetCore" Version="0.6.0" />
  </ItemGroup>
  <ItemGroup>
    <DotNetCliToolReference Include="Microsoft.VisualStudio.Web.CodeGeneration.Tools" Version="2.0.0" />
  </ItemGroup>
</Project> 
```

## 3. Initialize Database

To initialize the database with the correct file, add the following code at the start of the `Main` method:  


```csharp
const string databaseName = "defaultDatabase";

if (!Starcounter.Core.Options.StarcounterOptions.TryOpenExisting(databaseName))
{
    Directory.CreateDirectory(databaseName);
    Starcounter.Core.Bluestar.ScCreateDb.Execute(databaseName);
}
```

More information about this code can be found at [Creating the Database](../topic-guides/database/#creating-the-database).

## 4. Add Starcounter as a Service

To add Starcounter as a service, modify `BuildWebHost` to take an `IAppHost` as an argument and add Starcounter with the specified app host to the services:

```csharp
public static IWebHost BuildWebHost(
    string[] args, 
    Starcounter.Core.Abstractions.IAppHost appHost) 
{
    return WebHost.CreateDefaultBuilder(args)
        .UseStartup<Startup>()
        .ConfigureServices(services => services.AddStarcounter(appHost))
        .Build();
}
```

Then, in the `Main` method, create the app host with the database and pass it to `BuildWebHost`:

```csharp
using (var appHost = new Starcounter.Core.Hosting.AppHostBuilder()
    .UseDatabase(databaseName)
    .Build())
{
    BuildWebHost(args, appHost).Run();
}
```

## 5. Use Starcounter

With this in place, the Starcounter database can be used freely. For example, we can add this \(completely meaningless\) code in the `About` controller:

```csharp
public IActionResult About()
{
    Db.Transact(() =>
    {
        var person = Db.Insert<Person>(); // Create Person in database
        person.Name = "Gandalf";
    });

    var name = string.Empty;
    Db.Transact(() =>
    {
        var person = Db.SQL<Person>("SELECT p FROM Person p").FirstOrDefault(); // Query for the added Person
        name = person.Name;
    });

    ViewData["Message"] = $"Hi, my name is {name}";

    return View();
}
```

