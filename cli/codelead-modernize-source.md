---
title: codelead modernize source
description: Record how CodeLead starts the old application, so that the read stage can start it for the Explorer's crawl and stop it after.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize source

**Applies to:** CodeLead 0.1 and later

`codelead modernize source` records how CodeLead starts the old application: the program to run, its arguments, and the address where it answers. The `read` stage then starts the application for the Explorer's crawl, and stops it after.

## Name

`codelead modernize source` - Records the old application's run profile.

## Synopsis

```console
codelead modernize source --command <EXE> [--args <WORDS>...] --url <URL> [--ready <PATH>]
                          [--cwd <FOLDER>] [--start-ms <MS>] [--env <NAME=VALUE>...] [--no-allow]
                          [-- <WORDS>...]
codelead modernize source --preset iis-express --url <URL> [--project <NAME>]
codelead modernize source --show
codelead modernize source --clear
codelead modernize source -h|--help
```

## Description

The **run profile** of the old application says how to start it on this machine. CodeLead keeps it under `modernization.source` in `.codelead/config.json`. See [Start the old application for CodeLead](../modernization/source-run-profile.md) and [Run profile](../reference/run-profile.md).

When the profile is recorded and allowed, the `read` stage of [`codelead modernize start`](codelead-modernize-start.md) does this:

1. It starts the application with the command.
2. It waits until the `--ready` path answers 200, for at most `--start-ms` milliseconds.
3. The [Explorer](../analyst/explore-an-application.md) crawls the application.
4. It stops the application. An application that already answered before the stage is never stopped.

**Recording the profile is your yes** to CodeLead starting the application. With `--no-allow`, the profile is kept, but CodeLead starts nothing: the `read` stage waits for you to start the application.

Run [`codelead init`](codelead-init.md) in the folder first: `source` refuses in a folder with no CodeLead configuration.

> [!NOTE]
> **New to this?** An old ASP.NET application does not start by itself. On Windows, Visual Studio starts it with IIS Express. The run profile tells CodeLead the same thing, so that the Explorer can see the application's screens.

### One executable, never a shell line

`--command` names one executable, on the `PATH` or as a path. The arguments go in `--args`, or after `--` when they start with `-`. CodeLead never runs a shell line. `--command "dotnet run"` is refused, because it holds a space.

For an SDK project:

```console
codelead modernize source --url http://localhost:5080/ --command dotnet -- run --project <web project>.csproj --urls http://localhost:5080
```

### The IIS Express preset

On Windows, `--preset iis-express` composes the profile for you: `iisexpress.exe /path:<the web project's folder> /port:<the URL's port>`. The preset runs on Windows only. On another system it is refused, with the `dotnet run` line to use instead.

> [!TIP]
> **For experienced users:** IIS Express answers 400 to `127.0.0.1`, so the preset's URL uses `localhost`. An absolute path in `--command` can hold spaces, as in `C:\Program Files (x86)\…`.

## Arguments

- **`<WORDS>...`**

  The application's arguments after `--`: the ones that start with `-`.

## Options

- **`--command <EXE>`**

  The executable that starts the application: one, on the `PATH` or as a path. Never a shell line.

- **`--args <WORDS>...`**

  Its arguments. The ones that start with `-` go after `--`.

- **`--url <URL>`**

  Where the application answers on this machine: `http://localhost:<port>/`.

- **`--ready <PATH>`**

  A path that answers 200 once the application is up. The default is `/`.

- **`--cwd <FOLDER>`**

  Where the application runs, inside this folder. The default is this folder.

- **`--start-ms <MS>`**

  How long the application may take to answer, in milliseconds. The default is 120000 (2 minutes).

- **`--env <NAME=VALUE>...`**

  Environment variables to set for the application, over CodeLead's environment.

- **`--preset <NAME>`**

  `iis-express`: the folder's web project served by IIS Express, on Windows: `iisexpress.exe /path:<its folder> /port:<the URL's port>`.

- **`--project <NAME>`**

  The web project that the preset serves, when the folder holds several.

- **`--no-allow`**

  Records the profile without your yes. CodeLead starts nothing, and the `read` stage waits for you to start the application.

- **`--show`**

  Prints the run profile.

- **`--clear`**

  Takes the run profile out of the configuration.

## Examples

- Record how to start an SDK project with `dotnet run`:

  ```console
  codelead modernize source --url http://localhost:5080/ --command dotnet -- run --project WebApplication1/webapp.csproj --urls http://localhost:5080
  ```

  ```output
  The old application's run profile is recorded in .codelead/config.json (modernization.source):
    command: dotnet run --project WebApplication1/webapp.csproj --urls http://localhost:5080
    answers at: http://localhost:5080/ (ready when / answers 200, within 120 s)
    allowed: yes, CodeLead starts it for the stages that need it running (the read stage's Explorer crawl) and stops it after
  Then: codelead modernize start (or codelead modernize run read): the read stage starts the application for the Explorer's crawl and stops it after.
  ```

- Record a profile, but start the application yourself:

  ```console
  codelead modernize source --url http://localhost:5080/ --command dotnet --no-allow -- run
  ```

  ```output
  The old application's run profile is recorded in .codelead/config.json (modernization.source):
    command: dotnet run
    answers at: http://localhost:5080/ (ready when / answers 200, within 120 s)
    allowed: no, CodeLead does not start it: start it by hand before the read stage, or record the profile again without --no-allow
  Then: codelead modernize start (or codelead modernize run read): the read stage waits for you to start the application.
  ```

- Show the profile, then take it out:

  ```console
  codelead modernize source --show
  codelead modernize source --clear
  ```

  ```output
  The old application's run profile (modernization.source in .codelead/config.json):
    command: dotnet run --project WebApplication1/webapp.csproj --urls http://localhost:5080
    answers at: http://localhost:5080/ (ready when / answers 200, within 120 s)
    allowed: yes, CodeLead starts it for the stages that need it running (the read stage's Explorer crawl) and stops it after
  The run profile is taken out of .codelead/config.json: the program starts nothing; the read stage's Explorer crawls the address the state file names (codelead modernize start without --base names none).
  ```

- Try the IIS Express preset on a Mac. It is refused, with the line to use instead:

  ```console
  codelead modernize source --preset iis-express --url http://localhost:5080/
  ```

  ```output
  Refused: IIS Express runs on Windows, and this machine is macOS: run the preset on the Windows machine the application runs on, or record another way to start it here (an SDK project: codelead modernize source --url http://localhost:5080/ --command dotnet -- run --project <web project>.csproj --urls http://localhost:5080).
  ```

- Give a shell line as the command. It is refused:

  ```console
  codelead modernize source --command "dotnet run" --url http://localhost:5080/
  ```

  ```output
  Refused: the run profile is not recorded: modernization.source.command "dotnet run" holds whitespace: it names one executable (on the PATH, or a path without spaces), and its arguments go in modernization.source.args.
  …
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The profile was recorded, shown or taken out. |
| `2` | The profile was refused: a missing `--url`, a shell line, a preset on the wrong system, a folder with no CodeLead configuration. Nothing was recorded. |

## Files

`source` reads and writes `modernization.source` in `.codelead/config.json`.

## See also

- [Start the old application for CodeLead](../modernization/source-run-profile.md)
- [Run profile](../reference/run-profile.md)
- [`codelead modernize start`](codelead-modernize-start.md): the stage that starts the application.
- [Explore an application](../analyst/explore-an-application.md)
- [Build .NET Framework projects on Windows](../how-to/build-on-windows.md)
