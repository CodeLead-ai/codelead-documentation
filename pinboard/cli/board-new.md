---
title: codelead board new
description: Create a card with its event, or every card of a markdown file at once, all or none, with a dry run that shows the ids first.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board new

**Applies to:** CodeLead 0.1 and later

`codelead board new` creates a card. With `--batch`, it creates every card of a markdown file in one step: all of them, or none.

## Name

`codelead board new` - Creates a card, or the cards of a file.

## Synopsis

```console
codelead board new --as <PARTICIPANT> --type <TYPE> --title <TEXT> [--column <COLUMN>] [--tag <TAG>]... [--body <MARKDOWN> | --body-file <PATH>] [--relate <KIND:ID>]... [--field <NAME=VALUE>]... [--session <ID>] [--dry-run] [--rewrite-paths] [--board <DIR>] [--json]
codelead board new --as <PARTICIPANT> --batch <FILE> [--column <COLUMN>] [--tag <TAG>]... [--session <ID>] [--dry-run] [--rewrite-paths] [--board <DIR>] [--json]
codelead board new -h|--help
```

## Description

The card gets the next id from `nextId` in `board.json`, and `nextId` goes up in the same step. The card lands in the first column, Inbox, unless `--column` names another. The card is tagged `by:<participant>`, and its `item.created` event is written to the log.

An agent creates cards only in the columns of `agents.mayMoveTo`. It never creates a card with a decision, or with the tag `for:codelead`. A description that names someone with `@<id>` addresses them: their next check lists the card first.

> [!NOTE]
> **New to this?** On the page, press `n` to write a card. The page does not tag your own cards `by:<you>`; that tag marks an agent's proposals. See [Use the board](../use-the-board.md#write-a-card).

### Several cards from one file

`--batch <FILE>` reads a markdown file. Each card is a heading `## <title>`, then a fenced `yaml` block, then its description. The block takes `type` and, when needed, `key`, `column`, `tags`, `fields`, `relations` and `session`:

````markdown
## Keep the product price on each order line

```yaml
key: price
type: task
column: backlog
tags: [area:orders, repo:webapp]
relations:
  - { kind: related, to: 1 }
```

Today an order line reads the product's current price.

## Should a deleted product keep its old orders?

```yaml
key: deleted
type: question
relations:
  - { kind: related, to: "@price" }
```

A product can be deleted from the Products page.
````

A relation names a card on the board by its number, or a card of the same file by its `key` with `@`. Before the first card, only a `#` title and blank lines may stand. Every card is checked before any is written. One bad card refuses the file, and the refusal names its heading.

With `--batch`, the command's `--column`, `--tag` and `--session` apply to every card that does not give its own. The options of one card, such as `--type` and `--title`, are refused.

### Fields

