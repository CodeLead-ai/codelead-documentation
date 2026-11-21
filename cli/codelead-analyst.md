---
title: codelead analyst
description: The CodeLead Analyst, which records, explores and documents a running web application from its screens, and its subcommands.
kind: reference
area: cli
date: 2026-11-20
---

# codelead analyst

**Applies to:** CodeLead 0.1 and later

`codelead analyst` runs the CodeLead Analyst. The Analyst documents a running web application from its screens and the requests behind them, without its source code. It drafts journey specifications from what it saw, and replays them with CodeLead's walk. Each subcommand has its own page in the [Analyst reference](../analyst/index.md).

## Name

`codelead analyst` - The CodeLead Analyst: record, explore, draft, check, replay, rules and facts.

## Synopsis

```console
codelead analyst <SUBCOMMAND> [<OPTIONS>]
codelead analyst help [<SUBCOMMAND>]
codelead analyst
```

## Description

Every word after `analyst` is the Analyst's own, its options and `--help` included. `codelead analyst explore --base http://localhost:5173` runs `analyst explore --base http://localhost:5173`. `codelead analyst` alone prints the Analyst's help. The exit code is the Analyst's.

The Analyst works in two ways:

- **A person drives**: [`record`](../analyst/reference/analyst-record.md) opens a browser and a control window. A person uses the application, and the Analyst cuts each journey into steps. See [Record a journey](../analyst/record-a-journey.md).
- **The Analyst drives**: [`explore`](../analyst/reference/analyst-explore.md) clicks every link and menu item that sends no form, as each role, and records each new screen. No person and no model. See [Explore an application](../analyst/explore-an-application.md).

The other subcommands turn the recordings into [journeys](../reference/journeys-format.md), rules and facts. See [From a recording to a specification](../analyst/from-recording-to-spec.md).

> [!NOTE]
> **New to this?** Use the Analyst when you must document an application whose code you cannot read, or before you rebuild one. The [Analyst view](../extension/analyst-view.md) in Visual Studio Code runs the same commands with buttons.

### The pro plan

The Analyst's commands need the pro plan in the current folder. The Analyst's own help is free: `codelead analyst`, `codelead analyst --help` and `codelead analyst help <subcommand>` always work. On the free plan, `codelead analyst <subcommand> --help` is refused like the subcommand itself: use `help <subcommand>` there. See [Licence and plans](../concepts/licence-and-plans.md).

> [!TIP]
> **For experienced users:** the Analyst is the package `@codelead/analyst`, built into CodeLead. It loads only when this command runs, so that the other commands start faster. Its help, the next steps that it prints and the files that it writes name `codelead analyst`, the command that you ran.

## The subcommands

| Subcommand | What it does |
|---|---|
| [`record`](../analyst/reference/analyst-record.md) | Opens the control window and a visible browser. A person uses the application; the Analyst cuts each journey into steps and records each step. |
| [`explore`](../analyst/reference/analyst-explore.md) | Explores the running application by itself, as each role. Records one recording per role (`codelead.recording/1`). |
| [`draft`](../analyst/reference/analyst-draft.md) | Turns a recording into a journey specification (`codelead.journeys/1`), with a list of what to confirm. |
| [`check`](../analyst/reference/analyst-check.md) | Checks a journey specification against the `codelead.journeys/1` format, and prints the problems. |
| [`replay`](../analyst/reference/analyst-replay.md) | Walks a journey specification against the running application with CodeLead's walk, as a new user would. |
| [`rules`](../analyst/reference/analyst-rules.md) | States what the application was seen to enforce, from one or more recordings: `rules.observed.json` and `.md`. |
| [`facts`](../analyst/reference/analyst-facts.md) | Lists what the recordings show of the application, as facts for its dossier: `facts.observed.json` and `.md`. |

## Examples

- Print the Analyst's help. This works on every plan:

  ```console
  codelead analyst
  ```

  ```output
  Usage: codelead analyst [options] [command]

  CodeLead Analyst: documents a running web application from its screens and the
  requests behind them, and drafts CodeLead journey specifications from what it
  saw, which it replays with CodeLead's walk

  Options:
    -h, --help                       display help for command

  Commands:
    record [options]                 open the control window and a visible
                                     browser at the base address, a window per
                                     role once the role is started; …
    explore [options]                explore the running application by itself,
                                     with no person and no model: …
    draft [options] <recording>      turn a recording into a journey
                                     specification (codelead.journeys/1), …
    check <journeys>                 check a journey specification against the
                                     codelead.journeys/1 format and print the
                                     problems
  …
  ```

- Print the help of one subcommand:

  ```console
  codelead analyst help explore
  ```

  ```output
  Usage: codelead analyst explore [options]

  explore the running application by itself, with no person and no model: as each
  …
  ```

- Run a subcommand in a folder without the pro plan. The Analyst refuses, and the exit code is 1:

  ```console
  codelead analyst check journeys.json
  ```

  ```output
  The Analyst is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.
  ```

- Explore the running application, in a folder with the pro plan:

  ```console
  codelead analyst explore --base http://localhost:5080/
  ```

  <!-- described, not captured -->
  ```output
  The Analyst signs in as each role, then clicks every link, menu item and button that sends no form, breadth first from the first page. It writes one recording for each role.
  ```

  See [`codelead analyst explore`](../analyst/reference/analyst-explore.md) for its options and its real output.

## Exit codes

The exit code is the Analyst's. See each subcommand's page. `1` also means that the folder does not have the pro plan.

## Files

The Analyst writes its recordings and its observed files. See [Files the Analyst writes](../analyst/files.md).

## See also

- [Analyst and Explorer overview](../analyst/index.md)
- [Roles, journeys and steps](../analyst/concepts.md)
- [Tutorial: record journeys with the Analyst](../tutorials/record-journeys-with-the-analyst.md)
- [The Analyst view](../extension/analyst-view.md)
- [`codelead walk`](codelead-walk.md): the walk that `replay` uses.
- [`codelead licence`](codelead-licence.md): see the folder's plan.
