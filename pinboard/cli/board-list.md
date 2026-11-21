---
title: codelead board list
description: List the board's cards by column, with filters by column, tag, type and project, and the archived cards when asked.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board list

**Applies to:** CodeLead 0.1 and later

`codelead board list` prints the cards column by column. Filters keep every column and show how many cards match of the total.

## Name

`codelead board list` - Lists the cards by column.

## Synopsis

```console
codelead board list [--column <COLUMN>] [--tag <TAG>]... [--type <TYPE>] [--repo <NAME>] [--archived] [--board <DIR>] [--json]
codelead board list -h|--help
```

## Description

Each column is a line with its name and its count. Then comes one line for each card: its number, its type, its title and its tags. Archived cards are left out unless you give `--archived`.

With a filter, each count reads "matching of total", such as `Backlog (1 of 1)`. The columns stay in their order, so you see where the matching cards stand. With `--column`, only that column is printed.

`list` only reads. It writes nothing.

> [!NOTE]
> **New to this?** The page shows the same board, with more on each card. `list` is for an agent, a script, or a quick look in a terminal.

## Options

- **`--column <COLUMN>`**

  One column, by its id or its name, such as `backlog` or `"In progress"`.

- **`--tag <TAG>`**

  Cards with this tag. Repeat it to require every tag.

- **`--type <TYPE>`**

  Cards of this type: question, decision, task, idea, review or note.

- **`--repo <NAME>`**

  Cards tagged `repo:<NAME>`: one project.

- **`--archived`**

  Includes the archived cards.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints `{ "columns": [ { "id", "name", "total", "cards": [...] } ], "unreadable": [...] }`.

## Examples

- List the whole board:

  ```console
  codelead board list
  ```

  ```output
  Inbox (2)
    #3    decision Pick the date format of the orders list  [repo:webapp, by:claude]
    #6    question Should a deleted product keep its old orders?  [source:conversation, by:claude]
  Backlog (1)
    #5    task     Keep the product price on each order line  [area:orders, repo:webapp, source:conversation, by:claude]
  Ideas (1)
    #4    idea     Export the orders as CSV  [repo:store-api, by:codex]
  Pinned (0)
  In progress (0)
  For review (1)
    #1    task     Show the stock on the order form  [area:orders, repo:webapp, by:sam, for:codelead]
  Done (1)
    #2    question Must a client see the orders of other clients?  [area:orders, repo:webapp, by:claude]
  ```

- List the cards with a tag:

  ```console
  codelead board list --tag area:orders
  ```

  ```output
  Inbox (0 of 2)
  Backlog (1 of 1)
    #5    task     Keep the product price on each order line  [area:orders, repo:webapp, source:conversation, by:claude]
  Ideas (0 of 1)
  Pinned (0 of 0)
  In progress (0 of 0)
  For review (1 of 1)
    #1    task     Show the stock on the order form  [area:orders, repo:webapp, by:sam, for:codelead]
  Done (1 of 1)
    #2    question Must a client see the orders of other clients?  [area:orders, repo:webapp, by:claude]
  ```

- Combine a type and a project:

  ```console
  codelead board list --type question --repo webapp
  ```

  ```output
  Inbox (0 of 2)
  Backlog (0 of 1)
  Ideas (0 of 1)
  Pinned (0 of 0)
  In progress (0 of 0)
  For review (0 of 1)
  Done (1 of 1)
    #2    question Must a client see the orders of other clients?  [area:orders, repo:webapp, by:claude]
  ```

- Show one column:

  ```console
  codelead board list --column backlog
  ```

  ```output
  Backlog (1 of 1)
    #5    task     Keep the product price on each order line  [area:orders, repo:webapp, source:conversation, by:claude]
  ```

- Find archived cards in Done:

  ```console
  codelead board list --archived --column done
  ```

  ```output
  Done (2 of 2)
    #1    task     Show the stock on the order form  [area:orders, repo:webapp, by:sam, for:codelead]
    #2    question Must a client see the orders of other clients?  [area:orders, repo:webapp, by:claude]
  ```

- Name a column that the board does not have:

  ```console
  codelead board list --column later
  ```

  ```output
  Refused: "later" is not a column of the board (inbox, backlog, ideas, pinned, in-progress, for-review, done). Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The cards were listed. |
| `1` | The column or the type is not on the board. |
| `2` | No board. |

## Files

`list` reads `board.json` and the card files. A file in `items/` that cannot be read as a card is listed at the end, with its problems.

## See also

- [`codelead board show`](board-show.md): one card in full.
- [`codelead board check`](board-check.md): what changed.
- [Use the board](../use-the-board.md#filter-and-search): the same filters on the page.
- [Projects, themes and bulk operations](../projects-themes-and-bulk.md): projects as `repo:` tags.
- [Cards, columns and participants](../concepts.md)
