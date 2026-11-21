---
title: codelead board archive
description: Archive one card, or several - each leaves the board and stays in its file, searchable - as a person only.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board archive

**Applies to:** CodeLead 0.1 and later

`codelead board archive` takes cards off the board. Each card stays in its file, and you can find it and bring it back. Only a person archives a card.

## Name

`codelead board archive` - Archives cards.

## Synopsis

```console
codelead board archive <ID>... --as <PERSON> [--if-updated <TIME>] [--board <DIR>] [--json]
codelead board archive -h|--help
```

## Description

An archived card gets an `archived` time in its front matter, and its `item.archived` event records the column it stood in. The card leaves the page's columns and the output of [`list`](board-list.md). `list --archived` and the page's **archived** filter show it again. [`codelead board unarchive`](board-unarchive.md) brings it back to the same column.

A card that is archived already is left as it is. With several ids, each card is archived on its own, and the output lists each card.

> [!NOTE]
> **New to this?** Archive the cards in Done that you no longer need to see. On the page, a card in Done has an **Archive** button, and the bulk bar archives several cards. See [Projects, themes and bulk operations](../projects-themes-and-bulk.md#change-several-cards-at-once).

## Arguments

- **`<ID>...`**

  The card's number, or several numbers: `3 4 7`.

## Options

- **`--as <PERSON>`**

  Who archives the card. It must be a person of the board. Required.

- **`--if-updated <TIME>`**

  Refuses the change when the card's `updated` time is not this one. With one id only.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the card and its events. With several ids, prints each card's result.

## Examples

- Archive one card:

  ```console
  codelead board archive 2 --as sam
  ```

  ```output
  Archived #2: it left the board and stays in its file (`list --archived` shows it).
  ```

- Archive two cards at once:

  ```console
  codelead board archive 1 2 --as sam
  ```

  ```output
  Archived (2 changed):
    #1 archived from Done
    #2 archived from Done
  ```

- Archive a card that is archived already. Nothing is written:

  ```console
  codelead board archive 1 --as sam
  ```

  ```output
  #1 was archived already; nothing was written.
  ```

- Find the archived cards again:

  ```console
  codelead board list --archived --column done
  ```

  ```output
  Done (2 of 2)
    #1    task     Show the stock on the order form  [area:orders, repo:webapp, by:sam, for:codelead]
    #2    question Must a client see the orders of other clients?  [area:orders, repo:webapp, by:claude]
  ```

- See an agent refused:

  ```console
  codelead board archive 4 --as codex
  ```

  ```output
  Refused: codex is an agent; only a person archives a card. Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Archived, or archived already. |
| `1` | Refused: an agent, a participant not on the board, or a stale write. |
| `2` | No board, no such card, no `--as`, or `--if-updated` with several ids. |

## Files

`archive` adds `archived:` to each card's front matter in `items/` and appends `item.archived` to `events.jsonl`. See [Files and events](../files-and-events.md#archived-cards).

## See also

- [`codelead board unarchive`](board-unarchive.md): bring a card back.
- [`codelead board list`](board-list.md): `--archived` shows archived cards.
- [`codelead board move`](board-move.md)
- [Rules for agents](../rules-for-agents.md#archiving-is-the-persons)
- [Cards, columns and participants](../concepts.md#columns-and-whose-they-are)
