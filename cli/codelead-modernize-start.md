---
title: codelead modernize start
description: Start the modernization program, write its state file and run the read stage.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize start

**Applies to:** CodeLead 0.1 and later

`codelead modernize start` starts the [modernization program](codelead-modernize.md) in the old application's folder. It writes the state file and runs the first stage, `read`. Then the program waits for the recordings, which a person makes.

## Name

`codelead modernize start` - Writes the state file and runs the read stage.

## Synopsis

```console
codelead modernize start [--base <URL>] [--unattended] [--project <NAME>]
codelead modernize start -h|--help
```

## Description

`start` does three things:

1. It writes `.codelead/modernization.json`, the state file, with the eight stages and the default answers. When a state file is already there, `start` keeps its stages and its answers.
2. It runs the `read` stage. The stage writes the [dossier](../modernization/dossier.md) under `docs/codelead/dossier/`: the survey of the application, its dependencies, its security exposure and the options to modernize it. No model is called.
3. It moves to the `recordings` stage, which waits for a person.

Run [`codelead init`](codelead-init.md) in the folder first.

> [!NOTE]
> **New to this?** The dossier is CodeLead's report on the old application. It says what the application is made of, what is risky, and which ways to modernize it fit. Read it before you decide anything.

### The Explorer's crawl

With `--base <url>`, the `read` stage also runs the [Explorer](../analyst/explore-an-application.md) on the running application. The Explorer clicks every link and menu item that sends no form, and records each screen. The Explorer is in the pro plan. See [Licence and plans](../concepts/licence-and-plans.md).

When a run profile is recorded with [`codelead modernize source`](codelead-modernize-source.md), CodeLead starts the old application for the crawl and stops it after. Its address is then the default for `--base`.

### The default answers

`start` records two answers for later stages: the rebuild folder (`../<folder>-rebuild`) and the template (`dotnet-next-shadcn`). Change them with [`codelead modernize answer`](codelead-modernize-answer.md).

> [!TIP]
> **For experienced users:** the survey runs `scripts/dossier/assembleDossier.ts` with tsx, and takes the intake's defaults when there is no terminal. The stage is done when the survey completes and derives the options.

## Options

- **`--base <URL>`**

  The running application's address. The `read` stage's Explorer crawls it. This needs the pro plan.

- **`--unattended`**

  The transforms' approvals are answered yes. The default is approve mode: you approve each landing, as the transforms ask.

- **`--project <NAME>`**

  The web project to modernize, when the folder holds several.

## Examples

- Start the program on the Basic Store sample, with no address:

  ```console
  codelead modernize start
  ```

  ```output
  The modernization program of webapp (aspnet-mvc), approve mode: started at .codelead/modernization.json; no address (the Explorer does not crawl).
  ▸ Read the application (codelead modernize run read)
  Surveying <your project> into docs/codelead/dossier (read-only; D1–D15)...
    opportunitiesReport.ts: D11 report: <your project>/docs/codelead/dossier/opportunities.{json,md} — 8 opportunities
    securityExposureReport.ts: D15 report: <your project>/docs/codelead/dossier/security-exposure.{json,md} — 11 finding(s): 2 proven, 9 suspected, over 75 source and 2 configuration files (1 note(s))
  …
  Dossier bundle: docs/codelead/dossier/ (34 files)
  Survey: complete. Intake: defaults (all defaults).
  Options: 5 of 12 fit (T1, T2, T5, T8, T10); recommended sequence: T5 → T10 → T1 → T2 → T8.
  …
    Done: The dossier: 34 files; the options' sequence T5 → T10 → T1 → T2 → T8; no address, so no crawl (start with --base <url> for the Explorer).
  ▸ Record the journeys (codelead modernize run recordings)
    Waiting for you: The recordings are a person's: 0 recordings drafted so far; continue when they are made.
    Record the journeys: The Analyst's recordings are a person's: someone who knows the application walks its journeys while the Analyst records them.
    The way out: Record each journey with the Analyst view's "Record a journey", or run codelead analyst record --base <the application's address>; then continue.
    Then: codelead modernize continue
  ```

- Start the program and let the Explorer crawl the running application:

  ```console
  codelead modernize start --base http://localhost:5080/
  ```

  <!-- described, not captured -->
  ```output
  The modernization program of webapp (aspnet-mvc), approve mode: started at .codelead/modernization.json; …
  ▸ Read the application (codelead modernize run read)
  …
    Done: The dossier: 34 files; the options' sequence …; the Explorer crawled http://localhost:5080/ into …
  ▸ Record the journeys (codelead modernize run recordings)
    Waiting for you: …
  ```

  In a folder without the pro plan, the crawl is refused. The dossier is still written, and the stage says so.

- Start the program in unattended mode, for a folder with two web projects:

  ```console
  codelead modernize start --unattended --project webapp
  ```

  <!-- described, not captured -->
  ```output
  The modernization program of webapp (aspnet-mvc), unattended mode: started at .codelead/modernization.json; …
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The `read` stage is done, and the program waits at the recordings. |
| `1` | The `read` stage failed. Its problem says why, and what to do. |
| `2` | The command was refused: another process runs a program here, or the state file cannot be read. |

## Files

| File | What `start` does with it |
|---|---|
| `.codelead/modernization.json` | Writes the state file, or keeps the one that is there. |
| `docs/codelead/dossier/` | The `read` stage writes the dossier. |
| `.codelead/config.json` | Reads the run profile, `modernization.source`. |
| `.codelead/evidence/ev-*.json` | One record for the `read` stage, and one for the waiting recordings. |

## See also

- [`codelead modernize`](codelead-modernize.md): the program and its stages.
- [`codelead modernize continue`](codelead-modernize-continue.md): the next command, after the recordings.
- [`codelead modernize source`](codelead-modernize-source.md): let CodeLead start the old application.
- [The dossier](../modernization/dossier.md)
- [Record a journey](../analyst/record-a-journey.md)
- [The Modernize wizard](../extension/modernize-wizard.md)
