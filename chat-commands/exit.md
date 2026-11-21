---
title: /exit
description: End the CodeLead session - after it closes what the session left open.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /exit

**Applies to:** CodeLead 0.1 and later

## Name

`/exit` - Ends the session.

## Synopsis

```console
/exit
/quit
```

## Description

`/exit` ends the session of [`codelead interactive`](../cli/codelead-interactive.md), or the chat's session in Visual Studio Code. [`/quit`](quit.md) does the same. The words `exit` and `quit`, without the `/`, do the same too.

Before the session ends, CodeLead closes what the session left open:

- A new app whose build has unfinished steps gets a recovery report, `RECOVERY.md` in the project folder. It lists the steps that were done, and the next actions.
- A composed part whose gate has not passed is unapplied, so that its files are as they were before the patch. At a terminal, CodeLead asks first: `Unapply it, putting those files back as they were before the patch? [y/n]`. See [`/modernize`](modernize.md).
- The servers that an acceptance walk started are stopped.
- The chat's conversation ends with the session.

Then CodeLead prints `Exiting CodeLead.`. The session log closes with the reason `user_exit`. See [Sessions](../concepts/sessions.md).

> [!NOTE]
> **New to this?** To continue an `/implement` in a later session, type [`/session --pause`](session.md) instead. It saves where the `/implement` stands, then ends the session.

`/exit` takes no option and no words. `/help exit` shows its usage, and the chat's completion offers it when you type `/`.

## What you see

The result's outcome is `done`, and the command's status over JSON is `exited`. See [Session protocol](../reference/session-io-protocol.md).

## Examples

- End the session:

  ```console
  /exit
  ```

  ```output
  Exiting CodeLead.
  ```

- Show the usage of `/exit`:

  ```console
  /help exit
  ```

  ```output
  Help: /exit

  Usage:
    /exit
    /quit
  ```

- Give words after it. CodeLead does not know that form, and the session goes on:

  ```console
  /exit now
  ```

  ```output
  Unknown command: /exit

  Did you mean?
    /exit

  Use /help to see available commands.
  ```

## Files

| File | What `/exit` does with it |
|---|---|
| `.codeleadsessions/<session id>/` | Closes the session log. |
| `RECOVERY.md` | Writes it for a new app whose build has unfinished steps. |

## See also

- [`/quit`](quit.md): the same command.
- [`/session`](session.md): pause an `/implement` before you leave.
- [Stop and resume work](../how-to/stop-and-resume.md)
- [Chat command reference](index.md)
- [`codelead interactive`](../cli/codelead-interactive.md)
