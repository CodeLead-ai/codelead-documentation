---
title: /repair
description: Ask for a small patch that fixes the last failed validation, apply it with /edit, and know when the repair loop stops.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /repair

**Applies to:** CodeLead 0.1 and later

## Name

`/repair` - Asks a model for a minimal patch that fixes the last failed validation.

## Synopsis

```console
/repair
```

## Description

`/repair` reads the last failed validation of this session and the files that the task changed. It asks a model for the smallest patch that makes the validation pass. Then it checks that patch. It never applies the patch: [`/edit`](edit.md) applies it, after it asks you, and [`/validate`](validate.md) judges it. One `/repair`, `/edit` and `/validate` together are one repair round. See [Repair](../concepts/repair.md).

You use `/repair` after a `/validate` that failed on the code, for example a failing test or a compiler error. It needs two things from this session: a patch that was applied, and a validation that failed. Without them, it does nothing and says why.

> [!NOTE]
> **New to this?** A repair is a second, smaller change that fixes what the first change broke. CodeLead sends the model the failure, the files that the failure names, and what earlier rounds tried. You see the patch and you decide.

### How CodeLead chooses what the model sees

- The files that the failure names: the paths in the errors, and the types and members in the messages and the stack.
- The files that this task changed: the patches of this session and the earlier increments of the same evidence chain. Other changes in the working tree are listed apart.
- From the second round on, each earlier round: its diagnosis, its diff and the verdict that followed it.

### When the repair loop stops

CodeLead compares each file that a repair would change with the states that earlier rounds left. A repair that brings a file back to an earlier state is a cycle. At the first cycle, the next `/repair` asks once more, with the history and a wider scope. At the second cycle, the loop stops for you: the result says what each round tried. After that, `/repair` asks no model until the failure changes.

`/repair` never runs for a failure of the environment, such as the pins' database that does not answer. A patch cannot fix the environment.

> [!TIP]
> **For experienced users:** the repair uses the `repair` model role when the configuration sets one. Otherwise it uses the role of the patch strategy. See [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md). A pin or an in-place transform repairs itself. Its limit, `modernization.repair.rounds` (the default is 2), counts the rounds that did not move the failure. A round that ends with a new failure earns one more, up to twice the limit. For example, round 1 fixes a compiler error and round 2 moves the failure again, so a third round runs.

## Options

`/repair` takes no option and no request text. `/repair <words>` is refused with `Unknown /repair subcommand: <words>`. [`/validate --repair`](validate.md) is the same command.

## What CodeLead asks

`/repair` asks nothing. The question comes with `/edit`: `Apply this patch?`. A pin or a transform asks once before its own repair rounds: `The gate refused the pins: <failure>. Repair it? [Y/n]`. CodeLead recommends yes, and Enter gives yes.

## What you see

When the session knows the task's evidence chain, the headline starts with the round's number.

| Outcome | Headline | The way on |
|---|---|---|
| `done` | `Repair round 1 — the repair for MSBuild is ready to apply (1 file); nothing is changed yet` | `/edit`, `/patch --show` |
| `done` | `The repair for MSBuild is ready to apply (2 files); nothing is changed yet` (outside a chain) | `/edit`, `/patch --show` |
| `failed` | `The repair patch for MSBuild could not be checked: Hunk 1 does not match the file` | `/patch --retry`, `/patch --show` |
| `failed` | `The repair for MSBuild produced no patch: <the model's reason>` | `Change something, then /validate`, `/validate --override` |
| `failed` | `Failed to generate repair patch: <the error>` | `/repair` |
| `failed` | `The repair loop stopped: this repair returns <file> to its state after repair 1, which failed with <error>` | `Fix it by hand, then /validate`, `/validate --override` |
| `nothing` | `Nothing to repair: no failed validation in this session` | `/validate` |
| `nothing` | `Nothing to repair: no patch was applied in this session` | `/implement <request>` |

The facts of a ready repair are the files, the patch check, the command it repairs and the first error. For example: `Files: webapp.Characterization.Db/AccountControllerTests.cs`, `Patch check: can apply cleanly (3 hunks, +65 −12)`, `Repairs: dotnet test webapp.Characterization.Db.csproj --no-build -nologo (exit 1)`.

## Examples

- Ask for a repair after a failed validation in a pin's chain. This result comes from a session of the Basic Store sample on Windows:

  ```console
  /repair
  ```

  ```output
  ✔ Repair round 3 — the repair for dotnet test is ready to apply (1 file); nothing is changed yet; next: /edit
  ```

  Then apply it and validate it:

  ```console
  /edit
  /validate
  ```

  ```output
  ✔ Repair round 3 — applied the repair to 1 file; not validated yet; next: /validate
  ✔ Validation passed after repair round 3: 4 commands
  ```

- The model server does not answer. Nothing changed, and the way on is to run `/repair` again when the server runs:

  ```output
  Repair
  Using last failed validation result and current changed files.
  Generating minimal repair patch...
  Failed to generate repair patch.
  Could not connect to lmstudio at http://localhost:1234/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ✖ Failed to generate repair patch: Could not connect to lmstudio at http://localhost:1234/v1; next: /repair
  ```

  Check the model server with [`codelead doctor`](../cli/codelead-doctor.md), then run `/repair` again.

- There is nothing to repair, because no validation failed in this session:

  ```console
  /repair
  ```

  ```output
  No failed validation result found.

  Run /validate first, or use /validate --override if you manually verified the change.
  ○ Nothing to repair: no failed validation in this session; next: /validate
  ```

- A validation failed, but no patch was applied in this session. `/repair` repairs a change of this session only:

  ```output
  No applied patch context found.

  Repair needs a recently applied patch and a failed validation result.
  ○ Nothing to repair: no patch was applied in this session
  ```

  Fix the code by hand, or describe the fix with [`/implement`](implement.md).

- The loop meets the same state twice and stops for you:

  <!-- described, not captured -->
  ```output
  ✖ The repair loop stopped: this repair returns webapp.Characterization/Fakes/FakeApplicationSignInManager.cs to its state after repair 1, which failed with Expected 302, got 200; next: Fix it by hand, then /validate
  ```

  The facts list what each round tried, for example `Repair 1 changed the fake: the test failed.` Read the failure, fix the file by hand, then run `/validate`.

## Files

- Reads the last failed validation and the patches of this session, and the evidence chain in `.codelead/evidence/`.
- Writes the model's prompt and answer to the session folder (`.codeleadsessions/<session>/`).
- The round's evidence record (`.codelead/evidence/<id>.json`) follows the record before it. It is written when the round's validation runs, or immediately when the round applied nothing.

## See also

- [`/validate`](validate.md) and [`/edit`](edit.md)
- [`/patch`](patch.md)
- [Repair](../concepts/repair.md)
- [Evidence records](../concepts/evidence-records.md)
- [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md)
- [Fix a stop](../how-to/fix-a-stop.md)
