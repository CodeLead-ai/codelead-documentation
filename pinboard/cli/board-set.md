---
title: codelead board set
description: Change the title, description, type, tags, relations, attachments, status line, colour or fields of one card, or the same change on several cards, each change with its event.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board set

**Applies to:** CodeLead 0.1 and later

`codelead board set` changes a card. It changes the title, the description, the type, the tags, the relations, the attachments, the status line, the colour mark or the fields. Give several ids to make the same change on each card.

## Name

`codelead board set` - Changes the title, description, type, tags, relations, attachments, status, colour or fields of cards.

## Synopsis

```console
codelead board set <ID>... --as <PARTICIPANT> [--title <TEXT>] [--body <MARKDOWN> | --body-file <PATH>] [--type <TYPE>] [--tag <TAG>]... [--untag <TAG>]... [--relate <KIND:ID>]... [--unrelate <KIND:ID>]... [--attach <PATH[:LINES]>]... [--status <TEXT>] [--color <HEX|NAME|none>] [--field <NAME=VALUE>]... [--unset-field <NAME>]... [--if-updated <TIME>] [--rewrite-paths] [--board <DIR>] [--json]
codelead board set -h|--help
```

## Description

Each tag, each relation and each attachment gets its own event. The other changes share one `item.updated` event that names them. A change that would leave the card as it is writes nothing, and says so.

The person records a decision with `--field decided=<option>`. The board adds who decided and the day, and every agent's next check lists it under "Decided". An agent cannot set, change or remove `decided`: it writes its recommendation with `--field ask="…"`.

`--field` takes only the fields of the card's type, so a typo is refused. A value that reads as a JSON list or object is one. `--unset-field` removes a field, also one that the card keeps from another type.

> [!NOTE]
> **New to this?** `set` is the same edit as the card's panel on the page. Two things are the page's alone: a card's place inside its column, and the view choices. See [Use the board](../use-the-board.md#read-and-answer-a-card).

### Title, description and type

`--title`, `--body` and `--body-file` give one card its own text, so they take one id only. The title is one line. An empty `--body ""` empties the description.

`--type` changes the card to another type of the board: question, decision, task, idea, review or note. The card keeps the fields of its old type, and `--unset-field` removes them. With `--type`, `--field` takes the fields of the new type.

### Attachments

`--attach` adds a path to the card's attachments, with lines when needed: `webapp/Controllers/OrdersController.cs:10-40`. A file of another repository of the workspace is `<repository>:<path>`. The board does not check that the file exists. A path that is attached already is not added again.

### Colours

The colour mark is the person's own coding, and only a person sets it. The five colours are `b33f40` red, `ed944d` orange, `edd94c` yellow, `48a14d` green and `6577b3` blue. Give the hex, with or without `#`, or the name. `none` removes the mark. Quote a value that starts with `#`, because the shell reads `#` as the start of a comment.

### What an agent cannot change

The commands refuse three changes by an agent, and write nothing: a decision (`decided`), the colour mark, and adding the tag `for:codelead`. The person tags a card `for:codelead` to give it to CodeLead. An agent asks for that in the card's thread. See [Rules for agents](../rules-for-agents.md).

## Arguments

- **`<ID>...`**

  The card's number, or several numbers: `3 4 7`.

## Options

- **`--as <PARTICIPANT>`**

  Who changes the card. Required.

- **`--title <TEXT>`**

  The card's new title, on one line. With one id only.

- **`--body <MARKDOWN>`**

  The card's new description. An empty text empties it. With one id only.

- **`--body-file <PATH>`**

  The card's new description, read from a file. With one id only. It cannot go with `--body`.

- **`--type <TYPE>`**

  The card's new type, one of the board's types. The card keeps the fields of its old type.

- **`--tag <TAG>`**

  Adds a tag. Repeat it for several. An agent cannot add `for:codelead`.

- **`--untag <TAG>`**

  Removes a tag. Repeat it for several.

- **`--relate <KIND:ID>`**

  Adds a relation, such as `related:12`. Repeat it for several.

- **`--unrelate <KIND:ID>`**

  Removes a relation. Repeat it for several.

- **`--attach <PATH[:LINES]>`**

  Attaches a file, such as `docs/plan.md:40-60` or `<repository>:<path>`. Repeat it for several.

- **`--status <TEXT>`**

  The one-line status beside the column. An empty text removes it.

- **`--color <HEX|NAME|none>`**

  The person's colour mark, or `none`. An agent cannot set it.

- **`--field <NAME=VALUE>`**

  Sets a field of the card's type, such as `ask="…"` on a decision. Repeat it for several.

- **`--unset-field <NAME>`**

  Removes a field. Repeat it for several.

- **`--if-updated <TIME>`**

  Refuses the change when the card's `updated` time is not this one. With one id only.

- **`--rewrite-paths`**

  Writes a path under your home folder as `<home>/…` instead of refusing it. It applies to every text of the change, an attachment included.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the card and its events. With several ids, prints each card's result.

