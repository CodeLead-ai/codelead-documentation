---
title: codelead board validate
description: Check every file of the board - board.json, the cards, the event log and the cursors - and name each problem, with exit code 1 when something is wrong.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board validate

**Applies to:** CodeLead 0.1 and later

`codelead board validate` reads every file of the board and checks it against the board's rules. It names each problem with its file, and exits with code 1 when there is one.

## Name

`codelead board validate` - Checks every file of the board.

## Synopsis

```console
codelead board validate [--board <DIR>] [--json]
codelead board validate -h|--help
```

## Description

Run `validate` after an edit of the board's files by hand, and before you commit them. Run it also when a write is refused because a card file has problems. It reads, and it writes nothing.

It checks:

- **`board.json`**: its shape, the column and participant ids, and that `agents.mayMoveTo` names no person's column.
- **`AGENTS.md`**: that it is there.
- **Each card**:
  - it reads as a card, and its file name starts with its id;
  - its type, its column and its author are on the board;
  - each comment is by a participant;
  - each relation has a known kind and names a card on the board;
  - `decided` holds an option, a participant and a day;
  - no two cards share an id;
  - no text holds a secret, a machine path or a private name.
- **`events.jsonl`**: each line is JSON with `at`, `by` and `kind`, and the kind is known. `by` is a participant or `unknown`. `addressed` and `decided` name participants.
- **Each cursor**: that it is JSON, that its participant is on the board, and that its times and line count are valid.

> [!NOTE]
> **New to this?** A card file with a problem still shows on the page, with a warning. Every write to it is refused until you fix the file, so the board never makes a bad file worse.

## Options

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints `{ "problems": [ { "file", "message" } ], "cards", "events", "cursors" }`.

## Examples

- Check a good board:

  ```console
  codelead board validate
  ```

  ```output
  The board is valid: 6 cards, 37 events, 2 cursors.
  ```

- Find a card that an edit by hand broke. Here a line was added after `## Thread` without a comment heading:

  ```console
  codelead board validate
  ```

  ```output
  The board has 1 problem (6 cards, 40 events, 2 cursors):
    items/0004-export-the-orders-as-csv.md: line 15: the thread holds text before its first comment; a comment begins with "### <participant> · <time>"
  ```

  A write to that card is refused until the file is fixed:

  ```output
  Refused: items/0004-export-the-orders-as-csv.md has problems: line 15: the thread holds text before its first comment; a comment begins with "### <participant> · <time>". Nothing was written; `codelead board validate` lists them.
  ```

  Move the line above `## Thread`, into the description, and run `validate` again.

- Read the result as JSON:

  ```console
  codelead board validate --json
  ```

  ```output
  {
    "problems": [],
    "cards": 6,
    "events": 39,
    "cursors": 2
  }
  ```

- Check a second board in another folder:

  ```console
  codelead board validate --board other-board
  ```

  ```output
  The board is valid: 0 cards, 0 events, 0 cursors.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The board is valid. |
| `1` | At least one problem. Each is listed with its file. |
| `2` | No board: there is no `board.json` in the folder. |

## Files

`validate` reads `board.json`, `AGENTS.md`, every file in `items/`, `events.jsonl`, `cursors/*.json` and `private-names.txt`. It writes nothing. See [Files and events](../files-and-events.md).

## See also

- [Files and events](../files-and-events.md): the format that `validate` checks.
- [Rules for agents](../rules-for-agents.md)
- [`codelead board check`](board-check.md): records hand edits as events by `unknown`.
- [`codelead board show`](board-show.md): shows a card with its `Problems:` line.
- [Use the board](../use-the-board.md#check-that-it-worked)
