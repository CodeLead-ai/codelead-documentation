---
title: Build .NET Framework projects on Windows
description: Set up a Windows machine so that CodeLead can build and test an old .NET Framework project, and check what CodeLead found.
kind: how-to
area: how-to
date: 2026-11-20
---

# Build .NET Framework projects on Windows

This guide sets up a Windows machine so that CodeLead can build an old .NET Framework project, such as the Basic Store sample's `webapp`. CodeLead builds such a project with Visual Studio's MSBuild, and runs its tests with `dotnet test`. It finds MSBuild by itself once it is installed.

**In short**
- You know why `dotnet build` cannot build an old web project, and what CodeLead uses in its place.
- You install the Build Tools with the web workload, or Visual Studio with the web workload.
- You check what CodeLead found with `codelead doctor`, and you can declare your own build commands.

## Before you begin

- A Windows machine. Visual Studio's MSBuild runs on Windows only. On macOS or Linux, CodeLead refuses the pins and the transforms of such a project before any model call.
- [CodeLead installed](../get-started/install.md), with the .NET SDK. `dotnet test` runs the tests.
- The project's own .NET Framework version, for example 4.5. Read it in the dossier's `modernization-readiness.md`, or in the project file's `TargetFrameworkVersion`.

## Why dotnet build cannot build an old web project

An old-style web project imports Visual Studio's web targets: `$(VSToolsPath)\WebApplications\Microsoft.WebApplication.targets`. The .NET SDK resolves that path into its own folder, where the file does not exist. So `dotnet build` fails before it compiles anything, with MSB4019:

```output
WebApplication1\webapp.csproj(327,3): error MSB4019: The imported project "C:\Program Files\dotnet\sdk\11.0.100\Microsoft\VisualStudio\v18.0\WebApplications\Microsoft.WebApplication.targets" was not found.
```

The same happens to any project that references such a project, for example the pins' test project. CodeLead reads this from the project files themselves: the old format, the imports that only Visual Studio carries, and the project type. It never guesses from a project's name.

## Install the build tools

Install one of these:

- **Visual Studio** 2022 or later, any edition, with the workload **ASP.NET and web development**. Use this on a developer's machine. It also installs IIS Express, which runs the old application.
- **Build Tools for Visual Studio**, with the workload **Web development build tools**. Its id is `Microsoft.VisualStudio.Workload.WebBuildTools`. Use this on a build machine, where nobody opens the code.

Then make sure that the project's .NET Framework version has its reference assemblies. When a build says `MSB3644` (the reference assemblies for a version were not found), install that version's developer pack or targeting pack.

> [!IMPORTANT]
> MSBuild's `-restore` does not restore the packages of a `packages.config` file. Build the solution once in Visual Studio, which restores them, before CodeLead builds it.

## Check what CodeLead found

Run [`codelead doctor`](../cli/codelead-doctor.md). It lists MSBuild beside the .NET SDK. When CodeLead finds MSBuild, the row is `ok`, with the path and how it was found:

<!-- described, not captured -->
```output
ok       .NET SDK (for .NET projects)                          SDK 10.0.302
ok       MSBuild (Visual Studio, for .NET Framework projects)  found at <Visual Studio folder>\MSBuild\Current\Bin\MSBuild.exe (vswhere)
```

When it does not find MSBuild, the row says so. On Windows, the fix names the workload to install, and the places that CodeLead looked. On a Mac, the row reads:

```output
missing  MSBuild (Visual Studio, for .NET Framework projects)  not found: needed only for non-SDK .NET Framework projects  → Visual Studio's MSBuild runs on Windows only; a non-SDK .NET Framework project is built there (or declare its commands in .codelead/validation.json).
```

The row is not required: a folder with no .NET Framework project does not need MSBuild.

### Where CodeLead looks

On Windows, CodeLead looks in this order, and keeps the first MSBuild that it finds:

1. **vswhere**, the tool that the Visual Studio installer puts in `%ProgramFiles(x86)%\Microsoft Visual Studio\Installer\`. CodeLead asks it for the latest instance that has MSBuild: `vswhere -latest -products * -requires Microsoft.Component.MSBuild -find MSBuild\**\Bin\MSBuild.exe`. This finds every edition and the Build Tools.
2. **`MSBuild.exe` on the PATH**, as in a Developer Command Prompt.
3. **The Visual Studio folders**: 2026 (folder `18`), 2022, 2019 and 2017, in each edition: Enterprise, Professional, Community, BuildTools, Preview and Insiders.

On another system, CodeLead looks only for Mono's `msbuild` on the PATH.

## See what the gate runs

For a project of this class, [the gate](../concepts/validation-and-the-gate.md) runs two commands in place of `dotnet build`:

```text
<MSBuild.exe> <the project or the solution> -restore -nologo -v:minimal
dotnet test <the test project> --no-build -nologo
```

The scope's row in the validation says why. In CodeLead's tests, on a copy of the Basic Store sample's project, it reads:

```output
MSBuild (Visual Studio): the project references a .NET Framework web project that the .NET SDK cannot build (webapp: non-SDK, v4.5, web application project type)
```

A test project in the old format cannot run under `dotnet test`. The row then says that its tests are not run, and names the runner to declare: `vstest.console.exe`.

### What CodeLead does about MSB4019

CodeLead recovers before it asks you:

- **MSBuild found**: a `dotnet build` or `dotnet test` that fails with MSB4019 on Visual Studio's targets is run once more with MSBuild. The output says so: `dotnet build could not evaluate Legacy.Web\Legacy.Web.csproj (MSB4019); building with Visual Studio's MSBuild instead.`
- **MSBuild not found**: the gate does not run `dotnet build`, because it could only fail. It stops with `Visual Studio's MSBuild is needed and was not found`, what it looked for, and the ways out.

The pins have one more way out. Sometimes MSBuild is missing, but a Windows build left the project's assembly in its `bin` folder. Then the pins reference that assembly in place of the project.

See [Messages and what to do](../reference/messages.md) for each message.

## Declare your own commands

When CodeLead's choice does not fit, declare the commands yourself in `.codelead/validation.json`. A declared scope replaces what CodeLead detects. A bare `msbuild` command is found the same way as above, so the file works on any Windows machine. For example, to build the Basic Store sample's solution first:

```json
{
  "schema": "codelead.validation/1",
  "scopes": {
    ".": [
      {
        "label": "build sampleapp.slnx (MSBuild)",
        "command": "msbuild",
        "args": ["sampleapp.slnx", "-restore", "-nologo", "-v:minimal"]
      }
    ]
  }
}
```

Keep the entries that the pins added to this file. See [Declare validation commands](declare-validation-commands.md) and [Validation file](../reference/validation-file.md).

## Check that it worked

1. Run `codelead doctor`. The MSBuild row is `ok`.
2. In the chat of the old application, run [`/validate`](../chat-commands/validate.md). The scope's row names MSBuild, and its build passes.
3. Run the pins: [`/modernize --pin webapp`](../chat-commands/modernize.md#--pin). They no longer stop with `The pins of webapp need Visual Studio's MSBuild, and it is not on this machine.`

## See also

- [Pins](../modernization/pins.md)
- [Transforms](../modernization/transforms.md)
- [`codelead doctor`](../cli/codelead-doctor.md)
- [Declare validation commands](declare-validation-commands.md)
- [Validation and the gate](../concepts/validation-and-the-gate.md)
- [Start the old application for CodeLead](../modernization/source-run-profile.md)
- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)
