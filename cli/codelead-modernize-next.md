---
title: codelead modernize next
description: Run the next stage of the modernization program that is not done, and the stages after it, until one waits, one fails or all are done.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize next

**Applies to:** CodeLead 0.1 and later

`codelead modernize next` moves the [modernization program](codelead-modernize.md) forward. It runs the next stage that is not done, then the stages after it. It stops when a stage waits for you, when a stage fails, or when every stage is done.

## Name

`codelead modernize next` - Runs the next stages until one waits, one fails, or all are done.

## Synopsis

```console
codelead modernize next [--no-install]
codelead modernize next -h|--help
```

## Description

Use `next` after you gave an answer with [`codelead modernize answer`](codelead-modernize-answer.md), or after you fixed what a waiting stage asked for.

`next` follows these rules:

- It runs a **waiting** stage again, because a person can do what it waited for at any time.
- It never runs the `recordings` stage. The recordings are a person's work: mark them done with [`codelead modernize continue`](codelead-modernize-continue.md).
- It stops at a waiting `pins` stage, so that the transforms do not run on too few pins. Run the stage again with `codelead modernize run pins`, or accept the pins with `codelead modernize answer pins=enough`.
- It never runs a **failed** stage again. Fix the cause, then run the stage with [`codelead modernize run`](codelead-modernize-run.md). Or go past it with [`codelead modernize skip`](codelead-modernize-skip.md) and your reason.

> [!NOTE]
> **New to this?** "Waiting" means that the program needs something from a person: recordings, an answer, a decision. The waiting stage's problem says what, and the command to run after.

> [!TIP]
> **For experienced users:** `--no-install` passes to the `rebuild-init` stage, which then does not run `npm install` in the new rebuild. Run it yourself before the `rebuild` stage.

## Options

- **`--no-install`**

  The rebuild's initialization does not run `npm install`.

## Examples

- Run `next` while the program waits at the recordings. The recordings are not run, and the program says what to do:

  ```console
  codelead modernize next
  ```

  ```output
  Record the journeys: waiting for you.
    Record the journeys: The Analyst's recordings are a person's: someone who knows the application walks its journeys while the Analyst records them.
    The way out: Record each journey with the Analyst view's "Record a journey", or run codelead analyst record --base <the application's address>; then continue.
    Then: codelead modernize continue
  ```

- Give the answers that the rebuild needs, then move on without `npm install`:

  ```console
  codelead modernize answer identity=keep database=sample-seed
  codelead modernize next --no-install
  ```

  <!-- described, not captured -->
  ```output
  Answers recorded: identity=keep, database=sample-seed.
  ▸ Initialize the rebuild (codelead modernize run rebuild-init)
  Rebuild of webapp (.) from the template dotnet-next-shadcn …, in ../basic-store-rebuild.
  …
  ```

- Run `next` when every stage is done:

  <!-- described, not captured -->
  ```output
  Every stage is done: the audit pack is the program's last word.
  ```

- Run `next` while a stage has failed. `next` does not run it again. It repeats the problem, and the exit code is 1:

  ```console
  codelead modernize next
  ```

  ```output
  Pin the contracts: failed.
    The pins of webapp need Visual Studio's MSBuild, and it is not on this machine: webapp is a .NET Framework web project that the .NET SDK cannot build (webapp: non-SDK, v4.5, imports Microsoft.WebApplication.targets). …
    The way out: Run it on Windows with Visual Studio or its Build Tools (the web development build tools workload): …
    Then: codelead modernize run pins
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The run ended at a stage that waits for you, or every stage is done. |
| `1` | A stage failed. |
| `2` | The command was refused: another process runs a program here, or the state file cannot be read. |

## Files

`next` reads and writes `.codelead/modernization.json`, and each stage writes its own files. See [`codelead modernize`](codelead-modernize.md#files).

## See also

- [`codelead modernize`](codelead-modernize.md)
- [`codelead modernize status`](codelead-modernize-status.md): see which stage comes next.
- [`codelead modernize run`](codelead-modernize-run.md): run one stage again.
- [`codelead modernize continue`](codelead-modernize-continue.md): after the recordings.
- [The eight stages](../modernization/the-eight-stages.md)
