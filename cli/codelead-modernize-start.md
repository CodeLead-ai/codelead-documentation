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
codelead modernize start [--base <URL>] [--unattended] [--project <NAME>] [--as <ROLE>=<ACCOUNT>]...
codelead modernize start -h|--help
```

## Description

`start` does three things:

1. It writes `.codelead/modernization.json`, the state file, with the eight stages and the default answers. When a state file is already there, `start` keeps its stages and its answers.
2. It runs the `read` stage. The stage writes the [dossier](../modernization/dossier.md) under `docs/codelead/dossier/`: the survey of the application, its dependencies, its security exposure and the options to modernize it. No model is called.
3. It moves to the `recordings` stage, which waits for a person.

Run [`codelead init`](codelead-init.md) in the folder first. In a folder where CodeLead is not initialized, `start` is refused before any work, with exit code 2: `Refused: There is no CodeLead configuration in this folder (.codelead/config.json): initialize the folder first (codelead init, or Initialize in the extension), then start the program. Nothing was written or run.`

The program is in CodeLead's Pro plan. In a folder without the Pro plan, `start` is refused before any work, with exit code 2, and nothing is written. See [Licence and plans](../concepts/licence-and-plans.md).

> [!NOTE]
> **New to this?** The dossier is CodeLead's report on the old application. It says what the application is made of, what is risky, and which ways to modernize it fit. Read it before you decide anything.

### The Explorer's crawl

With `--base <url>`, the `read` stage also runs the [Explorer](../analyst/explore-an-application.md) on the running application. The Explorer clicks every link and menu item that sends no form, and records each screen.

When the Explorer explores nothing, the `read` stage waits. This happens when the application refuses every sign-in, or when no role reaches a screen. The dossier is written and kept. The problem names each role with what the Explorer saw. Its ways out:

- Open the address in a browser, and try the sign-in with the same account and password. Check each account and its password variable. Then run the stage again.
- Or go on without the crawl: `codelead modernize skip read --reason "<WHY>"`, then `codelead modernize next`.

When one role is refused and another one explores, the stage is done. Its summary ends with a note that names the refused role.

### Sign-in accounts for the crawl

With `--as <role>=<account>`, the Explorer signs in as that account, and explores what the role can reach. Repeat `--as` for each role. The program keeps the roles and the accounts in the state file, as `application.accounts`, for every later run of the `read` stage. It never keeps a password.

The `read` stage reads each password when it runs: from `CODELEAD_WALK_PASSWORD_<ROLE>`, with the role in capitals, else from the file that `CODELEAD_WALK_CREDENTIALS` names. That file must be outside the folder. When a password is missing, the stage waits, and its problem names the variable. Nothing is started or crawled. Set the variable, then run the stage again with [`codelead modernize run read`](codelead-modernize-run.md).

A role's name starts with a letter, then holds letters, digits or underscores. A wrong `--as` is refused before any work, with exit code 2, and nothing is recorded.

When a run profile is recorded with [`codelead modernize source`](codelead-modernize-source.md), CodeLead starts the old application for the crawl and stops it after. Its address is then the default for `--base`.

### The default answers

`start` records two answers for later stages: the rebuild folder (`../<folder>-rebuild`) and the template (`dotnet-next-shadcn`). Change them with [`codelead modernize answer`](codelead-modernize-answer.md).

> [!TIP]
> **For experienced users:** the survey runs `scripts/dossier/assembleDossier.ts` with tsx, and takes the intake's defaults when there is no terminal. The stage is done when the survey completes and derives the options. With an address, the Explorer must also explore a screen as one of its roles.

## Options

- **`--base <URL>`**

  The running application's address. The `read` stage's Explorer crawls it.

- **`--unattended`**

  Unattended mode: CodeLead gives each question its recommended answer. For example, a patch that passed the patch check with no blocking finding gets yes. The run stops, with the reason, at a step that is hard to undo or at a question with no recommended answer. A remote model's consent is asked at the start. The default is approve mode: you approve each landing, as the transforms ask. See [Approve mode and unattended mode](../modernization/the-program.md#approve-mode-and-unattended-mode).

- **`--project <NAME>`**

  The web project to modernize, when the folder holds several.

- **`--as <ROLE>=<ACCOUNT>`**

  A role that the `read` stage's Explorer signs in as, and its account. Repeat it for each role. The state file keeps it for every run of the stage. The password is read when the stage runs, from `CODELEAD_WALK_PASSWORD_<ROLE>`, else from the file that `CODELEAD_WALK_CREDENTIALS` names. It is never written. See [Sign-in accounts for the crawl](#sign-in-accounts-for-the-crawl).

## Examples

- Start the program on the Basic Store sample, with no address:

  ```console
  codelead modernize start
  ```

  <!-- described, not captured -->
  ```output
  The modernization program of webapp (aspnet-mvc), approve mode: started at .codelead/modernization.json; no address (the Explorer does not crawl).
  ▸ Read the application (codelead modernize run read)
  Surveying <your project> into docs/codelead/dossier (read-only; D1–D15)...
    opportunitiesReport.ts: D11 report: <your project>/docs/codelead/dossier/opportunities.{json,md} — 8 opportunities
    securityExposureReport.ts: D15 report: <your project>/docs/codelead/dossier/security-exposure.{json,md} — 11 finding(s): 2 proven, 9 suspected, over 75 source and 2 configuration files (1 note(s))
  …
  Dossier bundle: docs/codelead/dossier/ (34 files)
  Survey: complete. Intake: defaults (all defaults).
  Options: 4 of 12 fit (T1, T2, T5, T8); recommended sequence: T5 → T1 → T2 → T8.
  …
    Done: The dossier: 34 files; the options' sequence T5 → T1 → T2 → T8; no address, so no crawl (start with --base <url> for the Explorer).
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

