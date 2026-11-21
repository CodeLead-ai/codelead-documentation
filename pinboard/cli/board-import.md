---
title: codelead board import
description: Turn a project's PINNED.md, DECISIONS-PENDING.md and, with --plan, a plan's status table into cards, once, and print the mapping to check.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board import

**Applies to:** CodeLead 0.1 and later

`codelead board import` turns three kinds of project notes into cards: a `PINNED.md` list, a `DECISIONS-PENDING.md` list and, when you name it, a plan's status table. It runs once for each document, and prints a mapping of every item to its card, or to the reason it was skipped.

## Name

`codelead board import` - Turns a project's notes into cards, once.

## Synopsis

```console
codelead board import --from <FOLDER> [--plan <FILE>] [--section <N>] [--as <PERSON>] [--repo <NAME>] [--workspace <FOLDER>] [--dry-run] [--board <DIR>] [--json]
codelead board import -h|--help
```

## Description

Use `import` when a project kept its open questions and decisions in markdown files before it had a board. Run it with `--dry-run` first, read the mapping, then run it without `--dry-run`. The documents are only read; they do not change.

> [!NOTE]
> **New to this?** Most boards never need `import`. A new project starts with an empty board and writes cards on the page.

### What becomes a card

- **`PINNED.md`**: every list item becomes a card. Its section gives the type: "For action" a task, "For decision" a decision, "For review" a review, all in Pinned. "Answered" gives a decision in Done. An item named by a key, such as `0r — …`, is tagged `pinned:0r`.
- **`DECISIONS-PENDING.md`**: an item becomes a decision card in Pinned when it is open and `PINNED.md` does not already have it. Under a "Parked" heading, the card goes to Backlog. An item is closed when it is struck through, answered in place ("Answer:") or says "nothing open". It is also closed when it starts with a closing word, such as done, decided or shipped. An item is open when it says that something waits, such as "needs <name>", "<name>'s call", "your call", "undecided" or "open question". Every other item stays in the document, and the mapping says why.
- **The plan's status table**, only when `--plan` names the plan: each row becomes a task card, tagged `plan:<item>`. Its column comes from the first word of the status. "Done" gives Done. "After", "Later", "Not", "Planned", "Parked", "Waiting", "Blocked" and "Queued" give Backlog. Anything else gives In progress.

Every card is tagged `repo:<repository>`. Its first attachment is its source document and lines. Other paths that the item names, and that exist, become attachments too. A card relates to every other imported card whose key it names.

### Once for each document

The import marks each document in the `imports` list of `board.json`, with its hash and its cards. A second import of the same document is refused. A path of this machine in a document becomes its path in the repository. Anything that the sanitizer would remove is replaced, and the mapping says so.

## Options

- **`--from <FOLDER>`**

  The folder that holds `PINNED.md` and `DECISIONS-PENDING.md`, usually `docs`. Required.

- **`--plan <FILE>`**

  A plan whose status table becomes task cards. It has no default: without `--plan`, no plan is read, and the mapping's last line says so. A plan that you name and that is not there is skipped, and the mapping says so.

- **`--section <N>`**

  The plan's section that holds the status table. The default is `8`.

- **`--as <PERSON>`**

  Who imports: a person of the board. The default is the board's first person.

- **`--repo <NAME>`**

  The repository's name for the `repo:` tag. The default is the repository folder's name.

- **`--workspace <FOLDER>`**

  The folder of the workspace's repositories, for paths of other repositories. The default is the repository's parent.

- **`--dry-run`**

  Prints the mapping, with the ids the cards would get, and writes nothing.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the result: `from`, `as`, `items`, `sources`, `planNamed` and `written`. `planNamed` is `true` when `--plan` named a plan.

## Examples

- See the mapping first, with a plan of your own and its section 3:

  ```console
  codelead board import --from docs --plan docs/store-plan.md --section 3 --dry-run
  ```

  ```output
  Import from docs as sam: 5 cards from 3 documents; 1 item skipped. Dry run: nothing was written, and the ids are the ones the cards would get.

  docs/PINNED.md: 2 items, 2 cards
    line 5     → #8    task     Pinned      Give the test database a second client
    line 9     → #9    decision Pinned      0a: Keep the Basic Store on .NET Framework 4.5 for one more release?

  docs/DECISIONS-PENDING.md: 2 items, 1 card, 1 skipped
    line 3     → #10   decision Pinned      Which payment provider for the new checkout?
               open: it says "Sam's call"
    line 4     skipped: closed: "Done" in its first words · The product images move to the database

  docs/store-plan.md §3: 2 items, 2 cards
    line 7     → #11   task     Done        P1. Orders list
    line 8     → #12   task     Backlog     P2. Stock
  ```

- Run it for real. The output is the same mapping, without the dry-run sentence:

  ```console
  codelead board import --from docs --plan docs/store-plan.md --section 3
  ```

  ```output
  Import from docs as sam: 5 cards from 3 documents; 1 item skipped.
  …
  ```

- Read an imported card. Its description still starts with its title in bold; [`tidy`](board-tidy.md) removes that:

  ```console
  codelead board show 9
  ```

  ```output
  #9 · decision · Pinned
  0a: Keep the Basic Store on .NET Framework 4.5 for one more release?
  tags: repo:store
  by sam · created 2026-11-20T06:43:24Z · updated 2026-11-20T06:43:24Z
  file: items/0009-0a-keep-the-basic-store-on-net-framework.md
  attachments:
    docs/PINNED.md:9

  **0a: Keep the Basic Store on .NET Framework 4.5 for one more release?** The new API waits on this answer.

  Thread (0)
  ```

- Run it a second time. It is refused:

  ```console
  codelead board import --from docs --plan docs/store-plan.md --section 3
  ```

  ```output
  Refused: docs/PINNED.md was imported on 2026-11-20T06:43:24Z (board.json "imports"); the import runs once per document. Nothing was written.
  ```

- Import into another board, with another `repo:` tag. Without `--plan`, no plan is read, and the last line says so:

  ```console
  codelead board import --from docs --board other-board --repo webapp --workspace .. --dry-run
  ```

  ```output
  Import from docs as sam: 3 cards from 2 documents; 1 item skipped. Dry run: nothing was written, and the ids are the ones the cards would get.
  …
  No plan was named, so no status table was read: give --plan <file> to import one.
  ```

- Name a plan that is not there. The mapping skips it:

  ```console
  codelead board import --from docs --plan docs/missing-plan.md --dry-run
  ```

  ```output
  …
  docs/missing-plan.md: not found, nothing imported from it
  ```

- Forget `--from`, or name a folder that is not there:

  ```console
  codelead board import
  codelead board import --from notes
  ```

  ```output
  import needs --from <folder>: the folder that holds PINNED.md and DECISIONS-PENDING.md (docs, say); a plan is named with --plan <file>.
  No folder notes.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The cards were created, or a dry run was printed. |
| `1` | Refused: a document was imported already, or a card would break a rule. Nothing was written. |
| `2` | No board, no `--from`, or a folder that is not there. |

## Files

`import` reads the documents in `--from`, and the plan that `--plan` names. It writes one card file in `items/` for each card, appends `item.created` events with `data.source`, and adds a marker for each document to `imports` in `board.json`. See [Files and events](../files-and-events.md#imports).

## See also

- [`codelead board tidy`](board-tidy.md): clean the imported descriptions.
- [`codelead board validate`](board-validate.md): check the board after the import.
- [`codelead board list`](board-list.md): see where the cards landed.
- [Files and events](../files-and-events.md)
- [Cards, columns and participants](../concepts.md)
