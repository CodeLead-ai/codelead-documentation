---
title: codelead board unarchive
description: Bring one archived card, or several, back to the board, into the column each was archived from, as a person only.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board unarchive

**Applies to:** CodeLead 0.1 and later

`codelead board unarchive` brings archived cards back to the board. Each card returns to the column it was archived from. Only a person does this.

## Name

`codelead board unarchive` - Brings archived cards back to the board.

## Synopsis

```console
codelead board unarchive <ID>... --as <PERSON> [--if-updated <TIME>] [--board <DIR>] [--json]
codelead board unarchive -h|--help
```

## Description

`unarchive` removes the `archived` time from each card's front matter, and writes an `item.unarchived` event. The card shows again in its column on the page and in [`list`](board-list.md). A card that is not archived is left as it is.

The page's **Undo** after a bulk archive does the same, and so does **Bring it back to the board** on an archived card.

## Arguments

- **`<ID>...`**

  The card's number, or several numbers: `3 4 7`.

## Options

- **`--as <PERSON>`**

  Who brings the card back. It must be a person of the board. Required.

- **`--if-updated <TIME>`**

  Refuses the change when the card's `updated` time is not this one. With one id only.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the card and its events. With several ids, prints each card's result.

## Examples

- Bring one card back:

  ```console
  codelead board unarchive 2 --as sam
  ```

  ```output
  Brought #2 back to the board, in Done.
  ```

- Bring back a card that is not archived. Nothing is written:

  ```console
  codelead board unarchive 5 --as sam
  ```

  ```output
  #5 is not archived; nothing was written.
  ```

- See an agent refused:

  ```console
  codelead board unarchive 4 --as codex
  ```

  ```output
  Refused: codex is an agent; only a person brings an archived card back. Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Brought back, or not archived. |
| `1` | Refused: an agent, a participant not on the board, or a stale write. |
| `2` | No board, no such card, no `--as`, or `--if-updated` with several ids. |

## Files

`unarchive` removes `archived:` from each card's front matter in `items/` and appends `item.unarchived` to `events.jsonl`. See [Files and events](../files-and-events.md#archived-cards).

## See also

- [`codelead board archive`](board-archive.md)
- [`codelead board list`](board-list.md): `--archived` finds archived cards.
- [Rules for agents](../rules-for-agents.md#archiving-is-the-persons)
- [Use the board](../use-the-board.md#archive-a-card)
