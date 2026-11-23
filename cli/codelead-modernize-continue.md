---
title: codelead modernize continue
description: Mark the recordings of the modernization program as made, then run the next stages.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize continue

**Applies to:** CodeLead 0.1 and later

`codelead modernize continue` tells the [modernization program](codelead-modernize.md) that the recordings are made. It marks the `recordings` stage as done, then runs the next stages, as [`codelead modernize next`](codelead-modernize-next.md) does.

## Name

`codelead modernize continue` - Marks the recordings as done, then runs the next stages.

## Synopsis

```console
codelead modernize continue [--no-install]
codelead modernize continue -h|--help
```

## Description

The second stage, **Record the journeys**, is a person's work. Someone who knows the application walks its journeys while the [Analyst](../analyst/record-a-journey.md) records them. CodeLead cannot know when that work is finished, so the stage waits. Run `continue` when you have made the recordings.

`continue` counts the recordings that were drafted into journeys. It also continues with zero recordings. The rebuild then has only the journeys that the model drafted from the code.

> [!NOTE]
> **New to this?** A recording is better evidence than code: it shows what a real person does in the application. The more journeys you record, the more the walk of the new application can check.

> [!TIP]
> **For experienced users:** `continue` is the only command that marks `recordings` as done. `next` never runs it. In the chat, the same command is `/modernize --continue`.

The command is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, it is refused before anything runs, with exit code 2. See [Licence and plans](../concepts/licence-and-plans.md).

## Options

- **`--no-install`**

  The rebuild's initialization does not run `npm install`.

## Examples

- Continue after the recordings, on the Basic Store sample on a Mac. The `pins` stage then fails, because the sample needs Visual Studio's MSBuild:

  ```console
  codelead modernize continue --no-install
  ```

  ```output
  Record the journeys: done (0 recordings drafted).
  ▸ Pin the contracts (codelead modernize run pins)
    Failed: The pins of webapp need Visual Studio's MSBuild, and it is not on this machine.
    The pins of webapp need Visual Studio's MSBuild, and it is not on this machine: webapp is a .NET Framework web project that the .NET SDK cannot build (webapp: non-SDK, v4.5, imports Microsoft.WebApplication.targets). MSBuild was not found (looked: msbuild on the PATH (Mono's; Visual Studio's MSBuild runs on Windows only): not found). No built assembly of webapp to reference in its place (looked for WebApplication1/bin/WebApplication1.dll). Nothing was asked of the model: the gate's dotnet build of the pins could only fail with MSB4019.
    The way out: Run it on Windows with Visual Studio or its Build Tools (the web development build tools workload): …; Or copy the output of a Windows build of webapp to WebApplication1/bin/: the pins then reference WebApplication1.dll in place of the project; Or declare this scope's commands in .codelead/validation.json: …
    Then: codelead modernize run pins
  ```

  See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).

- Continue on Windows, where the pins can build. The `pins` stage then calls the model, and you approve each landing:

  ```console
  codelead modernize continue
  ```

  <!-- described, not captured -->
  ```output
  Record the journeys: done (3 recordings drafted).
  ▸ Pin the contracts (codelead modernize run pins)
  …
  Apply this patch? …
  …
    Done: …
  ▸ Transform in place (codelead modernize run in-place)
  …
  ```

- Continue in the chat, in a session:

  ```console
  codelead> /modernize --continue
  ```

  <!-- described, not captured -->
  ```output
  Record the journeys: done (…).
  ▸ Pin the contracts (codelead modernize run pins)
  …
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The run ended at a stage that waits for you, or every stage is done. |
| `1` | A stage failed. |
| `2` | The command was refused: a folder without the Pro plan, another process runs a program here, or the state file cannot be read. |

## Files

`continue` reads and writes `.codelead/modernization.json`. It reads the Analyst's recordings. Each stage that runs writes its own files. See [`codelead modernize`](codelead-modernize.md#files).

## See also

- [`codelead modernize start`](codelead-modernize-start.md): the command before it.
- [`codelead modernize next`](codelead-modernize-next.md)
- [Record a journey](../analyst/record-a-journey.md)
- [Tutorial: record journeys with the Analyst](../tutorials/record-journeys-with-the-analyst.md)
- [Pins](../modernization/pins.md): the stage after the recordings.
