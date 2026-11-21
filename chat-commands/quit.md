---
title: /quit
description: End the CodeLead session - the same command as /exit.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /quit

**Applies to:** CodeLead 0.1 and later

## Name

`/quit` - Ends the session. The same command as `/exit`.

## Synopsis

```console
/quit
/exit
```

## Description

`/quit` is another name for [`/exit`](exit.md). It ends the session of [`codelead interactive`](../cli/codelead-interactive.md), or the chat's session in Visual Studio Code, after it closes what the session left open. The `/exit` page says what CodeLead closes.

`/quit` takes no option and no words. `/help quit` shows its usage, and the chat's completion offers it when you type `/`.

## What you see

The result's outcome is `done`, and the command's status over JSON is `exited`. CodeLead prints `Exiting CodeLead.`.

## Examples

- End the session:

  ```console
  /quit
  ```

  ```output
  Exiting CodeLead.
  ```

- The same, without the `/`:

  ```console
  quit
  ```

  ```output
  Exiting CodeLead.
  ```

- Show the usage of `/quit`:

  ```console
  /help quit
  ```

  ```output
  Help: /quit

  Usage:
    /exit
    /quit
  ```

## See also

- [`/exit`](exit.md): the same command, with the details.
- [`/session`](session.md): pause an `/implement` before you leave.
- [Chat command reference](index.md)
- [`codelead interactive`](../cli/codelead-interactive.md)
