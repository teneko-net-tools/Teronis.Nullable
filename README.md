# Teronis.Nullable [![Nuget](https://img.shields.io/nuget/v/Teronis.Nullable)][NuGet Package]

_Use .NET Core 3.0's new nullable attributes in older target frameworks._

[:running: Quick Start Guide](#quick-start-guide) &nbsp; | &nbsp; [:books: Guides](https://github.com/manuelroemer/Nullable/wiki) &nbsp; | &nbsp; [:package: NuGet Package][NuGet Package]

## Prologue

This project (Teronis.Nullable) is a fork of [Nullable][Nullable]. In fact the project Nullable is in the SDK format but the package creation is not in the SDK format and uses nuget.exe directly. This fork has just moved the NuGet part to the new SDK format. Morover is the the unattractive and separate build process obsolete now. The project Teronis.Nullable is in such a way designed to be includable by ProjectReference and is heavily used by Teronis.Dotnet.

**When you face any issues with Teronis.Nullable please do not open an issue in the repository of Nullable but open an issue [here](https://github.com/teroneko/Teronis.DotNet/issues).**

## About Teronis.Nullable

With the release of C# 8.0, support for [nullable reference types](https://docs.microsoft.com/en-us/dotnet/csharp/tutorials/nullable-reference-types)
has been added to the language.
Futhermore, .NET Core 3.0 added [new nullable attributes](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis?view=netcore-3.0)
like the [`AllowNullAttribute`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis.allownullattribute?view=netcore-3.0)
which are sometimes required to exactly declare when and how `null` is allowed in specific code
sections.

Unfortunately, these attributes are not available in older target frameworks like .NET Standard 2.0
which makes annotating existing code harder.
Luckily, this problem can be solved by re-declaring the attributes as `internal class`es - the C#
compiler will still use them for generating warnings, even though the target framework doesn't
support these attributes by itself.

This repository hosts the code for the ["Teronis.Nullable" NuGet Package][NuGet Package]
which, when referenced, adds support for these attributes.

The code for these attributes is added **at compile time** and gets **built into the referencing project**.
This means that the resulting project **does not have an explicit dependency** on the `Teronis.Nullable`
package, because the attributes are not distributed as a library.

Futhermore, the code only gets added to the project if the nullable attributes are not already
supported by the target framework.
The images below show an example - in the library which targets .NET Standard 2.0, the attributes
have been added during the compilation.
That is not the case for the library targeting .NET Standard 2.1, because the attributes are
available through the .NET BCL there.
This allows you to easily **multi-target** your projects without having to change a single line of
code.

| .NET Standard 2.0 | .NET Standard 2.1 |
| ----------------- | ----------------- |
| ![.NET Standard 2.0](img/CompiledNetStandard2.0.png) | ![.NET Standard 2.1](img/CompiledNetStandard2.1.png) |


## Compatibility

Nullabe is currently compatible with the following target frameworks:

* .NET Standard >= 1.0
* .NET Framework >= 2.0

Please have a look at the [guides](https://github.com/manuelroemer/Nullable/wiki) for additional information on how to
install the package for your target framework.


## Quick Start Guide

> :warning: **Important:** <br/>
> You **must** use a C# version >= 8.0 with the `Teronis.Nullable` package - otherwise, your project won't compile.

The steps below assume that you are using the **new SDK `.csproj`** style.
Please find installation guides and notes for other project types (for example `packages.config`)
[here](https://github.com/manuelroemer/Nullable/wiki).

1. **Reference the package** <br/>
   Add the package to your project, for example via:

   ```sh
   Install-Package Teronis.Nullable

   --or--

   dotnet add package Teronis.Nullable
   ```
2. **Ensure that the package has been added as a development dependency** <br/>
   Open your `.csproj` file and ensure that the new package reference looks similar to this:

   ```xml
   <PackageReference Include="Teronis.Nullable" Version="<YOUR_VERSION>">
     <PrivateAssets>all</PrivateAssets>
     <IncludeAssets>runtime; build; native; contentfiles; analyzers; buildtransitive</IncludeAssets>
   </PackageReference>

   <!-- This style also works, but is not automatically used by .NET: -->
   <PackageReference Include="Teronis.Nullable" Version="<YOUR_VERSION>" PrivateAssets="all" />
   ```

   This is especially important for libraries that are published to NuGet, because without this,
   the library will have an **explicit dependency** on the `Nullable` package.
3. **Build the project** <br/>
   Ensure that the project compiles. If a build error occurs, you will most likely have to update
   the C# language version (see next step).
4. **Enable usage of the attributes** <br/>
   Still in your `.csproj` file you need to activate the feature to fully use it.
   The following activation sample is what seems to be the most common use case. But do not hesitate to look at my [guides](https://github.com/manuelroemer/Nullable/wiki) for other considerations.

   ```xml
   <PropertyGroup>
      <!-- a sample list of target frameworks including legacy ones. This list must depend on your needs. -->
     <TargetFrameworks>net472;netstandard2.0;netstandard2.1;netcoreapp3.0</TargetFrameworks>
     <!-- force lang version at least to C#8 to be able to use the feature with all frameworks.-->
     <LangVersion>8.0</LangVersion> <!-- or --> <LangVersion>latest</LangVersion>
     <!-- enable the feature but without warning for legacy frameworks to avoid false positives. -->
     <Nullable>annotations</Nullable>
     <!-- enable the feature including warnings only on top of the most recent framework you target. -->
     <!-- it will serve as a reference for warnings for all others. -->
     <Nullable Condition="'$(TargetFramework)' == 'netcoreapp3.0'">enable</Nullable>
   </PropertyGroup>
   ```

5. **Build the project and fix warnings** <br/>
   If you're not starting a new project you will probably get a lot of warnings now, since your code base is not yet annotated.
   If you don't expect to fix all right now, one solution could be to disable the feature in all files before reviewing each file one by one.
   For that run the following powershell script to add a `#nullable disable` directive at the top of each file.

   ```PowerShell
   Get-ChildItem -Recurse -Filter *.cs | ForEach-Object {
     "#nullable disable`n" + (Get-Content $_ -Raw) | Set-Content $_
   }
   ```

You should now be ready to play with nullable references and the attributes even when targeting legacy frameworks.


## Compiler Constants

The [included C# files](src/)
makes use of some compiler constants that can be used to enable or disable certain features.

### `NULLABLE_ATTRIBUTES_DISABLE`

If the `NULLABLE_ATTRIBUTES_DISABLE` constant is defined, the attributes are excluded from the build.
This can be used to conditionally exclude the attributes from the build if they are not required.

In most cases, this should not be required, because the package automatically excludes the attributes
from target frameworks that already support these attributes.


### `NULLABLE_ATTRIBUTES_INCLUDE_IN_CODE_COVERAGE`

Because the attributes are added as source code, they could appear in code coverage reports.
By default, this is disabled via the [`ExcludeFromCodeCoverage`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis.excludefromcodecoverageattribute?view=netcore-3.0)
and [`DebuggerNonUserCode`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.debuggernonusercodeattribute?view=netcore-3.0)
attributes.

By defining the `NULLABLE_ATTRIBUTES_INCLUDE_IN_CODE_COVERAGE` constant, the [`ExcludeFromCodeCoverage`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.codeanalysis.excludefromcodecoverageattribute?view=netcore-3.0)
and [`DebuggerNonUserCode`](https://docs.microsoft.com/en-us/dotnet/api/system.diagnostics.debuggernonusercodeattribute?view=netcore-3.0)
attributes are not applied and the nullable attributes may therefore appear in code coverage reports.


## Building

No outer build process needed, just run:

```
dotnet build Teronis.Nullable.csproj
```

The MSBuild-target **CopyToStaticAnalyseAttributeCSPPFilesFolder** is running before MSBuild-target `AddStaticAnalyseAttributesCSPPFilesToPackage` and copies the .cs-files to folder `obj/StaticAnalysisAttributes` with new extension ".cs.pp".

The MSBuild-target **AddStaticAnalyseAttributesCSPPFilesToPackage** is running before `Pack` and adds dependent on framework the necessary files in their respective framework folders; once for folder content (package) and once for folder contentFiles (package).

## Contributing

As the Author of [Nullable][Nullable] stated, the source files itself won't change much, when ever, but when you have ideas, features or bug fixes, feel free to send me a pull request.

## License

The original project is licensed under the MIT license, this won't change. See updated [LICENSE](LICENSE) file for details.

[Nullable]: https://github.com/manuelroemer/Nullable
[NuGet Package]: https://www.nuget.org/packages/Teronis.Nullable