# Getting Started

## Install .NET Core

The .NET Core 2.0 SDK is needed for development. You can download it at [microsoft.com](https://www.microsoft.com/net/download/core).

To check that it works, run `dotnet --version` . It should return `2.0.0` , or a later version.

Optionally, you can also install  [.NET Core 1.0.4 SDK 1.1.1](https://github.com/dotnet/core/blob/master/release-notes/download-archives/1.0.4-download.md) and [.NET 4.6.1](https://support.microsoft.com/en-us/help/3102438/the--net-framework-4-6-1-web-installer-for-windows) if you want to build for those frameworks. .NET 4.6.1 can only be installed on Windows.

## Install Boost and SWI-prolog on Linux

If your development environment is Linux, install the [boost 1.58 libraries](http://www.boost.org/users/history/version_1_58_0.html) and [SWI-Prolog](http://www.swi-prolog.org/) packages. 

On Ubuntu, this can be done by executing this:  


```bash
sudo apt-get install -qy libboost1.58-all-dev swi-prolog-nox
```

## Download an Editor

To work with Starcounter on .NET Core, we recommend to use either [Visual Studio Code](https://code.visualstudio.com/) or [Visual Studio 2017](https://www.visualstudio.com/downloads/) as they are well integrated with .NET Core. The version for Visual Studio 2017 should be 15.3 or later.

## Summary

With this, everything is ready to start development with Starcounter on .NET Core. There's no need to install Starcounter separately - it's installed as a NuGet package in your .NET Core projects. 

