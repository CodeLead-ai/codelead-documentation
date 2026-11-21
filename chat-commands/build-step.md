---
title: /build-step
description: Build the next approved step of a new app's plan, look at where the build stands, and repair a step whose validation failed.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /build-step

**Applies to:** CodeLead 0.1 and later

## Name

`/build-step` - Builds the next approved step of the plan, shows the build's state, or repairs a failed step.

## Synopsis

```console
/build-step
/build-step --show
/build-step --repair
```

## Description

`/build-step` builds one step of the plan that [`/plan --approve`](plan.md) approved. Each step goes through the same path:

1. A model writes the step's execution plan: the files it creates or changes, and how the step is checked. CodeLead asks you to approve it.
2. A model writes the step's patch. The patch check reads it.
3. CodeLead asks you to apply the checked patch.
4. CodeLead asks to run the step's validation, then runs it.
5. When the validation passes, the step is complete, and the session names the next step.

The first step of a plan often runs a scaffold command, such as an `npm create` command that makes the project's skeleton. CodeLead asks before it runs a scaffold and before it installs dependencies.

When a step's validation fails, the step needs a repair. `/build-step --repair` asks for a patch that fixes it, then asks to run the validation again. `/build-step --show` shows where the build stands at any time.

[`/create-app --auto`](create-app.md) runs `/build-step` for every step by itself and gives every recommended answer. It runs a failed scaffold command once more before it takes the recommended no. At a step with only a manual check, the step's gates decide, and the record says `manualValidation: "unconfirmed"`.

> [!NOTE]
> **New to this?** A step is a small part of the app, such as "the task list" or "the done checkbox". Building one step at a time lets you look at the app between steps.

## Options

- **`--show`**

  Shows the current step: its state, its execution plan and its validation.

- **`--repair`**

  Repairs the current step after its validation failed, then validates it again.

`/build-step` takes no request text. The forms without dashes, such as `/build-step repair`, are refused with the form to use.

## What CodeLead asks

| Question | CodeLead recommends | Enter gives |
|---|---|---|
| `Approve this build-step execution plan? [Y/n]` | Yes. | Yes. |
| `Apply checked patch? [Y/n]` (or `Apply generated patch?`, `Apply repair patch?`) | Yes, when the patch check passed with no blocking finding. | Yes. |
| `Run build-step validation commands? [Y/n]` | Yes. | Yes. |
| `Run validation again for step <id>? [Y/n]` (after a repair) | Yes. | Yes. |
| `Mark manual validation as passed? [y/N]` (a step with no validation command) | No: "CodeLead did not see it pass; say yes once you checked it yourself". | No. |
| `Run this scaffold command in the current target folder? [Y/n]` | Yes: "the plan calls for it". | Yes. |
| `Run install now? [Y/n]` and `Run dependency install now? [Y/n]` | Yes. | Yes. |
| `Retry scaffold command? [y/N]` (the scaffold failed) | No: "the last attempt failed; no falls back to model-generated files". | No. |
| `Delete the above and re-plan from scratch? [y/n]` (the recovery of a red build) | Nothing: it deletes files of the target folder. | Nothing: type `y` or `n`. |

## What you see

The build's commands keep their own words: the result's headline is the first line of the last thing the command said.

| Situation | Headline |
|---|---|
| A step passed | `✅ 👍 Step 2 of 6 completed. Validation passed.`, then `Next step:` and `Run /build-step again to continue.` |
| The last step passed | `✅ 🎉 All planned steps are complete. Validation passed.` |
| The step's validation failed | `Current build step needs repair.`, with the failed command and the way on: `Run /build-step --repair` |
| You declined the execution plan | `Build-step paused. No patch was generated.` |
| The plan is not approved | `Greenfield plan is not approved.`, with `Approve with /plan --approve` |
| No plan | `No greenfield plan found.` |

## Examples

- Build the next step of an approved plan:

  ```console
  /build-step
  ```

  <!-- described, not captured -->
  ```output
  Build-step execution plan ready.

  Step: <step id> - <step title>
  …
  Approve this build-step execution plan? [Y/n]
  …
  Apply checked patch? [Y/n]
  …
  Run build-step validation commands? [Y/n]
  …
  ✅ 👍 Step 2 of 6 completed. Validation passed.
  Manual validation and remaining visual/product criteria may still be outstanding.

  Next step:
  <next step id> — <next step title>

  Run /build-step again to continue.
  ```

- A step's validation failed. This is a stop, and its way on is a repair:

  <!-- described, not captured -->
  ```output
  Current build step needs repair.

  Step: <step id> - <step title>
  Status: validation failed

  The original patch was already applied, but validation failed.

  Failed command:
  npm run build

  Next:
  - Run /build-step --repair
  - Or fix manually and rerun validation
  - Or use /build-step --show to inspect state
  ```

- Run `/build-step` when no plan exists:

  ```console
  /build-step
  ```

  ```output
  No greenfield plan found.

  Create one with:
  /create-app --plan <request>
  ```

- Request text is refused:

  ```console
  /build-step now
  ```

  ```output
  /build-step does not accept request text.

  Usage:
    /build-step
    /build-step --show
    /build-step --repair
  ```

## Files

- Reads the plan and writes the build's state in the session folder: `.codeleadsessions/<session>/greenfield/build.json` and `state.json`.
- Writes the app's files through the patch check, inside the target folder.

## See also

- [`/plan`](plan.md)
- [`/create-app`](create-app.md)
- [`/greenfield`](greenfield.md)
- [`/build-incremental`](build-incremental.md)
- [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md)
- [Validation and the gate](../concepts/validation-and-the-gate.md)
