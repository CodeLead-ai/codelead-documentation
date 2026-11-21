---
title: /status
description: Show the state of the current session - the repository, the strategy, the last request, patch and validation, and the command that CodeLead recommends next.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /status

**Applies to:** CodeLead 0.1 and later

## Name

`/status` - Shows the state of the current session.

## Synopsis

```console
/status
```

## Description

`/status` prints a box with the state of this session. It shows the repository, the [strategy](strategy.md), the project type, and the languages, tools and validation commands that CodeLead found. It also says what the session holds. These are the last request, [preflight](preflight.md) and [scan](scan.md), the last patch and its check, and the last apply and validation. Its last lines name the command that CodeLead recommends next.

`/status` changes nothing. It calls no model and writes no file.

> [!NOTE]
> **New to this?** A session remembers what you did since it started. When you do not know where you are, type `/status`. The line `Next recommended command` says what to do next.

`/status` takes no option and no words. `/help status` shows its usage, and the chat's completion offers it when you type `/`.

## What you see

`/status` is informational: the box is the answer, so its result has no headline. Its outcome is `done`.

## Examples

- Show the state of a session that has done nothing yet:

  ```console
  /status
  ```

  ```output
  ╭──────────────────────────────────────────────────────────────────────────────╮
  │                                                                              │
  │   CodeLead Session Status                                                    │
  │                                                                              │
  │   Repo: <your project>                                                       │
  │   Strategy: auto                                                             │
  │   Project type: greenfield                                                   │
  │   Languages: none detected                                                   │
  │   Tools: none detected                                                       │
  │   Validation: none detected                                                  │
  │   Prompts processed: 1                                                       │
  │   Last request: (none)                                                       │
  …
  │   Last patch check: no                                                       │
  │   Last edit result: no                                                       │
  │   Patch retries: 0                                                           │
  │   Next recommended command: /patch <request>                                 │
  │   Last validation: none                                                      │
  │                                                                              │
  ╰──────────────────────────────────────────────────────────────────────────────╯
  ```

- Show the usage of `/status`:

  ```console
  /help status
  ```

  ```output
  Help: /status

  Usage:
    /status
  ```

- Give words after it. CodeLead does not know that form, and suggests the command:

  ```console
  /status now
  ```

  ```output
  Unknown command: /status

  Did you mean?
    /status

  Use /help to see available commands.
  ```

## Files

`/status` reads no file and writes none.

## See also

- [Chat command reference](index.md)
- [`/help`](help.md): lists every command.
- [`/strategy`](strategy.md): shows or sets the strategy that `/status` names.
- [`/session`](session.md): pauses or resumes an `/implement`.
- [Sessions](../concepts/sessions.md)
