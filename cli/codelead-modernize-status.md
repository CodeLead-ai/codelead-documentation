---
title: codelead modernize status
description: Print the modernization program's state, each stage's status and problem, the answers and what comes next.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize status

**Applies to:** CodeLead 0.1 and later

`codelead modernize status` prints where the [modernization program](codelead-modernize.md) stands: each stage with its status and its problem, the answers, and the command to run next. It runs nothing.

## Name

`codelead modernize status` - Prints the state file.

## Synopsis

```console
codelead modernize status [--json]
codelead modernize status -h|--help
```

## Description

The program keeps its state in `.codelead/modernization.json`. `status` reads it and prints it for people:

- one line for each stage, with its status and its times;
- the arrow `← the program is here` on the current stage;
- under a stage that waits or failed, its problem: what happened, the way out, and the line to run after;
- the answers that the rebuild uses;
- `Next:`, the command to run now.

A stage has one of six statuses: `pending`, `running`, `waiting`, `done`, `failed` or `skipped`.

> [!NOTE]
> **New to this?** Run `status` whenever you come back to a modernization. The `Next:` line says what to do.

> [!TIP]
> **For experienced users:** `--json` prints the state file as it is: `schema`, `application`, `mode`, `answers`, `current`, and `stages`. Each stage has `id`, `title`, `status`, `command`, `chat`, `startedAt`, `finishedAt`, `outputs`, `evidence` and `problem`. The [Modernize wizard](../extension/modernize-wizard.md) reads the same file. See [Modernization state file](../reference/modernization-state-file.md).

## Options

- **`--json`**

  Prints the state file as it is. The default is the summary in words.

## Examples

- See where the program stands, while it waits for the recordings:

  ```console
  codelead modernize status
  ```

  ```output
  The modernization program of webapp (aspnet-mvc), approve mode; updated 2026-11-20T04:20:59.164Z.
    1. Read the application: done (04:20Z–04:20Z)
    2. Record the journeys: waiting (04:20Z–04:20Z)  ← the program is here
         Record the journeys: The Analyst's recordings are a person's: someone who knows the application walks its journeys while the Analyst records them.
         The way out: Record each journey with the Analyst view's "Record a journey", or run codelead analyst record --base <the application's address>; then continue. Then: codelead modernize continue
    3. Pin the contracts: pending
    4. Transform in place: pending
    5. Extract the model: pending
    6. Initialize the rebuild: pending
    7. Build and verify: pending
    8. Write the audit pack: pending
  Answers: rebuildFolder=../store-modernize-rebuild, template=dotnet-next-shadcn, framework=(none), identity=(none), database=(none).
  Next: codelead modernize continue
  ```

- See a failed stage and its way out:

  ```console
  codelead modernize status
  ```

  ```output
  The modernization program of webapp (aspnet-mvc), approve mode; updated 2026-11-20T04:21:52.264Z.
    1. Read the application: done (04:20Z–04:20Z)
    2. Record the journeys: done (04:20Z–04:21Z)
    3. Pin the contracts: failed (04:21Z–04:21Z)  ← the program is here
         The pins of webapp need Visual Studio's MSBuild, and it is not on this machine: …
         The way out: Run it on Windows with Visual Studio or its Build Tools (the web development build tools workload): …
    4. Transform in place: pending
    5. Extract the model: done (04:21Z–04:21Z)
    6. Initialize the rebuild: pending
    7. Build and verify: pending
    8. Write the audit pack: pending
  Answers: rebuildFolder=../store-modernize-rebuild, template=dotnet-next-shadcn, framework=(none), identity=keep, database=sample-seed.
  Next: codelead modernize run pins
  ```

  The `model` stage is done here because it was run by hand with [`codelead modernize run model`](codelead-modernize-run.md).

- Read the state file as JSON:

  ```console
  codelead modernize status --json
  ```

  ```output
  {
    "schema": "codelead.modernization/1",
    "application": {
      "root": ".",
      "webProject": "webapp",
      "kind": "aspnet-mvc",
      "baseUrl": null
    },
    "mode": "approve",
    "answers": {
      "rebuildFolder": "../store-api",
      "template": "dotnet-next-shadcn",
      "framework": "net8.0",
      "identity": "keep",
      "database": "sample-seed"
    },
    "current": "pins",
    "stages": [
      {
        "id": "read",
        "title": "Read the application",
        "status": "done",
        "command": "codelead modernize run read",
        "chat": "/modernize --run read",
  …
  ```

- Run it in a folder with no program:

  ```console
  codelead modernize status
  ```

  ```output
  There is no modernization program here (.codelead/modernization.json): start it with codelead modernize start [--base <url>] [--unattended].
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The state was printed. |
| `1` | There is no program in this folder. |
| `2` | The state file cannot be read. |

## Files

`status` reads `.codelead/modernization.json`, and writes nothing.

## See also

- [`codelead modernize`](codelead-modernize.md)
- [Modernization state file](../reference/modernization-state-file.md)
- [The Modernize wizard](../extension/modernize-wizard.md): the same state, as a page.
- [`codelead modernize run`](codelead-modernize-run.md): run the stage that `Next:` names.
- [Results and stops](../concepts/results-and-stops.md)
