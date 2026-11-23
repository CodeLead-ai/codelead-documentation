---
title: codelead modernize
description: Run the modernization program of an old application, one stage after another, with its state kept in .codelead/modernization.json.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize

**Applies to:** CodeLead 0.1 and later

`codelead modernize` runs the **modernization program**: the eight stages that take an old application to a new one, one stage after another. Run it in the old application's folder. The program keeps its state in `.codelead/modernization.json`.

## Name

`codelead modernize` - The modernization program, stage by stage.

## Synopsis

```console
codelead modernize start [--base <URL>] [--unattended] [--project <NAME>] [--as <ROLE>=<ACCOUNT>]...
codelead modernize next [--no-install]
codelead modernize run <STAGE> [--no-install]
codelead modernize continue [--no-install]
codelead modernize answer <KEY=VALUE>...
codelead modernize skip <STAGE> --reason <WHY>...
codelead modernize status [--json]
codelead modernize stop
codelead modernize source [<OPTIONS>]
codelead modernize -h|--help
```

## Description

To modernize an application is to move it to a current platform without losing what it does. CodeLead does it in eight stages. Each stage reads what the stage before it wrote. See [The modernization program](../modernization/the-program.md) and [The eight stages](../modernization/the-eight-stages.md).

| # | Stage id | Title | What it does | Model calls |
|---|---|---|---|---|
| 1 | `read` | Read the application | Writes the [dossier](../modernization/dossier.md). With an address, the Explorer crawls the running application. The stage waits when the Explorer explores nothing. | None |
| 2 | `recordings` | Record the journeys | Waits for a person, who records the journeys with the [Analyst](../analyst/record-a-journey.md). | None |
| 3 | `pins` | Pin the contracts | Writes the [pins](../modernization/pins.md): tests that record what the code does today. | Yes |
| 4 | `in-place` | Transform in place | Runs the [transforms](../modernization/transforms.md) that fit, in the old code. | Yes |
| 5 | `model` | Extract the model | Runs [`codelead model`](codelead-model.md), then writes the feature map. | None |
| 6 | `rebuild-init` | Initialize the rebuild | Runs [`codelead rebuild init`](codelead-rebuild-init.md). | None |
| 7 | `rebuild` | Build and verify | Runs [`codelead rebuild run`](codelead-rebuild-run.md), then the walk, parity and the report. | Yes |
| 8 | `audit` | Write the audit pack | Writes the [audit pack](../modernization/audit-pack.md). | None |

> [!NOTE]
> **New to this?** You do not need to run the eight stages by hand. `start` runs the first one, and `next` and `continue` run the others until one waits for you. The [Modernize wizard](../extension/modernize-wizard.md) in Visual Studio Code runs these same commands with buttons.

### The subcommands

| Subcommand | What it does |
|---|---|
| [`start`](codelead-modernize-start.md) | Writes the state file and runs the `read` stage. Then the program waits at the recordings. |
| [`next`](codelead-modernize-next.md) | Runs the next stage that is not done, and the ones after it, until one waits for you, one fails, or all are done. |
| [`run`](codelead-modernize-run.md) | Runs one stage, again if it ran before. |
| [`continue`](codelead-modernize-continue.md) | Marks the recordings as done, then runs the next stages. |
| [`answer`](codelead-modernize-answer.md) | Records the answers that the stages need. Runs nothing. |
| [`skip`](codelead-modernize-skip.md) | Goes past a stage that failed or waits, with your reason. Runs nothing. |
| [`status`](codelead-modernize-status.md) | Prints each stage's status and problem, the answers, and what comes next. |
| [`stop`](codelead-modernize-stop.md) | Stops the stage that is running. No further stage runs. |
| [`source`](codelead-modernize-source.md) | Records how CodeLead starts the old application for the crawl. |

### The overall flow

1. Optional: record how to start the old application, with [`source`](codelead-modernize-source.md).
2. Run [`start`](codelead-modernize-start.md). The `read` stage writes the dossier.
3. Record the journeys with the Analyst. Then run [`continue`](codelead-modernize-continue.md).
4. When a stage waits for an answer, give it with [`answer`](codelead-modernize-answer.md), then run [`next`](codelead-modernize-next.md).
5. When a stage fails, read its problem in [`status`](codelead-modernize-status.md). Fix the cause, then run the stage again with [`run`](codelead-modernize-run.md). When the cause cannot be fixed now, go past the stage with [`skip`](codelead-modernize-skip.md) and your reason.

### Approve mode and unattended mode

