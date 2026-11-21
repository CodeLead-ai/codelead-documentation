---
title: codelead board move
description: Move one card, or several, to a column, with an optional comment written in the same step; an agent moves only into the columns it may use.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board move

**Applies to:** CodeLead 0.1 and later

`codelead board move` moves one card, or several, to a column. With `--comment`, it writes a comment on each card in the same step. An agent moves cards only into the columns that `board.json` allows it.

## Name

`codelead board move` - Moves cards to a column.

## Synopsis

```console
codelead board move <ID>... --to <COLUMN> --as <PARTICIPANT> [--comment <MARKDOWN>] [--if-updated <TIME>] [--rewrite-paths] [--board <DIR>] [--json]
codelead board move -h|--help
```

## Description

The card moves to the top of the column, and its `item.moved` event is written. A card that is already in the column is left as it is, and nothing is written. With `--comment`, the comment is added even then.

With `--comment`, the comment and the move share one time, and a check shows them as one line: "commented and moved it to …". A comment whose first line starts with `Landed:` also adds the words after `Landed:` to a task's `landed` field. That is how an agent records finished work.

An agent moves cards only into the columns of `agents.mayMoveTo`, never into a person's column. On a new board, Pinned and Done are the person's. The command does not refuse an agent that moves a card out of Pinned or Done. The agents' guide asks the agent to do that only when the person asked for it in the session.

### Several cards

Give several ids to move them together. Each card is moved on its own, with its own rules and events. The cards keep their order in the new column. A refusal of one card leaves the others moved, and the output lists each card.

> [!NOTE]
> **New to this?** On the page, you drag a card instead. The commands are for agents, and for scripts. See [Use the board](../use-the-board.md#move-a-card).

## Arguments

- **`<ID>...`**

  The card's number, or several numbers: `3 4 7`.

## Options

- **`--to <COLUMN>`**

  The column, by its id or its name, such as `for-review` or `"For review"`. Required.

- **`--as <PARTICIPANT>`**

  Who moves the card. Required.

- **`--comment <MARKDOWN>`**

  A comment written with the move, on each card.

- **`--if-updated <TIME>`**

  Refuses the move when the card's `updated` time is not this one. With one id only.

- **`--rewrite-paths`**

  Writes a path under your home folder in the comment as `<home>/…` instead of refusing it.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the card and its events. With several ids, prints `{ "results": [...] }`: each card's `outcome` (`changed`, `unchanged` or `refused`), and the refusal's words.

## Examples

- Record finished work and hand the card to the person, as an agent:

  ```console
  codelead board move 1 --to for-review --as claude --comment "Landed: 4f2a9c1 on main; webapp/Controllers/OrdersController.cs, webapp/Views/Orders/Create.cshtml (3 tests)."
  ```

  ```output
  Moved #1 from In progress to For review, with the comment.
  ```

- Answer and close a question, as the person:

  ```console
  codelead board move 2 --to done --as sam --comment "Answered: own orders only."
  ```

  ```output
  Moved #2 from Inbox to Done, with the comment.
  ```

- Pin a card that waits on you:

  ```console
  codelead board move 6 --to pinned --as sam
  ```

  ```output
  Moved #6 from Inbox to Pinned.
  ```

- Move several cards at once. One is already there:

  ```console
  codelead board move 1 2 --to done --as sam
  ```

  ```output
  Moved to Done (1 changed, 1 unchanged):
    #1 from For review
    #2 already in Done; nothing was written
  ```

- See an agent refused at a person's column:

  ```console
  codelead board move 2 --to pinned --as claude
  ```

  ```output
  Refused: claude is an agent and may not move a card into Pinned (a person's column); an agent may move a card into Inbox, Backlog, Ideas, In progress, For review. Nothing was written.
  ```

- Start the work on a pinned card, as an agent. Do this only when the person asked for the work in the session:

  ```console
  codelead board move 6 --to in-progress --as claude
  ```

  ```output
  Moved #6 from Pinned to In progress.
  ```

- Name a column that the board does not have, or give `--if-updated` with several ids:

  ```console
  codelead board move 2 --to later --as sam
  codelead board move 5 6 --to backlog --as sam --if-updated 2026-11-20T06:41:28Z
  ```

  ```output
  Refused: "later" is not a column of the board (inbox, backlog, ideas, pinned, in-progress, for-review, done). Nothing was written.
  --if-updated names one card's updated time; give it with one id, not several.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Moved, or already there. With several ids: no card was refused. |
| `1` | A rule of the board refused the move: a person's column, a column not on the board, a participant not on the board, a stale write. With several ids: at least one refusal was a rule's. |
| `2` | The command cannot run as written: no board, no such card, or a missing or wrong option. With several ids: every refusal was a missing card or a usage error. |

## Files

`move` changes each card's file in `items/` and appends `item.moved` to `events.jsonl`. With `--comment`, it also appends `comment.added`, and `item.updated` for a `Landed:` line on a task. See [Files and events](../files-and-events.md#every-event-kind).

## See also

- [Work with agents](../work-with-agents.md#it-records-its-work)
- [Rules for agents](../rules-for-agents.md#the-persons-columns)
- [`codelead board reply`](board-reply.md): a comment without a move.
- [`codelead board archive`](board-archive.md): take finished cards off the board.
- [Projects, themes and bulk operations](../projects-themes-and-bulk.md#change-several-cards-at-once)
- [Cards, columns and participants](../concepts.md#columns-and-whose-they-are)