`--field name=value` sets a field of the card's type, such as a decision's `ask`. A value that reads as a JSON list or object is one: `--field options='["a", "b"]'`. A name that is not a field of the type is refused, so a typo never becomes a new field. The fields of each type are in [Cards and their types](../concepts.md#cards-and-their-types).

## Options

- **`--as <PARTICIPANT>`**

  Who creates the card. Required.

- **`--type <TYPE>`**

  The card's type: question, decision, task, idea, review or note. Required for one card.

- **`--title <TEXT>`**

  The title, on one line. Required for one card.

- **`--column <COLUMN>`**

  Where the card lands, by id or name. The default is the first column, Inbox.

- **`--tag <TAG>`**

  A tag. Repeat it for several tags. An agent cannot add `for:codelead`.

- **`--body <MARKDOWN>`**

  The description.

- **`--body-file <PATH>`**

  The description, read from a file. It cannot go with `--body`.

- **`--relate <KIND:ID>`**

  A relation, such as `child-of:7` or `related:5`. Repeat it for several. The kinds are `parent-of`, `child-of`, `related`, `blocks`, `blocked-by`, `duplicate-of` and `follows`.

- **`--session <ID>`**

  The agent session that the card came from.

- **`--field <NAME=VALUE>`**

  A field of the type, such as `ask="…"` on a decision: what is decided, the options and the recommendation. Repeat it for several fields.

- **`--batch <FILE>`**

  Creates every card of a markdown file, all or none.

- **`--dry-run`**

  Prints what would be created, with the ids, and writes nothing.

- **`--rewrite-paths`**

  Writes a path under your home folder, outside the workspace, as `<home>/…` instead of refusing it. A path inside the repository or the workspace is always written as a card cites it.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the card and its events. With `--batch`, prints every card and event, and `dryRun`.

## Examples

- Create a task as the person:

  ```console
  codelead board new --as sam --type task --title "Show the stock on the order form" --body "When a clerk adds a product to an order, the form must show how many are in stock. See webapp/Controllers/OrdersController.cs." --tag area:orders --tag repo:webapp
  ```

  ```output
  Created #1 (task) in Inbox: Show the stock on the order form
  docs/codelead/board/items/0001-show-the-stock-on-the-order-form.md
  ```

- Propose a decision as an agent, with its ask and its options:

  ```console
  codelead board new --as claude --type decision --title "Pick the date format of the orders list" --field ask="Show dates as 2026-11-20 or as 20 Nov 2026? I recommend 2026-11-20: it sorts and it reads the same in every language." --field options='["2026-11-20", "20 Nov 2026"]' --tag repo:webapp
  ```

  ```output
  Created #3 (decision) in Inbox: Pick the date format of the orders list
  docs/codelead/board/items/0003-pick-the-date-format-of-the-orders-list.md
  ```

- Read the description from a file, land in Backlog, relate to card 5 and record the session:

  ```console
  codelead board new --as codex --type task --title "Mark products that are out of stock" --body-file body.md --column Backlog --relate related:5 --session codex-2026-11-20-a --tag repo:webapp
  ```

  ```output
  Created #7 (task) in Backlog: Mark products that are out of stock
  docs/codelead/board/items/0007-mark-products-that-are-out-of-stock.md
  ```

- See the id that a card would get, and write nothing:

  ```console
  codelead board new --as sam --type note --title "Tests run on Windows only" --dry-run
  ```

  ```output
  Would create #7 (note) in Inbox: Tests run on Windows only; nothing was written.
  ```

- Turn a conversation into cards. Check first, then create:

  ```console
  codelead board new --as claude --batch cards.md --tag source:conversation --dry-run
  codelead board new --as claude --batch cards.md --tag source:conversation
  ```

  ```output
  Would create 2 cards from cards.md; nothing was written:
    #5 (task) in Backlog: Keep the product price on each order line  [area:orders, repo:webapp, source:conversation, by:claude]
        related #1
    #6 (question) in Inbox: Should a deleted product keep its old orders?  [source:conversation, by:claude]
        related #5
  Run it again without --dry-run to create them.
  Created 2 cards from cards.md:
    #5 (task) in Backlog: Keep the product price on each order line  [area:orders, repo:webapp, source:conversation, by:claude]
        related #1
    #6 (question) in Inbox: Should a deleted product keep its old orders?  [source:conversation, by:claude]
        related #5
  ```

- See the refusals an agent can meet:

  ```console
  codelead board new --as claude --type task --title "Fix the price rounding" --column pinned
  codelead board new --as claude --type decision --title "Pick a cache" --field decided="TTL"
  codelead board new --as claude --type task --title "Fix the price rounding" --tag for:codelead
  ```

  ```output
  Refused: claude is an agent and may not create a card in Pinned (a person's column); an agent may create a card in Inbox, Backlog, Ideas, In progress, For review. Nothing was written.
  Refused: claude is an agent; the decision is the person's: an agent writes its recommendation in fields.ask. Nothing was written.
  Refused: claude is an agent and may not tag a card for:codelead: the person queues a card for CodeLead; ask in the thread. Nothing was written.
  ```

  A batch file whose card has the tag `for:codelead` gets the same refusal, with the card's heading first.

- See the usage errors:

  ```console
  codelead board new --as claude --type task --title "Fix the price rounding" --field ask="x"
  codelead board new --as claude --batch cards.md --title x
  ```

  ```output
  "ask" is not a field of a task (definitionOfDone, size, landed); change the card's type first if it should have it.
  new --batch takes each card's type, title, description, relations and fields from the file, so --title goes with a card made alone; --column, --tag and --session apply to every card of the batch.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The card or cards were created, or a dry run was printed. |
| `1` | A rule of the board refused the card: the column, the type, the participant, a decision or the tag `for:codelead` by an agent, a secret, a machine path or a private name. Nothing was written. |
| `2` | The command cannot run as written: no board, a missing option, a field that the type does not have, or an option that does not go with `--batch`. |

## Files

`new` writes `items/<NNNN>-<slug>.md`, raises `nextId` in `board.json`, and appends `item.created` to `events.jsonl`. A batch also appends one `relation.added` for each relation. See [Files and events](../files-and-events.md).

## See also

- [Work with agents](../work-with-agents.md#turn-a-conversation-into-cards)
- [`codelead board set`](board-set.md): change a card after it exists.
- [`codelead board reply`](board-reply.md): add to its thread.
- [Rules for agents](../rules-for-agents.md)
- [Cards, columns and participants](../concepts.md)
