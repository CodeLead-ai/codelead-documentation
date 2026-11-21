---
title: /patch
description: Ask a model for a patch without applying it, look at it, check it against your files, and ask for a new one when the check refuses it.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /patch

**Applies to:** CodeLead 0.1 and later

## Name

`/patch` - Asks for a patch proposal, shows it, checks it, or asks for a new one.

## Synopsis

```console
/patch <REQUEST>
/patch --from-clipboard
/patch --from-preflight
/patch --show
/patch --check
/patch --retry
```

## Description

A patch is a description of changes to files, in the unified diff format that `git diff` prints. `/patch` is the step-by-step way to get one: it never applies anything. You look at the patch with `/patch --show`, check it with `/patch --check`, and apply it with [`/edit`](edit.md). See [Requests, plans and patches](../concepts/requests-plans-and-patches.md).

[`/implement`](implement.md) runs these steps for you. Use `/patch` when you want to stop between the steps, or when you continue a patch from a resumed session.

> [!NOTE]
> **New to this?** The patch check is CodeLead's own reading of the patch against your files, with no model. It answers two questions: does every change fit the file as it is now, and is every path safe to change? See [The patch check](../concepts/the-patch-check.md).

### What the patch check reads

- Each hunk (one block of changed lines) must match the current file. A hunk that does not match is searched for nearby, then in the whole file.
- Every path must be inside the repository.
- The patch must not target generated, dependency, build, cache or session files.
- The patch must not target sensitive files, such as keys.

## Arguments

- **`<REQUEST>`**

  The change you want, in plain words. For example: `add a clearCompleted function that removes the done tasks, with a test`.

## Options

- **`--from-clipboard`**

  Takes the request from the clipboard text in place of the words on the line.

- **`--from-preflight`**

  Asks for the patch from the last [`/preflight`](preflight.md) result: its plan of files and its constraints. With no preflight in the session, it stops and says so.

- **`--show`**

  Shows the last patch proposal: its request, its files, its counts and a preview of the diff.

- **`--check`**

  Runs the patch check on the last patch proposal. A check that passes lets `/edit` apply the patch.

- **`--retry`**

  Asks the model for a new patch after a check that refused the last one. The model gets the check's problems, and the retry asks for a smaller patch with smaller hunks.

The forms without dashes, such as `/patch show`, are refused with the form to use: `Use: /patch --show`.

## What CodeLead asks

`/patch` asks nothing. It changes no file, so it needs no yes.

## What you see

| Command | Outcome | Headline | The way on |
|---|---|---|---|
| `/patch <request>` | `done` | `A patch is ready: 1 file (+3 −1); nothing is changed yet` | `/patch --check`, `/patch --show` |
| `/patch <request>` | `failed` | `Patch proposal failed: <the error>` | The error's own way out, then `Adjust the request and run /patch again` |
| `/patch <request>` | `failed` | `The patch proposal held no valid diff` | `Adjust the request and run /patch again` |
| `/patch --show` | `done` | `The patch: 2 files (+10 −1)` | `/patch --check`, or `/edit` when the check passed |
| `/patch --show` | `nothing` | `No patch proposal` | `/patch <request>` |
| `/patch --check` | `passed` | `Patch check passed: 2 files can apply cleanly` | `/edit` |
| `/patch --check` | `failed` | `The patch check refused the patch: <the first problem>` | `/patch --retry`, `/patch --show` |
| `/patch --check` | `nothing` | `No patch proposal to check` | `/patch <request>` |
| `/patch --retry` | `done` | `A retried patch is ready: …; nothing is changed yet` | `/patch --check`, `/patch --show` |
| `/patch --retry` | `failed` | `Failed to retry patch proposal: <the error>` | `/patch --retry` |

The facts of a check are the files and the counts: `Files: src/tasks.js, test/tasks.test.js` and `Patch check: can apply cleanly (3 hunks, +10 −1)`.

## Examples

- Look at the patch that waits in the session:

  ```console
  /patch --show
  ```

  ```output
  Patch Summary
  Request: add a clearCompleted function that removes the done tasks, with a test
  Parse status: valid unified diff
  Files affected: 2
  Added lines: 10
  Removed lines: 1
  Warnings: none
  Affected files:
  - src/tasks.js
  - test/tasks.test.js
  Patch preview:
  --- a/src/tasks.js
  +++ b/src/tasks.js
  @@ -9,3 +9,7 @@
   export function openCount(tasks) {
     return tasks.filter((task) => !task.done).length;
   }
  +
  +export function clearCompleted(tasks) {
  +  return tasks.filter((task) => !task.done);
  +}
  …
  Use /copy --patch to copy the full patch.
  ✔ The patch: 2 files (+10 −1); next: /patch --check
  ```

- Check the patch. Every file can apply, and the way on is `/edit`:

  ```console
  /patch --check
  ```

  ```output
  Patch check
  Status: can apply cleanly
  Files:
    ✓ src/tasks.js
    ✓ test/tasks.test.js
  Summary:
    Files changed: 2
    Hunks checked: 3
    Additions: 10
    Removals: 1
  Safety:
    ✓ All paths are inside the repository.
    ✓ No generated/dependency/build/cache/session artifact files are targeted.
    ✓ No sensitive files are targeted.
    ✓ Patch applies cleanly in check mode.

  Next:
    Review the patch with /patch --show.
    Apply with /edit after confirmation.
  ✔ Patch check passed: 2 files can apply cleanly; next: /edit
  ```

- The check refuses a patch that was written against an older file. This is a stop, and the way on is a retry:

  ```output
  Patch check
  Status: cannot apply cleanly
  Files:
    ✗ src/tasks.js
    ✓ test/tasks.test.js
  …
  Problems:
    - src/tasks.js: Hunk 1 does not match current file contents. Tried line 9 and searched nearby/whole file. Expected context line was not found: "export function countOpen(tasks) {". …

  Safety:
    ✓ All paths are inside the repository.
    ✓ No generated/dependency/build/cache/session artifact files are targeted.
    ✓ No sensitive files are targeted.
    ✗ Patch does not apply cleanly in check mode.

  Next:
    Regenerate the patch, or adjust the request with more context.
    Review the patch with /patch --show.
  ✖ The patch check refused the patch: src/tasks.js: Hunk 1 does not match current file contents. …; next: /patch --retry
  ```

  Run `/patch --retry`. When the retry is refused too, say more in the request, for example the function and the file.

- Ask for a patch when the model server does not run:

  ```console
  /patch add a clearCompleted function that removes the done tasks
  ```

  ```output
  Patch proposal failed.
  Failed to generate patch proposal.
  Could not connect to lmstudio at http://localhost:1234/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ✖ Patch proposal failed: Could not connect to lmstudio at http://localhost:1234/v1; next: If this is LM Studio, make sure it is open and the Local Server is running
  ```

- Ask for a patch from a preflight that was not run:

  ```console
  /patch --from-preflight
  ```

  ```output
  No preflight result is available. Run /preflight <request> first.
  ```

## Files

- Reads the files of the repository that the request and the plan name.
- Writes the model's prompt and answer to the session folder (`.codeleadsessions/<session>/`). After an `/implement`, the proposal also goes to the task's state, `.codeleadsessions/<session>/implement/state.json`.
- Changes no file of the project.

## See also

- [`/edit`](edit.md)
- [`/implement`](implement.md)
- [`/preflight`](preflight.md)
- [`/copy`](copy.md)
- [The patch check](../concepts/the-patch-check.md)
- [Requests, plans and patches](../concepts/requests-plans-and-patches.md)
