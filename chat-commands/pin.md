---
title: /pin
description: List what waits on you in this project, add an item by hand, or mark an item as done.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /pin

**Applies to:** CodeLead 0.1 and later

## Name

`/pin` - Lists what waits on you, adds an item by hand, or marks one as done.

## Synopsis

```console
/pin
/pin add <TEXT>
/pin --done <ID>
```

## Description

CodeLead keeps a short list of what waits on a person in `docs/codelead/PINNED.md`: decisions, answers, reviews and fixes by hand. Sessions add items to it, and you can add your own. `/pin` shows the open items. `/pin add` adds one, and `/pin --done` closes one.

You use `/pin` at the start of a day, to see what waits on you, and when you want to remember a decision for later.

> [!IMPORTANT]
> `/pin` is not [`/modernize --pin`](modernize.md#--pin). `/pin` manages a list of items for people. `/modernize --pin` writes pins: tests that record what old code does today, before a modernization changes it. The two share a word and nothing else.

## Arguments

- **`add <TEXT>`**

  Adds an item with this text. The word `add` has no dashes. The text is required.

- **`<ID>`**

  The id of an open item, as `/pin` lists it, for example `pin-202611200429-26d8j`.

## Options

- **`--done <ID>`**

  Marks the item as done and records who closed it. It takes exactly one id. The form without dashes, `/pin done`, is refused.

## What you see

| Command | Headline |
|---|---|
| `/pin` | `Nothing is waiting on you. (docs/codelead/PINNED.md)`, or `Pinned (1 open)` and the list |
| `/pin add <text>` | `Pinned pin-202611200429-26d8j: <text>` |
| `/pin --done <id>` | `Resolved <id>: <title>`. For an id that is not open, the outcome is `failed`, with `No open pin with id <id>: nothing was resolved` and the way on `/pin`. |

## Examples

- Add an item, then list the open items:

  ```console
  /pin add Decide the target framework for the rebuild
  /pin
  ```

  ```output
  Pinned pin-202611200429-26d8j: Decide the target framework for the rebuild
  Pinned (1 open)
    pin-202611200429-26d8j  Decide the target framework for the rebuild
        Decide the target framework for the rebuild

  Resolve with /pin --done <id>; the full list is in docs/codelead/PINNED.md.
  ```

- List the items when nothing waits:

  ```console
  /pin
  ```

  ```output
  Nothing is waiting on you. (docs/codelead/PINNED.md)
  ```

- Close an item with an id that is not open. The command fails, and nothing changes:

  ```console
  /pin --done pin-1
  ```

  ```output
  No open pin with id pin-1.
  ✖ No open pin with id pin-1: nothing was resolved; next: /pin
  ```

- The form without dashes is refused:

  ```console
  /pin done
  ```

  ```output
  Unsupported command form: /pin done

  Use:
    /pin --done <id>
  ```

## Files

- Reads and writes `docs/codelead/PINNED.md`.
- [`/log --rebuild`](log.md) writes the list again from every session on disk.

## See also

- [`/log`](log.md)
- [`/modernize`](modernize.md)
- [Sessions](../concepts/sessions.md)
- [Files and folders](../reference/files-and-folders.md)