In **approve** mode (the default), you approve each landing of a transform, as the transforms ask. The Pin stage also asks `Narrate webapp now? [Y/n]` when the web project has no narration. With `start --unattended`, CodeLead gives each question its recommended answer. For example, a patch that passed the patch check with no blocking finding gets yes. The Pin stage narrates without asking. The run stops, with the reason, at a step that is hard to undo or at a question with no recommended answer. Nobody is there to answer them.

### One program at a time

One program runs at a time in a folder. A run takes the old application's run lock, `.codelead/run.lock`, and gives it back at its end. A second run is refused while the lock is held. When a stage shows "running" and nobody holds the lock, the process that ran it has ended: the stage is marked failed, with that reason.

> [!TIP]
> **For experienced users:** every stage that ends writes an evidence record (`modernize · <stage>`, with the outcome success, failure or asked). The state file is `codelead.modernization/1`, written after every change of a stage's status. See [Modernization state file](../reference/modernization-state-file.md).

> [!NOTE]
> **For team leads:** the [audit pack](../modernization/audit-pack.md) at the end gathers every stage's evidence for a review.

Every subcommand except `stop` needs the Pro plan in the folder. On the free plan, the subcommand is refused before anything runs, with exit code 2. Every `--help`, and `codelead modernize stop`, stay free: a person can always stop what runs on the machine. See [Licence and plans](../concepts/licence-and-plans.md).

## Examples

- Start the program on the Basic Store sample:

  ```console
  codelead modernize start
  ```

  <!-- described, not captured -->
  ```output
  The modernization program of webapp (aspnet-mvc), approve mode: started at .codelead/modernization.json; no address (the Explorer does not crawl).
  ▸ Read the application (codelead modernize run read)
  …
    Done: The dossier: 34 files; the options' sequence T5 → T1 → T2 → T8; no address, so no crawl (start with --base <url> for the Explorer).
  ▸ Record the journeys (codelead modernize run recordings)
    Waiting for you: The recordings are a person's: 0 recordings drafted so far; continue when they are made.
  …
    Then: codelead modernize continue
  ```

- See where the program stands:

  ```console
  codelead modernize status
  ```

  ```output
  The modernization program of webapp (aspnet-mvc), approve mode; updated 2026-11-20T04:20:59.164Z.
    1. Read the application: done (04:20Z–04:20Z)
    2. Record the journeys: waiting (04:20Z–04:20Z)  ← the program is here
  …
    3. Pin the contracts: pending
  …
  Next: codelead modernize continue
  ```

- Continue after the recordings. On a Mac, the pins stage fails, and its problem says the way out:

  ```console
  codelead modernize continue
  ```

  ```output
  Record the journeys: done (0 recordings drafted).
  ▸ Pin the contracts (codelead modernize run pins)
    Failed: The pins of webapp need Visual Studio's MSBuild, and it is not on this machine.
  …
    Then: codelead modernize run pins
  ```

## Exit codes

The subcommands share these codes.

| Code | Meaning |
|---|---|
| `0` | The run ended with every stage done, or at a stage that waits for a person. Your turn is not a failure. |
| `1` | A stage failed. For `status`: there is no program in this folder. |
| `2` | The command was refused before anything ran: a word that it does not take, a wrong option, a required option that is missing (such as `skip` without `--reason`), a program that runs in another process, a state file that cannot be read, a folder without the Pro plan. |

## Files

| File | What the program does with it |
|---|---|
| `.codelead/modernization.json` | The state file: each stage's status, outputs, evidence and problem, and the answers. |
| `.codelead/modernization.stop` | The program's stop file, written by `stop`. |
| `.codelead/run.lock` | The run lock, held while a stage runs. |
| `.codelead/config.json` | `source` writes the run profile under `modernization.source`. |
| `docs/codelead/dossier/` | The `read` stage writes the dossier here. |
| `docs/codelead/rebuild/` | The `model` stage writes the System Model and the feature map here. |
| `.codelead/evidence/ev-*.json` | One evidence record for each stage that ends. |

The rebuild itself is made in another folder, `../<folder>-rebuild` by default.

## See also

- [The modernization program](../modernization/the-program.md)
- [The eight stages](../modernization/the-eight-stages.md)
- [Modernization state file](../reference/modernization-state-file.md)
- [The Modernize wizard](../extension/modernize-wizard.md)
- [`/modernize`](../chat-commands/modernize.md): the same program in a session, with `--run`, `--continue`, `--answer` and `--skip`.
- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)
