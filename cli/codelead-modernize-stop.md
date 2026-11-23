---
title: codelead modernize stop
description: Stop the modernization program; the running stage ends failed and no further stage runs.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize stop

**Applies to:** CodeLead 0.1 and later

`codelead modernize stop` stops the [modernization program](codelead-modernize.md). The stage that is running ends as failed, "stopped by the person", and no further stage runs.

## Name

`codelead modernize stop` - Stops the program.

## Synopsis

```console
codelead modernize stop
codelead modernize stop -h|--help
```

## Description

Run `stop` from a second terminal, while `start`, `next`, `run` or `continue` runs in the first one. `stop` writes the program's stop file, `.codelead/modernization.stop`. The running program reads it and stops:

- The stage that is running ends **failed**, with the reason "stopped by the person".
- No further stage runs.
- A rebuild run stops after the part that is running. Its own stop file, `.codelead/STOP` in the rebuild, is written for that.
- When CodeLead started the old application for the stage, the run that started it stops it.

Nothing else is stopped. `stop` never kills a process that CodeLead did not start.

> [!NOTE]
> **New to this?** A stopped stage is marked failed so that the program does not run it again by itself. Run it again with [`codelead modernize run`](codelead-modernize-run.md) when you are ready.

> [!TIP]
> **For experienced users:** sometimes a process ends before it reads the stop file. The next run then removes that stop file, with a note. In the chat, stop a program with the Stop button.

This command is free: it needs no licence. A person can always stop what runs on the machine. See [Licence and plans](../concepts/licence-and-plans.md).

## Examples

- Stop a program while a stage runs. In the first terminal:

  ```console
  codelead modernize next
  ```

  In a second terminal:

  ```console
  codelead modernize stop
  ```

  <!-- described, not captured -->
  ```output
  Stopped: Read the application ends failed (stopped by the person); the step in flight finishes first, and no further stage is run.
  ```

  During the `rebuild` stage, one more line says that the rebuild run's stop file is written. The rebuild run stops after the part that is running.

  The first terminal then shows:

  <!-- described, not captured -->
  ```output
  Stopped by the person: no further stage is run.
  ```

- Run `stop` when nothing runs:

  ```console
  codelead modernize stop
  ```

  ```output
  Nothing is running: no stage to stop.
  ```

- See the stopped stage in the status:

  ```console
  codelead modernize status
  ```

  <!-- described, not captured -->
  ```output
  …
    1. Read the application: failed (…)  ← the program is here
         … stopped by the person …
  …
  Next: codelead modernize run read
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The stop was asked, or nothing was running. |
| `2` | The command was refused: the state file cannot be read. |

## Files

| File | What `stop` does with it |
|---|---|
| `.codelead/modernization.stop` | Writes it. The running program reads it and removes it. |
| `.codelead/STOP` in the rebuild | Written when the `rebuild` stage runs, so that the rebuild run stops after its running part. |
| `.codelead/modernization.json` | The running program marks the stopped stage failed. |

## See also

- [`codelead modernize`](codelead-modernize.md)
- [`codelead modernize run`](codelead-modernize-run.md): run the stopped stage again.
- [`codelead rebuild run`](codelead-rebuild-run.md): its own stop file.
- [Stop and resume work](../how-to/stop-and-resume.md)