## Examples

- Hand a card to CodeLead:

  ```console
  codelead board set 1 --as sam --tag for:codelead
  ```

  ```output
  #1: tagged it for:codelead.
  ```

- Record a decision as the person:

  ```console
  codelead board set 3 --as sam --field decided="2026-11-20"
  ```

  ```output
  #3: decided: 2026-11-20 (by sam, 2026-11-20).
  ```

- Change the ask of a decision, as an agent:

  ```console
  codelead board set 3 --as claude --field ask="Show dates as 2026-11-20 or as 20 Nov 2026? I recommend 2026-11-20."
  ```

  ```output
  #3: changed its ask.
  ```

- Change a card's title, and its description from a file:

  ```console
  codelead board set 1 --as sam --title "Show the stock on the order form and the product page"
  codelead board set 5 --as claude --body-file body.md
  ```

  ```output
  #1: changed its title.
  #5: changed its description.
  ```

- Turn an idea into a task. The card keeps the fields of the idea:

  ```console
  codelead board set 7 --as sam --type task
  ```

  ```output
  #7: changed its type.
  ```

- Attach the files that a card is about, one with its lines:

  ```console
  codelead board set 1 --as claude --attach webapp/Controllers/OrdersController.cs:10-40 --attach webapp/Views/Home/Index.cshtml
  ```

  ```output
  #1: attached webapp/Controllers/OrdersController.cs:10-40; attached webapp/Views/Home/Index.cshtml.
  ```

- Set a status line, and a task's fields:

  ```console
  codelead board set 1 --as claude --status "the stock query works; the view is next"
  codelead board set 5 --as sam --field definitionOfDone="An old order keeps its price after a price change." --field size=M
  ```

  ```output
  #1: changed its status.
  #5: changed its fields.
  ```

- Add a tag to two cards, and relate one card to another:

  ```console
  codelead board set 5 6 --as sam --tag area:catalog
  codelead board set 4 --as sam --relate related:5
  ```

  ```output
  Changed (2 changed):
    #5 tagged it area:catalog
    #6 tagged it area:catalog
  #4: related it: related #5.
  ```

- Mark a card red, then try a colour that the board does not have:

  ```console
  codelead board set 6 --as sam --color "#b33f40"
  codelead board set 6 --as sam --color purple
  ```

  ```output
  #6: changed its colour.
  Refused: "purple" is not a card colour; the colours are #b33f40 (red), #ed944d (orange), #edd94c (yellow), #48a14d (green) and #6577b3 (blue), or none. Nothing was written.
  ```

- See the refusals for a decision by an agent, and for a field of another type:

  ```console
  codelead board set 3 --as claude --field decided="2026-11-20"
  codelead board set 4 --as sam --field size=M
  ```

  ```output
  Refused: claude is an agent; the decision is the person's: an agent writes its recommendation in fields.ask. Nothing was written.
  "size" is not a field of an idea (hypothesis, worthItIf); change the card's type first if it should have it.
  ```

- See an agent refused at the tag `for:codelead` and at the colour. Both exit with code 1:

  ```console
  codelead board set 1 --as claude --tag for:codelead
  codelead board set 6 --as claude --color red
  ```

  ```output
  Refused: claude is an agent and may not tag a card for:codelead: the person queues a card for CodeLead; ask in the thread. Nothing was written.
  Refused: claude is an agent and may not set a card's colour: the colour is the person's own mark. Nothing was written.
  ```

- Give a title to two cards, or forget to say what to change. Both exit with code 2:

  ```console
  codelead board set 1 4 --as sam --title "Stock"
  codelead board set 5 --as sam
  ```

  ```output
  --title gives one card its title; give it with one id, not several.
  set needs something to change: --title, --body, --body-file, --type, --tag, --untag, --relate, --unrelate, --attach, --status, --color, --field or --unset-field.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Changed, or nothing to change. |
| `1` | A rule of the board refused it: a decision, the colour or the tag `for:codelead` by an agent, a bad colour, a type that the board does not have, a card related to itself, a title of two lines, a participant not on the board, a secret, a path, a private name, a stale write. |
| `2` | The command cannot run as written: no board, no such card, nothing to change given, a field that the type does not have, `--title` or a description with several ids, `--body` with `--body-file`, or an unknown option. With several ids: every refusal was of this kind. |

## Files

`set` changes each card's file in `items/` and appends `tag.added`, `tag.removed`, `relation.added`, `relation.removed`, `attachment.added` or `item.updated` to `events.jsonl`. See [Files and events](../files-and-events.md#every-event-kind).

## See also

- [Record a decision](../use-the-board.md#record-a-decision) on the page.
- [Rules for agents](../rules-for-agents.md#a-decision-is-the-persons)
- [Work with agents](../work-with-agents.md#give-a-card-to-codelead): the tag `for:codelead`.
- [`codelead board move`](board-move.md) and [`codelead board new`](board-new.md)
- [Cards, columns and participants](../concepts.md#decisions-and-the-ask)
