---
title: codelead modernize skip
description: Go past a stage of the modernization program that failed or waits for you, with your reason, so that the program goes on.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize skip

**Applies to:** CodeLead 0.1 and later

`codelead modernize skip` goes past a stage of the [modernization program](../modernization/the-program.md) that failed or waits for you. The stage ends `skipped`, with your reason and the time. Then [`codelead modernize next`](codelead-modernize-next.md) goes on to the stage after it. The command runs no stage itself.

## Name

`codelead modernize skip` - Goes past a stage that failed or waits, with your reason.

## Synopsis

```console
codelead modernize skip <STAGE> --reason <WHY>...
codelead modernize skip -h|--help
```

## Description

The program never runs a failed stage again by itself, and a waiting stage waits for you. Sometimes the cause cannot be fixed now. For example, the machine has no Visual Studio's MSBuild, and the transforms must wait for a Windows machine. Then you skip the stage, and say why.

A skip changes nothing in the code. It changes the stage's status in the state file, `.codelead/modernization.json`, and writes an [evidence record](../concepts/evidence-records.md):

- The stage gets the status `skipped`, and its problem is removed.
- The stage keeps three fields: `skippedBy` (`"person"`), `skipReason` (your words) and `skippedAt` (the time).
- The evidence record keeps what the stage was before the skip: its status and its problem.
- The stage keeps its earlier outputs and evidence.

[`codelead modernize status`](codelead-modernize-status.md) then shows the skip under the stage, with your reason.

> [!NOTE]
> **New to this?** A skip is a decision, not a fix. The work of the stage is not done. A later stage that needs it fails, and its problem says which stage you skipped. Run the skipped stage after all with [`codelead modernize run`](codelead-modernize-run.md) `<STAGE>`.

### What CodeLead refuses

`skip` takes only a stage that failed or waits. It refuses, and records nothing, in these cases:

| The stage | The words |
|---|---|
| `recordings` | `Record the journeys is not skipped: say the recordings are made (or that none will be): codelead modernize continue.` |
| pending | `… has not run (it is pending): a stage is skipped only when it failed or waits for you; run it first (…)` |
| running | `… is running: wait for it to end, or stop it (codelead modernize stop); a stage that failed or waits can be skipped` |
| done | `… is done: nothing to skip` |
| skipped already | `… is skipped already (by the person: <your reason>): codelead modernize next goes on` |
| no reason | `skipping <title> needs a reason: --reason <why> …` |
| a word that is no stage | `<word> is not a stage of the program: read, recordings, pins, in-place, model, rebuild-init, rebuild, audit.` |

The recordings are never skipped. When no recording will be made, say so with [`codelead modernize continue`](codelead-modernize-continue.md).

### After the skip

The command's last line names the next step. It is `next` with the next stage, or the line of a stage that already fails or waits. For example, `Then: codelead modernize continue (Record the journeys waits).`

Three stages need what an earlier stage makes:

| This stage | Needs what this stage makes |
|---|---|
| `in-place` | `read`: the options report. |
| `rebuild-init` | `model`: the System Model. |
| `rebuild` | `rebuild-init`: the rebuild folder. |

When such a stage cannot run because you skipped the stage before it, its problem says so. In CodeLead's tests, the Rebuild stage after a skipped Set up the rebuild fails with this reason and this way out:

<!-- described, not captured -->
```output
../shop-rebuild holds no rebuild (docs/codelead/rebuild/rebuild.json); Initialize the rebuild (rebuild-init), which makes it, was skipped by the person (the rebuild is made by another team)
Run Initialize the rebuild after all (codelead modernize run rebuild-init), then this stage again
```

When you run a skipped stage again, it is no longer skipped: its three skip fields go.

