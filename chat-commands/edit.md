---
title: /edit
description: Apply the last checked patch to your files after CodeLead asks you, and read what was applied.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /edit

**Applies to:** CodeLead 0.1 and later

## Name

`/edit` - Applies the last checked patch to the working tree, after it asks you.

## Synopsis

```console
/edit
```

## Description

`/edit` applies the patch that the last [`/patch --check`](patch.md) passed. It shows what it will change, says what CodeLead recommends, and asks `Apply this patch?`. Only a yes changes files. A patch (a description of changes to files) can come from [`/patch`](patch.md), from [`/repair`](repair.md), or from a paused [`/implement`](implement.md) that you resumed.

The order is always the same: a patch, then its check, then `/edit`, then [`/validate`](validate.md). `/edit` refuses a patch that was not checked, a patch that changed after its check, and a patch whose check did not pass. It never runs the validation itself.

> [!NOTE]
> **New to this?** "Apply" means that CodeLead writes the changes of the patch into your files. Before that moment, nothing in your folder changed. After it, use `git diff` to see the change, and `/validate` to test it.

### What `/edit` refuses

- No patch in this session: `No patch proposal is available. Run /patch <request> first.`
- A patch that was never checked: `Run /patch --check before /edit.`
- A patch that changed after its check: `The patch has changed since the last check. Run /patch --check again before /edit.`
- A check that did not pass: `The last patch check did not pass. Run /patch --check and resolve the reported issues before using /edit.`
- A repair that would bring a file back to an earlier state: `Not applied: <the cycle in words>`. See [`/repair`](repair.md).

> [!TIP]
> **For experienced users:** a patch applied in a repair round names the round in its result. `/edit` refuses a repair cycle too, because a repair patch can also come from `/patch --retry`.

## Options

`/edit` takes no option and no words. CodeLead refuses another form as invalid input, applies nothing, and prints the usage. With words after it, CodeLead says:

```output
/edit takes no request: add a total. It applies the last checked patch: /patch <request>, then /patch --check, then /edit.

Usage:
  /edit
```

With `--from-clipboard`, it says `Unknown option: --from-clipboard`, then the usage.

## What CodeLead asks

| Question | CodeLead recommends | Enter gives |
|---|---|---|
| `Apply this patch? [Y/n]` when the check passed with no blocking finding | Yes. The words before it say so: `The patch check passed. CodeLead recommends applying.` Notes never change the yes. | Yes. |
| `Apply this patch? [y/N]` when the check has a blocking finding | No: `The patch check has 1 blocking finding. CodeLead recommends not applying it.` | No. |

## What you see

| Outcome | Headline | The way on |
|---|---|---|
| `done` | `Applied the patch to 2 files; not validated yet` | `/validate` |
| `done` | `Repair round 1 — applied the repair to 1 file; not validated yet` | `/validate` |
| `failed` | `The patch did not apply: <the first problem>` | `/patch --check`, `/patch --retry` |
| `stopped` | `Nothing applied: the answer was no` | `/edit` |
| `stopped` | `Run /patch --check before /edit.` | `/patch --check` |
| `nothing` | `No patch to apply` | `/patch <request>` |
| `failed` | `/edit takes no request: <words>. It applies the last checked patch: /patch <request>, then /patch --check, then /edit` | `/help` |

The facts of an apply are the files with their operation and the hunks: `Files: src/tasks.js (modify), test/tasks.test.js (modify)` and `Hunks applied: 3 (+10 −1)`. A patch that you declined is kept in the session: run `/edit` again to apply it.

## Examples

- Apply a checked patch, and press Enter to accept the recommended yes:

  ```console
  /edit
  ```

  ```output
  Apply checked patch

  Files: 2
  Hunks: 3
  Additions: 10
  Removals: 1

  This will modify files in your working tree.
  The patch check passed. CodeLead recommends applying.

  Apply this patch? [Y/n]
  Patch applied successfully.

  Files changed: 2
  Files created: 0
  Files deleted: 0
  Hunks applied: 3

  Files:
  - src/tasks.js (modify)
  - test/tasks.test.js (modify)

  Next:
  Run your validation commands.
  ✔ Applied the patch to 2 files; not validated yet; next: /validate
  ```

- Answer no. Nothing changes, and the patch stays in the session:

  ```output
  Apply this patch? [Y/n] n
  Edit canceled. No files were changed.
  ■ Nothing applied: the answer was no; next: /edit
  ```

- Run `/edit` before the patch check. This is a stop with its way on:

  ```console
  /edit
  ```

  ```output
  Run /patch --check before /edit.
  ```

  Run `/patch --check`, then `/edit` again.

- Run `/edit` when the session has no patch:

  ```output
  No patch proposal is available. Run /patch <request> first.
  ○ No patch to apply
  ```

## Files

- Changes the files that the patch names, inside the repository only.
- Writes the apply's result to the session log. After an `/implement`, it also writes the task's state to `.codeleadsessions/<session>/implement/state.json`.

## See also

- [`/patch`](patch.md)
- [`/validate`](validate.md)
- [`/repair`](repair.md)
- [`/implement`](implement.md)
- [The patch check](../concepts/the-patch-check.md)
- [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md)