- Start the program, and let the Explorer sign in as a clerk. Set the password in the environment first:

  ```console
  export CODELEAD_WALK_PASSWORD_CLERK=<the password>
  codelead modernize start --base http://localhost:5080/ --as clerk=clerk@example.test
  ```

  <!-- described, not captured -->
  ```output
  The modernization program of webapp (aspnet-mvc), approve mode: started at .codelead/modernization.json; …
  The Explorer signs in as clerk (clerk@example.test), kept in .codelead/modernization.json for every run of the read stage; each password is read when the stage runs, from CODELEAD_WALK_PASSWORD_CLERK, else the file CODELEAD_WALK_CREDENTIALS names, and is never written.
  ▸ Read the application (codelead modernize run read)
  …
  The Explorer: crawling http://localhost:5080/ as clerk (clerk@example.test) (no forms sent) into …
  ```

  Without the password, the stage waits with the problem `The Explorer needs the password of an account`, and names `CODELEAD_WALK_PASSWORD_CLERK`. When the application refuses the account or its password, the stage waits with the problem `The Explorer's sign-in as clerk was not accepted`.

- Start the program in a folder without the Pro plan. The start is refused before any work:

  ```console
  codelead modernize start
  ```

  ```output
  The modernization program is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.
  ```

  The exit code is 2. Nothing is written.

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
| `0` | The `read` stage is done, and the program waits at the recordings. Or the `read` stage waits for you: a password, the old application, or an Explorer that explored nothing. |
| `1` | The `read` stage failed. Its problem says why, and what to do. |
| `2` | The command was refused before any work: a folder without the Pro plan, a folder where CodeLead is not initialized, another process that runs a program here, a state file that cannot be read, or an `--as` that is not valid. |

## Files

| File | What `start` does with it |
|---|---|
| `.codelead/modernization.json` | Writes the state file, or keeps the one that is there. With `--as`, writes the roles and the accounts in `application.accounts`. |
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
