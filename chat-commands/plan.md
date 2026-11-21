---
title: /plan
description: Show, approve or revise the execution plan of a new app before CodeLead builds it.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /plan

**Applies to:** CodeLead 0.1 and later

## Name

`/plan` - Shows, approves or revises the plan of the new app.

## Synopsis

```console
/plan --show
/plan --approve
/plan --revise <INSTRUCTION>
```

## Description

[`/create-app --plan`](create-app.md) writes an execution plan: the numbered steps that build the new app. `/plan` is how you take part in it before anything is built.

1. `/plan --show` shows the plan: its status, the project, the stack, the target folder, the plan's quality and each step.
2. `/plan --revise <INSTRUCTION>` asks the planner to change the plan, for example `use a dark theme` or `add a search box to the list`. The plan is a draft again after a revision.
3. `/plan --approve` approves the plan. Then [`/build-step`](build-step.md) builds it one step at a time.

`/plan` never builds anything and never changes the app's files. Only `--revise` calls a model.

> [!NOTE]
> **New to this?** Read the plan the way you read a ticket before work starts. When a step is missing or wrong, revise the plan now: changing a plan is cheaper than changing code. Use [`/spec`](spec.md) to read what the app will do.

## Arguments

- **`<INSTRUCTION>`**

  What to change in the plan, in plain words. Used with `--revise`.

## Options

- **`--show`**

  Shows the current plan.

- **`--approve`**

  Approves the current plan. The next command is `/build-step`.

- **`--revise <INSTRUCTION>`**

  Asks the planner to revise the specification and the plan. The session prints `Plan revised.` and a summary of the changes.

`/plan` alone is refused, and so are the forms without dashes, such as `/plan show`. `--show` and `--approve` take no text.

## What you see

| Command | Headline | The way on |
|---|---|---|
| `/plan --show` | `Greenfield Plan`, with the steps | `/spec`, `/plan --approve`, `/plan --revise <instruction>`; after the approval, `/build-step` |
| `/plan --approve` | `Plan approved.` | `/build-step` |
| `/plan --revise …` | `Plan revised.`, with the summary of changes | Show it again, then approve it. |
| Any form, with no plan | `No greenfield plan found.` | `/create-app --plan <request>` |

## Examples

- Approve the plan:

  ```console
  /plan --approve
  ```

  <!-- described, not captured -->
  ```output
  Plan approved.

  Next:
  - /build-step
  ```

- Revise the plan before you approve it:

  ```console
  /plan --revise add a search box above the task list
  ```

  <!-- described, not captured -->
  ```output
  Revising greenfield plan...
  …
  Plan revised.

  Summary of changes:
  - …
  ```

- Show the plan when there is none yet:

  ```console
  /plan --show
  ```

  ```output
  No greenfield plan found.

  Create one with:
  /create-app --plan <request>
  ```

- A revision without its instruction is refused:

  ```console
  /plan --revise
  ```

  ```output
  Missing instruction for /plan --revise.

  Usage:
    /plan --show
    /plan --approve
    /plan --revise <instruction>
  ```

## Files

- Reads and writes the plan in the session folder: `.codeleadsessions/<session>/greenfield/plan.json`, `spec.json` and `state.json`.

## See also

- [`/create-app`](create-app.md)
- [`/spec`](spec.md)
- [`/build-step`](build-step.md)
- [`/greenfield`](greenfield.md)
- [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md)
