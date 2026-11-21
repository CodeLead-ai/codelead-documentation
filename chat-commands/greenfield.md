---
title: /greenfield
description: See where the plan and the build of a new app stand, and which command comes next.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /greenfield

**Applies to:** CodeLead 0.1 and later

## Name

`/greenfield` - Shows where the new app's plan and build stand.

## Synopsis

```console
/greenfield --status
```

## Description

A greenfield project is a new app that starts from an empty folder. `/greenfield --status` prints its state in one block:

- the project's name, the plan's status, whether the plan is approved, and the number of steps;
- the build's progress, step by step;
- the target folder, the plan's quality and the current phase;
- the next command.

You use it when you come back to a new app, or when you are not sure which command comes next. It reads only.

A session in a folder with a build in progress continues that build: `/greenfield --status` shows where it stands.

## Options

- **`--status`**

  Shows the state. It is required: `/greenfield` alone and `/greenfield status` are refused.

## What you see

`/greenfield --status` is informational: its output is the answer, so its result has no headline. Without a new app in this session, it prints `No greenfield session found.`.

## Examples

- Show the state of a new app:

  ```console
  /greenfield --status
  ```

  <!-- described, not captured -->
  ```output
  Greenfield Status

  Project: <the project's name>
  Status: approved
  Plan approved: yes
  Steps: 6

  Build progress:
  …
  Target folder: …
  Plan quality: …
  Current phase: …

  Next:
  …
  ```

- Show it when no new app was planned:

  ```console
  /greenfield --status
  ```

  ```output
  No greenfield session found.

  Create one with:
  /create-app --plan <request>
  ```

- The form without dashes is refused:

  ```console
  /greenfield status
  ```

  ```output
  Unsupported command form: /greenfield status

  Use:
    /greenfield --status
  ```

## Files

- Reads the session's greenfield state: `.codeleadsessions/<session>/greenfield/state.json`. It writes no file.

## See also

- [`/create-app`](create-app.md)
- [`/plan`](plan.md)
- [`/build-step`](build-step.md)
- [Stop and resume work](../how-to/stop-and-resume.md)
