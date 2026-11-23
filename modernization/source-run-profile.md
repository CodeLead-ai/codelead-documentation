---
title: Start the old application for CodeLead
description: Record how CodeLead starts the old application, so that the Read stage can start it for the Explorer and stop it after.
kind: how-to
area: modernization
date: 2026-11-20
---

# Start the old application for CodeLead

This guide records the old application's **run profile**: how CodeLead starts the application on this machine, and where it answers. With the profile, the Read stage starts the application for the Explorer, waits until it answers, and stops it when the Explorer is done. Without it, you start the application yourself before the Read stage.

**In short**
- You record a run profile with the IIS Express preset on Windows, or with your own command.
- You know that recording the profile is your yes to CodeLead starting the application.
- You check the profile, and see what the Read stage does when the application does not answer.

## Before you begin

- Initialize CodeLead in the old application's folder: [`codelead init`](../cli/codelead-init.md), or **Initialize** in the extension. The profile is refused in a folder that has no CodeLead configuration.
- **Build the application first**. IIS Express serves what the build put in `bin`. On Windows, build the solution once in Visual Studio.
- Know the address where the application answers on this machine, for example `http://localhost:5080/`.

> [!NOTE]
> **New to this?** An old ASP.NET application does not start by itself. On Windows, Visual Studio starts it with IIS Express, a small web server. The run profile tells CodeLead how to do the same.

## Record the profile with the IIS Express preset

Use the preset for a .NET Framework web project on Windows.

1. Open a terminal in the old application's folder.
2. Run the preset with the address that you want:

   ```console
   codelead modernize source --preset iis-express --url http://localhost:5080/
   ```

   CodeLead composes the profile: `iisexpress.exe /path:<the web project's folder> /port:5080`, with IIS Express from the Program Files folder. When the folder holds more than one web project, name it with `--project <NAME>`.

> [!IMPORTANT]
> Use `localhost` in the address, not `127.0.0.1`. IIS Express answers `400 Invalid Hostname` to `127.0.0.1`, so the preset takes `localhost` only.

The preset runs on Windows only. On another system it is refused, with the line to use instead:

```output
Refused: IIS Express runs on Windows, and this machine is macOS: run the preset on the Windows machine the application runs on, or record another way to start it here (an SDK project: codelead modernize source --url http://localhost:5080/ --command dotnet -- run --project <web project>.csproj --urls http://localhost:5080).
```

## Record a profile with your own command

Use your own command for an SDK-style project, or for any other way to start the application. Name one executable with `--command`. Put the arguments that start with `-` after `--`:

```console
codelead modernize source --url http://localhost:5080/ --command dotnet -- run --project WebApplication1/webapp.csproj --urls http://localhost:5080
```

<!-- described, not captured -->
```output
The old application's run profile is recorded in .codelead/config.json (modernization.source):
  command: dotnet run --project WebApplication1/webapp.csproj --urls http://localhost:5080
  answers at: http://localhost:5080/ (ready when / answers with a page, a redirect or a sign-in challenge (HTTP 200 to 399, 401 or 403), within 120 s)
  allowed: yes, CodeLead starts it for the stages that need it running (the read stage's Explorer crawl) and stops it after
Then: codelead modernize start (or codelead modernize run read): the read stage starts the application for the Explorer's crawl and stops it after.
```

CodeLead never runs a shell line. A command with a space in it is refused:

```output
Refused: the run profile is not recorded: modernization.source.command "dotnet run" holds whitespace: it names one executable (on the PATH, or a path without spaces), and its arguments go in modernization.source.args.
```

> [!TIP]
> **For experienced users:** `--ready <PATH>` names the page that must answer before the application counts as ready (the default is `/`). It counts as ready on HTTP 200 to 399, 401 or 403: a page, a redirect or a sign-in challenge. A 5xx is not ready. `--start-ms <MS>` is how long to wait (the default is 120000). `--cwd <FOLDER>` and `--env NAME=value` set where and with what it runs.

## Record the profile without your yes

Recording a profile is your yes to CodeLead starting the application. To keep the profile and start the application yourself, add `--no-allow`:

```console
codelead modernize source --url http://localhost:5080/ --command dotnet --no-allow -- run
```

<!-- described, not captured -->
```output
The old application's run profile is recorded in .codelead/config.json (modernization.source):
  command: dotnet run
  answers at: http://localhost:5080/ (ready when / answers with a page, a redirect or a sign-in challenge (HTTP 200 to 399, 401 or 403), within 120 s)
  allowed: no, CodeLead does not start it: start it by hand before the read stage, or record the profile again without --no-allow
Then: codelead modernize start (or codelead modernize run read): the read stage waits for you to start the application.
```

## Check that it worked

1. Print the profile:

   ```console
   codelead modernize source --show
   ```

2. Run the Read stage. Start the program with [`codelead modernize start`](../cli/codelead-modernize-start.md), or run the stage again with `codelead modernize run read`. Without `--base`, the program takes the profile's address.

When it works, the stage's line says that CodeLead started the application and stopped it:

<!-- described, not captured -->
```output
  Done: The dossier: 34 files; the options' sequence T5 → T10 → T1 → T2 → T8; the Explorer crawled http://localhost:5080/ into analyst-recordings/<time> (the old application started from its run profile, and stopped after).
```

When the application does not answer within the time, or when the profile has no yes, the stage waits. Its problem is titled **Start the old application**, and it says what CodeLead tried and the way out. Start the application by hand, or fix the profile, then run `codelead modernize run read`.

When the address already answers before the stage, CodeLead uses the application as it is, and never stops it.

## Take the profile out

```console
codelead modernize source --clear
```

```output
The run profile is taken out of .codelead/config.json: the program starts nothing; the read stage's Explorer crawls the address the state file names (codelead modernize start without --base names none).
```

## Details

- **Where**: `modernization.source` in the old application's `.codelead/config.json`.
- **What CodeLead does with the process**: it starts the application in its own process group, and records it in `.codelead/acceptance/servers.json`. The application's output goes to `.codelead/acceptance/servers/source.log`. It stops it when the Explorer is done, and again at the end of the stage, whatever happened.
- **Stop**: `codelead modernize stop` stops a running stage, and the application that CodeLead started for it, within half a second.
- **A profile with a problem** is refused as a whole, with every problem in words. A bad profile never breaks other commands.

## See also

- [`codelead modernize source`](../cli/codelead-modernize-source.md)
- [`/modernize --source`](../chat-commands/modernize.md#--source)
- [Run profile](../reference/run-profile.md)
- [The dossier](dossier.md): the Read stage and the Explorer
- [Explore an application](../analyst/explore-an-application.md)
- [Build .NET Framework projects on Windows](../how-to/build-on-windows.md)
