---
title: /build-incremental
description: Build a new app one verified increment at a time, with a checkpoint commit after each increment and a live run report.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /build-incremental

**Applies to:** CodeLead 0.1 and later

## Name

`/build-incremental` - Builds a new app one verified increment at a time.

## Synopsis

```console
/build-incremental <REQUEST>
```

## Description

`/build-incremental` plans the app as a short list of increments. An increment is a slice of the app that works on its own. The first is a `skeleton` that builds. Then each `capability` adds one feature, and a `polish` increment can end the list. You approve the list once. Then CodeLead builds each increment and verifies it: the build, probes of the running app, and the walk of its spec when there is one. A verified increment gets a checkpoint commit. CodeLead asks before it goes to the next increment.

You use `/build-incremental` for a larger app, or when you want a commit after every working slice. [`/create-app`](create-app.md) plans and builds in steps, without the commits.

Each increment has at most 3 attempts. An increment that still fails is recorded as failed, and the run goes on with the next one. The run ends with a count of the increments that were built, verified and checkpointed.

CodeLead writes a run report while it builds: `.codelead/run-report.html`. The page refreshes at every checkpoint. Warnings and failures in it carry the gate's account and the ways out.

> [!NOTE]
> **New to this?** A checkpoint commit is a git commit that CodeLead makes for you after an increment passes. When a later increment goes wrong, every earlier increment is still safe in git.

## Arguments

- **`<REQUEST>`**

  The app to build, in plain words. The request is required.

## What CodeLead asks

| Question | CodeLead recommends | Enter gives |
|---|---|---|
| `Build these increments one at a time? [Y/n]` | Yes. | Yes. |
| `Continue to the next increment? [Y/n]` | Yes. | Yes. |

Inside each increment, CodeLead answers the questions of its steps by their recommended answers.

## What you see

| Situation | Headline |
|---|---|
| Every increment passed | `Incremental build complete: 4/4 increments built, verified, and checkpointed.` |
| Some increments failed | `Incremental build finished with failures: 3/4 increments built, verified, and checkpointed.`, with one `FAILED (not built)` line each, then `Next:` `Build each failed increment with /implement <the increment's objective>; the increments above are committed` |
| You answered no between increments | `Incremental build paused after 2/4 increments.`, then `Next: rerun /build-incremental to continue, or use /implement for the rest.` |
| You answered no to the plan | `Incremental build not started. Nothing was created.` |
| The planner failed | `Failed to create the increment plan`, with the reason |

## Examples

- Build a task manager in increments:

  ```console
  /build-incremental a task manager app with a list of tasks and a done checkbox
  ```

  <!-- described, not captured -->
  ```output
  Build incremental
  Request: a task manager app with a list of tasks and a done checkbox
  Increment plan (3 increments):
  1. [skeleton] …
  2. [capability] …
  3. [capability] …
  Build these increments one at a time? [Y/n]
  Run report: <your project>/.codelead/run-report.html — refreshed at every checkpoint.
  …
  Increment <id> verified and checkpointed (committed).
  Continue to the next increment? [Y/n]
  …
  Incremental build complete: 3/3 increments built, verified, and checkpointed.
  Run the app in the target folder to exercise it.
  ```

- The model server does not run:

  ```output
  Build incremental
  Request: a task manager app with a list of tasks and a done checkbox
  Failed to create the increment plan
  Could not connect to lmstudio at http://localhost:1234/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ```

- A request is required:

  ```console
  /build-incremental
  ```

  ```output
  Missing request for /build-incremental.

  Usage:
    /build-incremental <request>
  ```

## Files

- Writes the app's files in the folder, and a checkpoint commit per verified increment.
- Writes `.codelead/run-journal.json` (every event of the run) and `.codelead/run-report.html` (the page built from it).
- Writes the increment ledger and the session's files under `.codeleadsessions/`.

## See also

- [`/create-app`](create-app.md)
- [`/implement`](implement.md)
- [Quickstart: build a new app](../get-started/quickstart-new-app.md)
- [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md)
- [Files and folders](../reference/files-and-folders.md)
