---
title: codelead board
description: Pinboard, the board between you and your coding agents, kept as files in the repository, and its subcommands.
kind: reference
area: cli
date: 2026-11-25
---

# codelead board

**Applies to:** CodeLead 0.1 and later

`codelead board` is Pinboard: the board between you and your coding agents. The board is a set of cards with threads, kept as files in `docs/codelead/board/` of the repository. Each subcommand has its own page in the [board commands](../pinboard/cli/index.md).

## Name

`codelead board` - Pinboard: the board between you and your coding agents.

## Synopsis

```console
codelead board <SUBCOMMAND> [<OPTIONS>]
codelead board -h|--help
```

## Description

A **card** is one piece of work. It has a title, a description, tags, a thread of comments, and a column: Inbox, Backlog, In progress, For review or Done. People and agents are **participants** of the board. Every change to a card is also an **event** in `events.jsonl`, and each participant has a **cursor**: how far it has read the events. See [Cards, columns and participants](../pinboard/concepts.md).

Most people use the board as a page, in [Visual Studio Code](../extension/board-view.md) or with [`codelead board serve`](../pinboard/cli/board-serve.md). The coding agents use the commands. See [Use the board](../pinboard/use-the-board.md) and [Rules for agents](../pinboard/rules-for-agents.md).

> [!NOTE]
> **New to this?** Because the board is files in the repository, it stays with the code. A card that asks for a change and the commit that makes it are in the same history.

### CodeLead on the board

A board that `codelead board init` makes has CodeLead on it from the start, as the participant `codelead`. Give a card to CodeLead with the tag `for:codelead`. [`codelead implement --queue`](codelead-implement.md) takes those cards, top of Backlog first.

Or move a card to **Ready**, once it says how to tell it is done, what it may touch and its limit. [`codelead implement --ready`](codelead-implement.md) asks the board for those cards with [`take`](../pinboard/cli/board-take.md), and works each on its own branch. A board made before Ready gets the column with [`codelead board add-ready`](../pinboard/cli/board-add-ready.md).

> [!TIP]
> **For experienced users:** the commands come from the package `@codelead/pinboard`, with their words, exit codes and `--json`. The package's own command, `pinboard`, runs the same tree. CodeLead adds its logger and the audit pack's sanitizer, so that a card follows the same rules as the audit pack.

## The subcommands

| Subcommand | What it does |
|---|---|
| [`init`](../pinboard/cli/board-init.md) | Creates the board folder in this repository: `board.json`, `AGENTS.md`, `events.jsonl`, `.gitignore`, `items/`, `cursors/`. |
| [`check`](../pinboard/cli/board-check.md) | What changed since your cursor: the events by others, grouped by card, what is addressed to you first. `--ack` moves the cursor to the end. |
| [`list`](../pinboard/cli/board-list.md) | The cards by column. Filters keep the columns, and show "matching of total". |
| [`show`](../pinboard/cli/board-show.md) | One card: its fields, its relations both ways, its attachments, its description and its thread. |
| [`new`](../pinboard/cli/board-new.md) | Creates a card, or every card of a markdown file (`--batch`), all or none. |
| [`reply`](../pinboard/cli/board-reply.md) | Adds a comment to a card's thread. |
| [`move`](../pinboard/cli/board-move.md) | Moves one card, or several, to a column. An agent moves only into the columns that it may use. |
| [`set`](../pinboard/cli/board-set.md) | Changes the title, description, type, tags, relations, attachments, status line, colour or fields of one card, or of several. |
| [`take`](../pinboard/cli/board-take.md) | An agent asks the board for its next card from Ready. The board gives one, held by a claim, or says why there is none. |
| [`renew`](../pinboard/cli/board-renew.md) | The agent that holds a card extends its claim while it works. |
| [`release`](../pinboard/cli/board-release.md) | The agent that holds a card gives it back to Ready, with the reason in its thread. |
| [`archive`](../pinboard/cli/board-archive.md) | Archives one card, or several (a person only). The card leaves the board and stays in its file. |
| [`unarchive`](../pinboard/cli/board-unarchive.md) | Brings archived cards back, into the column each was archived from (a person only). |
| [`validate`](../pinboard/cli/board-validate.md) | Checks every file of the board. Exit code 1, with the reasons, when something is wrong. |
| [`add-ready`](../pinboard/cli/board-add-ready.md) | Gives a board made before Ready its Ready column. No card moves. |
| [`import`](../pinboard/cli/board-import.md) | Turns `PINNED.md`, `DECISIONS-PENDING.md` and, with `--plan`, a plan's status table into cards, once. |
| [`tidy`](../pinboard/cli/board-tidy.md) | After an import: removes the title that an imported card's description repeats in bold at its start. |
| [`serve`](../pinboard/cli/board-serve.md) | Serves the board's page on this machine (`127.0.0.1` only), live as the files change. |

## Examples

- List the subcommands:

  ```console
  codelead board --help
  ```

  ```output
  Usage: codelead board [options] [command]

  Pinboard: the board between you and your coding agents, kept as files in
  docs/codelead/board (cards with threads, an event log, cursors)

  Options:
    -h, --help                    display help for command

  Commands:
    init [options]                Create the board folder in this repository:
                                  board.json, AGENTS.md, events.jsonl,
                                  .gitignore, items/, cursors/
    check [options]               What changed since your cursor: the events by
                                  others, grouped by card, what is addressed to
                                  you (an @mention of you) first, then the
                                  decisions recorded in a card's fields; --ack
                                  moves the cursor to the end
    list [options]                The cards by column; filters keep the columns
                                  and show matching of total
  …
  ```

- Make the board in a repository, then serve its page:

  ```console
  codelead board init
  codelead board serve
  ```

  <!-- described, not captured -->
  ```output
  Board created in docs/codelead/board: board.json, AGENTS.md, events.jsonl, .gitignore, items/, cursors/.
  Participants: … Agents read docs/codelead/board/AGENTS.md.

  Tell the agent, once, in the file it reads at the start of every session (CLAUDE.md, a root AGENTS.md or the like):
    …
  CodeLead joined the board as the participant codelead: tag a card for:codelead to hand it to CodeLead (codelead implement --queue takes those, top of Backlog first).
  ```

  `serve` then prints the page's address on `http://127.0.0.1:<port>/`. Open it in a browser.

  Each subcommand's page shows its exact output. See [Board commands](../pinboard/cli/index.md).

## Exit codes

The exit codes are each subcommand's. See its page. In general, `0` means done. `1` means that a rule of the board refused the write, or that `validate` found a problem. `2` means that the command cannot run as written. See [Board commands](../pinboard/cli/index.md#exit-codes).

## Files

| File | What it holds |
|---|---|
| `docs/codelead/board/board.json` | The board's columns, participants and settings. |
| `docs/codelead/board/items/` | One markdown file for each card. |
| `docs/codelead/board/events.jsonl` | Every change, one JSON line each. |
| `docs/codelead/board/cursors/` | Each participant's cursor. |
| `docs/codelead/board/AGENTS.md` | The board's rules for coding agents. |
| `docs/codelead/board/.gitignore` | Keeps the cursors, the lock, `private-names.txt` and `.local/` out of git. |

See [Files and events](../pinboard/files-and-events.md).

## See also

- [Pinboard overview](../pinboard/index.md)
- [Board commands](../pinboard/cli/index.md)
- [Work with agents](../pinboard/work-with-agents.md)
- [Tutorial: work with agents on the board](../tutorials/work-with-agents-on-the-board.md)
- [The Board view](../extension/board-view.md)
- [`codelead implement`](codelead-implement.md): CodeLead works the tagged cards.
