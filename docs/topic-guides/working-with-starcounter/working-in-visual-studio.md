# Working in Visual Studio

## Introduction

Developing Starcounter applications is straightforward and not tied to any certain development environment. Applications targeting .NET are normally built using Visual Studio, but would work using just `msbuild` or even the C\# compiler too.

In Visual Studio, applications can be built using a [Starcounter Visual Studio extension](https://marketplace.visualstudio.com/items?itemName=Starcounter.StarcounterforVisualStudio), or just using standard Visual Studio templates such as the C\# "Console Application" project template.

## Using the Visual Studio extension

Starcounter provides a Visual Studio extension to simplify the development of Starcounter applications. It provide templates to build on and the ability to start applications from Visual Studio. The extension can either be installed during the installation process of Starcounter by choosing that as an option, or by manually downloading and installing it from the [Visual Studio Marketplace](https://marketplace.visualstudio.com/items?itemName=Starcounter.StarcounterforVisualStudio).

To read more about the recent changes and a step by step guide to create starcounter application, refer to the annoucement about [Starcounter.VisualStudio 3.0.0](https://starcounter.io/starcounter-visualstudio-3-0-0-now-available-vs-marketplace/).

### Templates

There are currently two project templates and three item templates included in the extension.

**Project templates**:

The project templates are used to scaffold projects that target Starcounter. When instantiated, the project will have some default Starcounter-specific _assembly references_ set up and include some build hooks that will aid in building. For more information on the specifics, see section about [creating applications using standard templates](../../#create-a-starcounter-application-using-standard-templates) later in this text.

Project templates include:

* Starcounter Application
* Starcounter Class Library

**Item templates**:

* Starcounter HTML template with dom-bind
* Starcounter Typed JSON
* Starcounter Typed JSON with code-behind

#### Starcounter Application Template

The Starcounter application template is the starting point to creating applications with Starcounter. It contains four references: `Starcounter`, `Starcounter.Internal`, `Starcounter.Hosting`, and `Starcounter.XSON`. Additionally, it comes with a boilerplate `Program.cs` file that looks like this:

```csharp
using System;
using Starcounter;

namespace MyApp
{
    class Program
    {
        static void Main()
        {

        }
    }
}
```

#### Starcounter Class Library

The Starcounter class library template is the starting point for creating a shared data model to use across applications. For example, the [Simplified](https://github.com/Starcounter/Simplified) DLL that is used to provide a shared data model to the Starcounter [sample apps](https://github.com/Starcounter) is built with this template. It contains the same references as the Starcounter application template. This is how the boilerplate `Program.cs` file looks:

```csharp
using System;
using Starcounter;

namespace MyClassLibrary
{
    [Database]
    public class Entity1
    {
        public string Field1 { get; set; }
    }
}
```

#### Starcounter HTML template with dom-bind

This item templates gives a starting point for creating HTML view definitions with Polymer.

It contains the following code:

```markup
<link rel="import" href="/sys/polymer/polymer.html">

<template>
    <template is="dom-bind">

    </template>
</template>
```

#### Starcounter Typed JSON

This template is the starting point for creating a view-model definition using JSON-by-example. It is simply an empty `.json` file containing an empty JSON object:

```javascript
{
}
```

#### Starcounter Typed JSON with code-behind

This template is the same as the Starcounter Typed JSON file, except that it also provides a code-behind file. Thus, two files are created with this template, `.json` and `.json.cs`.

The `.json` file is identical to the Starcounter Type JSON file. The `.json.cs` file contains the following code:

```csharp
using Starcounter;

namespace MyApp
{
    partial class MyPage : Json
    {
    }
}
```

## Starting apps from Visual Studio

With the Visual Studio Extension, apps can be started directly from the development environment. This is done the same way any other application would be started from Visual Studio, by clicking the `Start` button or f5.

Further instructions on starting applications in Visual Studio can be found in [Starting and Stopping Apps](starting-and-stopping-apps.md). There it is also described how it is possible to set particular arguments on application start from `Debug` -&gt; `MyApp Properties` -&gt; `Debug`.

## Using standard Visual Studio

As was established in the introduction, the Visual Studio extension \(or even Visual Studio\) is not a requirement for building and running Starcounter applications. You could very well create applications using just a text editor and msbuild, or even the compiler.

In this section, we'll see how to work with applications using just standard Visual Studio.

### Create a Starcounter app with standard templates

Using the standard C\# "Console Application" project, we can turn that into a proper Starcounter application with these simple steps:

1. Create the project, name it for example "Hello".
2. Edit the `Hello.csproj`
   1. Specify compatibility with "2.4".    
   2. Add references to Starcounter assemblies.
   3. Add an import to the Starcounter .targets file.

`Hello.csproj` \(snippet showing additions\)

```markup
<PropertyGroup>
  <StarcounterVersionCompatibility>2.4</StarcounterVersionCompatibility>
</PropertyGroup>

<ItemGroup>
  <Reference Include="Starcounter, Version=2.0.0.0, Culture=neutral, PublicKeyToken=d2df1e81d0ca3abf" />
  <Reference Include="Starcounter.Internal, Version=2.0.0.0, Culture=neutral, PublicKeyToken=d2df1e81d0ca3abf" />
  <Reference Include="Starcounter.Hosting, Version=2.0.0.0, Culture=neutral, PublicKeyToken=d2df1e81d0ca3abf" />
  <Reference Include="Starcounter.XSON, Version=2.0.0.0, Culture=neutral, PublicKeyToken=d2df1e81d0ca3abf" />
</ItemGroup>

<Import Project="$(StarcounterBin)\Starcounter.MsBuild.targets" />
```

Without the extension, you can't start the application project simply from within Visual Studio. Luckily, it's easily done using tooling being part of Starcounter.

1. Open a command line prompt.
2. CD to the directory of your built application.
3. Run `star Hello.exe`.

