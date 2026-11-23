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

Each increment has at most 3 attempts. An increment that still fails is recorded as failed, and CodeLead puts back what it changed: `Increment inc-3 working tree restored to the last checkpoint.` Then the run goes on with the next one. The run ends with a count of the increments that were built, verified and checkpointed.

The restore touches only what the failed increment touched:

- Git restores the files that the increment's patches changed.
- CodeLead removes the new files that those patches created.
- No other file is touched. An edit that you made to another file during the run stays.
- A file that both the increment and you changed goes back to the last checkpoint, with your edit in it.
- CodeLead's own files stay as they are: `.codelead/`, `.codeleadsessions/` and `docs/codelead/`.

An increment that applied nothing says `Increment inc-3 had applied nothing; nothing to restore.` When the skeleton fails, the run stops, and nothing is put back.

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

Inside each increment, CodeLead answers the questions of its steps by their recommended answers. A step that is hard to undo is never answered: the build stops there, with the outcome `stopped`, the reason and the way out. A question with no recommended answer stops it too.

When a role uses a remote provider, CodeLead asks you for the consent after your yes to the increments, before the first model call. It asks once for each remote provider that the roles would use. A no ends the command before anything is built: `The incremental build did not start: <provider> was not allowed (the answer was no), and <role> uses it.` See [Privacy and remote models](../concepts/privacy-and-remote-models.md).

## What you see

| Situation | Headline |
|---|---|
| Every increment passed | `Incremental build complete: 4/4 increments built, verified, and checkpointed.` |
| Some increments failed | `Incremental build finished with failures: 3/4 increments built, verified, and checkpointed.`, with one `FAILED (not built)` line each, after its `working tree restored to the last checkpoint.` line, then `Next:` `Build each failed increment with /implement <the increment's objective>; the increments above are committed` |
| You answered no between increments | `Incremental build paused after 2/4 increments.`, then `Next: rerun /build-incremental to continue, or use /implement for the rest.` |
| You answered no to the plan | `Incremental build not started. Nothing was created.` |
| You answered no to the consent for a remote model | `The incremental build did not start: <provider> was not allowed (the answer was no), and <role> uses it` (outcome `stopped`) |
| The planner failed | `Failed to create the increment plan`, with the reason |

### When a probe objects

The probes of the running app are heuristics: they can be wrong. So by default, the build and the tests decide, and a probe warns. This default is the probe authority `evidence`. See [New apps](../concepts/new-apps.md#the-gates-of-an-increment).

| Situation | What you see |
|---|---|
| The acceptance probe rejected an increment twice, and the increment builds and passes its tests | `WARNING Increment inc-3 (Mark a task done): acceptance probe failed (visual_delta) — kept on its build and tests (probe authority: evidence; the probe's one retry is spent). The probe's account:`, then the account. The increment is committed. |
| The regression probe no longer finds the words or the drawing of an earlier increment | `WARNING Increment inc-3 (Mark a task done): cumulative regression: inc-2 no longer observable after inc-3 — flagged, not refused (probe authority: evidence). The probe's account:`, then the account. The increment is committed. |
| A page that was verified now renders nothing | `Increment inc-3 hard regression (a verified page is blank): <reason>. Retrying with the reason (attempt 2/3).` After the third attempt: `Increment inc-3 (Mark a task done) FAILED (not built): hard regression (a verified page is blank): <reason>.` |

An increment that is kept with a warning counts as built, verified and checkpointed in the last line. Its row in the run report reads `Verified · warnings`, with the probe's account. The blank page is the one finding that a probe still refuses.

With `greenfield.probeAuthority` set to `gate` in `.codelead/config.json`, or `CODELEAD_PROBE_AUTHORITY=gate`, every probe finding refuses the attempt. The increment fails when its 3 attempts are spent, and its changes are put back to the last checkpoint, for example: `Increment inc-3 (Mark a task done) FAILED (not built): built but could not be observed working.` See [Configuration](../reference/configuration.md#greenfield).

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
- Puts a failed increment's changes back to the last checkpoint: only the files that its patches changed or created.
- Writes `.codelead/run-journal.json` (every event of the run) and `.codelead/run-report.html` (the page built from it).
- Writes the increment ledger and the session's files under `.codeleadsessions/`.

## See also

- [`/create-app`](create-app.md)
- [`/implement`](implement.md)
- [Quickstart: build a new app](../get-started/quickstart-new-app.md)
- [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md)
- [Files and folders](../reference/files-and-folders.md)
