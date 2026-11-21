---
title: /spec
description: Show the full project specification of a new app, what CodeLead will build, before you approve its plan.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /spec

**Applies to:** CodeLead 0.1 and later

## Name

`/spec` - Shows the full project specification of the new app.

## Synopsis

```console
/spec
/spec --show
```

## Description

The project specification says what the new app does. The plan says how CodeLead builds it. [`/create-app`](create-app.md) writes both. `/spec` prints the specification:

- The project's name, type and stack.
- Each feature, with its behaviour and its acceptance criteria (each marked `✓`).
- The data model: each entity and its fields. The steps never invent fields that are not here.
- The screens, with their routes and their content.
- The non-goals, the constraints, the assumptions, the validation and the risks.

You use `/spec` before [`/plan --approve`](plan.md), to check that the app is the one you asked for. When it is not, revise the plan with `/plan --revise <instruction>`.

`/spec` reads only. It calls no model.

> [!NOTE]
> **New to this?** An acceptance criterion is a sentence that you can check on the running app. For example: "a checked task shows as done". Read them: they are what "finished" means for this app.

## Options

- **`--show`**

  The same as `/spec`.

Any other word after `/spec` is refused: `Unknown /spec subcommand: <word>`.

## What you see

`/spec` is informational: its output is the answer, so its result has no headline. Without a plan, it prints `No greenfield plan found.` and the command that creates one.

## Examples

- Show the specification before you approve the plan:

  ```console
  /spec
  ```

  <!-- described, not captured -->
  ```output
  Project Specification

  Project: <the project's name> (<its type>)
  Stack: …

  Features:
  - …
      ✓ …
  …
  Non-goals: …
  Constraints: …
  ```

- Show it when no plan exists yet:

  ```console
  /spec
  ```

  ```output
  No greenfield plan found.

  Create one with:
  /create-app --plan <request>
  ```

- A word that `/spec` does not know is refused:

  ```console
  /spec extra
  ```

  ```output
  Unknown /spec subcommand: extra

  Usage:
    /spec
    /spec --show
  ```

## Files

- Reads the specification from the session: `.codeleadsessions/<session>/greenfield/spec.json`. It writes no file.

## See also

- [`/plan`](plan.md)
- [`/create-app`](create-app.md)
- [`/greenfield`](greenfield.md)
- [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md)
