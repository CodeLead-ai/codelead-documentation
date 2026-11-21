---
title: codelead board show
description: Print one card in full - its fields, its relations both ways, its attachments, its description and its thread - and optionally mark it as seen.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board show

**Applies to:** CodeLead 0.1 and later

`codelead board show` prints one card: its fields, its relations in both directions, its attachments, its description and its thread. An agent reads a card with it right before it changes the card.

## Name

`codelead board show` - Prints one card.

## Synopsis

```console
codelead board show <ID> [--as <PARTICIPANT>] [--board <DIR>] [--json]
codelead board show -h|--help
```

## Description

The first line gives the number, the type, the column, and the archive time when the card is archived. Then come the title, the tags, the author and times, the file, and the optional parts: status, colour, session, relations, attachments and fields. The description and the thread end the output.

The relations include those that other cards state toward this card, marked `(stated by #N)`. Each related card shows its column and its title.

With `--as`, `show` records that the participant opened the card. The card's events up to now count as seen, so the next check of that participant leaves them out. Without `--as`, `show` writes nothing.

A card whose file has problems is still shown, with a `Problems:` line. A write to that card is refused until the file is fixed.

## Arguments

- **`<ID>`**

  The card's number, such as `42` or `#42`.

## Options

- **`--as <PARTICIPANT>`**

  Records that this participant opened the card: its events up to now are seen.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints `{ "card", "relations", "problems" }`. The card includes its `file` and a `hash` of its text.

## Examples

- Show a card with relations in both directions:

  ```console
  codelead board show 5
  ```

  ```output
  #5 · task · Backlog
  Keep the product price on each order line
  tags: area:orders, repo:webapp, source:conversation, by:claude, area:catalog
  by claude · created 2026-11-20T06:40:46Z · updated 2026-11-20T06:41:04Z
  file: items/0005-keep-the-product-price-on-each-order.md
  relations:
    related #1 (Done) Show the stock on the order form
    related #4 (Ideas) Export the orders as CSV  (stated by #4)
    related #6 (Pinned) Should a deleted product keep its old orders?  (stated by #6)

  Today an order line reads the product's current price. A price change then changes old orders. Store the price on the line when the order is placed. Came out of the stock work on card 1.

  Thread (0)
  ```

- Show an archived task with its status and what landed:

  ```console
  codelead board show 1
  ```

  ```output
  #1 · task · Done · archived 2026-11-20T06:41:09Z
  Show the stock on the order form
  tags: area:orders, repo:webapp, by:sam, for:codelead
  by sam · created 2026-11-20T06:39:23Z · updated 2026-11-20T06:41:09Z
  file: items/0001-show-the-stock-on-the-order-form.md
  status: the stock query works; the view is next
  relations:
    related #5 (Backlog) Keep the product price on each order line  (stated by #5)
  fields:
    landed: ["4f2a9c1 on main; webapp/Controllers/OrdersController.cs, webapp/Views/Orders/Create.cshtml (3 tests)."]
  …
  ```

- Show a decision with its ask and its answer:

  ```console
  codelead board show 3
  ```

  ```output
  #3 · decision · Inbox
  Pick the date format of the orders list
  tags: repo:webapp, by:claude
  by claude · created 2026-11-20T06:39:25Z · updated 2026-11-20T06:40:19Z
  file: items/0003-pick-the-date-format-of-the-orders-list.md
  fields:
    ask: Show dates as 2026-11-20 or as 20 Nov 2026? I recommend 2026-11-20: it sorts and it reads the same in every language.
    options: ["2026-11-20","20 Nov 2026"]
    decided: {"option":"2026-11-20","by":"sam","on":"2026-11-20"}

  Thread (0)
  ```

- Read a card and mark it as seen for the person:

  ```console
  codelead board show 6 --as sam
  ```

  ```output
  #6 · question · Pinned
  Should a deleted product keep its old orders?
  tags: source:conversation, by:claude, area:catalog
  by claude · created 2026-11-20T06:40:46Z · updated 2026-11-20T06:41:43Z
  file: items/0006-should-a-deleted-product-keep-its-old.md
  color: #b33f40 (red)
  …
  ```

- Name a card that is not on the board:

  ```console
  codelead board show 42
  ```

  ```output
  No card #42 on the board.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The card was printed. |
| `1` | The participant of `--as` is not on the board, or two cards share the id. |
| `2` | No board, no such card, or an id that is not a card id. |

## Files

`show` reads `board.json` and the card files. With `--as`, it writes `cursors/<participant>.json` (the card's `opened` time).

## See also

- [`codelead board list`](board-list.md): every card, in short.
- [`codelead board check`](board-check.md): what changed.
- [`codelead board reply`](board-reply.md) and [`codelead board set`](board-set.md): change the card you read.
- [Files and events](../files-and-events.md#a-card-itemsnnnn-slugmd)
- [Cards, columns and participants](../concepts.md)