> [!TIP]
> **For experienced users:** the chat has the same command: `/modernize --skip <STAGE> --reason <WHY>`. See [`/modernize`](../chat-commands/modernize.md#--run---continue---answer---skip). A stage that the program skips itself, such as a stage that `CODELEAD_MODERNIZE_STUB` names, has no skip fields.

> [!NOTE]
> **For team leads:** the reason stays on the stage in the state file, and in the skip's evidence record. The [audit pack](../modernization/audit-pack.md) states every skipped stage first in its README: who, when and why.

The command is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, it is refused before anything runs, with exit code 2. See [Licence and plans](../concepts/licence-and-plans.md).

## Arguments

- **`<STAGE>`**

  The stage's id: `read`, `recordings`, `pins`, `in-place`, `model`, `rebuild-init`, `rebuild` or `audit`. See [The eight stages](../modernization/the-eight-stages.md#the-names).

## Options

- **`--reason <WHY>...`**

  Why the program goes on without the stage. It is required. The words after it are joined with spaces, so quotes are not needed. Use quotes when the reason holds a character that your shell reads, such as `;`.

- **`-h|--help`**

  Prints the command's help.

## Examples

- Go past a stage that failed. On a Mac, the Upgrade in place stage of the Basic Store sample fails at its first transform, because the gate has no pins yet:

  ```console
  codelead modernize run in-place
  ```

  <!-- described, not captured -->
  ```output
  ▸ Transform in place (codelead modernize run in-place)
  T10: not on Microsoft.AspNet.Mvc (ASP.NET MVC, webapp): the platform the application is built on is not put behind an adapter (the platform itself: T1 or the rebuild).
  T10: not on EntityFramework (Entity Framework 6, webapp): not for an adapter, an ORM the code is written in (its query types are used directly): the data layer's own transform (T5D) or the rebuild.
    Failed: /modernize --transform T5D --scope webapp was refused.
    The transform T5D was refused: Pin first: the declared gate (.codelead/validation.json) carries no characterization suite, so no transform can be shown safe. …
    Then: codelead modernize run in-place
  ```

  Skip it, with the reason:

  ```console
  codelead modernize skip in-place --reason "the transforms run on the Windows machine later"
  ```

  ```output
  Transform in place: skipped by the person (the transforms run on the Windows machine later); it was failed: The transform T5D was refused.
  Then: codelead modernize continue (Record the journeys waits).
  ```

  Here the recordings still wait, so the last line names `continue`.

- See the skip in the program's status:

  ```console
  codelead modernize status
  ```

  ```output
  The modernization program of webapp (aspnet-mvc), approve mode; updated 2026-11-20T08:22:05.561Z.
    1. Read the application: done (08:21Z–08:21Z)
    2. Record the journeys: waiting (08:21Z–08:21Z)  ← the program is here
  …
    3. Pin the contracts: pending
    4. Transform in place: skipped (08:22Z–08:22Z)
         Skipped by the person at 2026-11-20T08:22:05.561Z: the transforms run on the Windows machine later.
    5. Extract the model: pending
  …
  Next: codelead modernize continue
  ```

- Try to skip the recordings. CodeLead refuses, records nothing, and the exit code is 2:

  ```console
  codelead modernize skip recordings --reason "no recordings this week"
  ```

  ```output
  Refused: Record the journeys is not skipped: say the recordings are made (or that none will be): codelead modernize continue. Nothing was recorded.
  ```

- Try to skip a stage that has not run:

  ```console
  codelead modernize skip pins --reason "the pins come later"
  ```

  ```output
  Refused: Pin the contracts has not run (it is pending): a stage is skipped only when it failed or waits for you; run it first (codelead modernize run pins). Nothing was recorded.
  ```

- Leave out the reason. Without `--reason`, the command line itself is refused. With an empty reason, the program refuses. The exit code is 2 in both cases:

  ```console
  codelead modernize skip in-place
  codelead modernize skip in-place --reason " "
  ```

  ```output
  error: required option '--reason <text...>' not specified
  Refused: skipping Transform in place needs a reason: --reason <why> (the stage's record and the audit pack keep why the program went on without it). Nothing was recorded.
  ```

- Skip in the chat. The chat checks the line before it asks the program:

  ```console
  codelead> /modernize --skip in-place
  ```

  ```output
  --skip needs --reason <why>: the stage's record and the audit pack keep why the program went on without it. /modernize --skip <stage> --reason <why> (the stages: read, recordings, pins, in-place, model, rebuild-init, rebuild, audit).
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The stage was skipped. |
| `2` | The skip was refused, and nothing was recorded: no `--reason` option, an empty reason, a word that is no stage, the recordings, a stage that is not failed or waiting, a folder with no program, a folder without the Pro plan, or a state file that cannot be read. |

## Files

| File | What `skip` does with it |
|---|---|
| `.codelead/modernization.json` | Reads it, and writes the stage's status, its problem (`null`) and its skip fields. See [Modernization state file](../reference/modernization-state-file.md#a-skipped-stage). |
| `.codelead/evidence/ev-*.json` | Writes one `modernize` record, with the status `skipped` and the outcome `success`. See [Evidence record format](../reference/evidence-record-format.md). |

## See also

- [`codelead modernize`](codelead-modernize.md)
- [`codelead modernize status`](codelead-modernize-status.md): see the skip and its reason.
- [`codelead modernize run`](codelead-modernize-run.md): run the skipped stage after all.
- [`codelead modernize next`](codelead-modernize-next.md): go on after the skip.
- [The modernization program](../modernization/the-program.md)
- [Fix a stop](../how-to/fix-a-stop.md#unblock-a-modernization-stage)
- [Modernization state file](../reference/modernization-state-file.md#a-skipped-stage)
