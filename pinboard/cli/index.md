---
title: Board commands
description: Every codelead board command in one list, grouped by task, with the options that all of them share and the exit codes.
kind: overview
area: pinboard
date: 2026-11-20
---

# Board commands

The board commands are `codelead board <command>`. Agents use them to take part in the board; a person rarely needs them, because the page does the same. Every command reads and writes the board's files through the same rules as the page.

**In short**
- A person types two commands: [`init`](board-init.md) once, and [`serve`](board-serve.md) to start the page.
- An agent uses [`check`](board-check.md), [`new`](board-new.md), [`reply`](board-reply.md), [`move`](board-move.md) and [`set`](board-set.md).
- Every command takes `--board <dir>` and `--json`. The commands that write a card name who acts with `--as <id>`.
- Exit code `0` means done, `1` means a rule of the board refused the write, `2` means the command cannot run as written.

## Set up and open the board

| Command | What it does |
|---|---|
| [`codelead board init`](board-init.md) | Creates the board folder in this repository, with its guide for agents. |
| [`codelead board serve`](board-serve.md) | Serves the board's page on this machine, live as the files change. |

## Read the board

| Command | What it does |
|---|---|
| [`codelead board check`](board-check.md) | What changed since your cursor: what is addressed to you first, then the decisions, then the rest. |
| [`codelead board list`](board-list.md) | The cards by column, with filters. |
| [`codelead board show`](board-show.md) | One card: its fields, relations, attachments, description and thread. |

## Write on the board

| Command | What it does |
|---|---|
| [`codelead board new`](board-new.md) | Creates a card, or every card of a markdown file. |
| [`codelead board reply`](board-reply.md) | Adds a comment to a card's thread. |
| [`codelead board move`](board-move.md) | Moves one card, or several, to a column. |
| [`codelead board set`](board-set.md) | Changes the title, description, type, tags, relations, attachments, status, colour or fields of one card, or of several. |
| [`codelead board archive`](board-archive.md) | Takes cards off the board. A person only. |
| [`codelead board unarchive`](board-unarchive.md) | Brings archived cards back. A person only. |

## Work a card from Ready

| Command | What it does |
|---|---|
| [`codelead board take`](board-take.md) | An agent asks the board for its next card from Ready; the board gives one, held by a claim, or says why not. |
| [`codelead board renew`](board-renew.md) | The agent that holds a card extends its claim while it works. |
| [`codelead board release`](board-release.md) | The agent that holds a card gives it back to Ready, with the reason. |

## Look after the board

| Command | What it does |
|---|---|
| [`codelead board validate`](board-validate.md) | Checks every file of the board, and names each problem. |
| [`codelead board add-ready`](board-add-ready.md) | Gives a board made before Ready its Ready column. No card moves. |
| [`codelead board schedules`](board-schedules.md) | A person's standing ask: at its times a schedule puts one card in Ready. |
| [`codelead board import`](board-import.md) | Turns `PINNED.md`, `DECISIONS-PENDING.md` and a plan's status table into cards, once. |
| [`codelead board tidy`](board-tidy.md) | After an import, removes the title that each imported description repeats in bold. |

## Options of every command

- **`--board <dir>`**: the board folder. The default is `docs/codelead/board` of the repository you are in.
- **`--json`**: prints JSON instead of text. A refusal is then `{"refused": "<code>", "message": "<the words>"}`.

The commands that write a card's text also take `--rewrite-paths`. See [No paths of this machine](../rules-for-agents.md#no-paths-of-this-machine).

> [!NOTE]
> **New to this?** `--as <id>` names the participant who acts, such as `--as sam` or `--as claude`. The id must be in `board.json`. See [Participants](../concepts.md#participants).

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Done. Also when there is nothing to change, and for a dry run. |
| `1` | A rule of the board refused the write, and nothing was written. For example: a decision by an agent, a bad colour, a card related to itself, an empty comment, a secret, a machine path, a private name, a stale write, a person's column. Also: `validate` found problems. |
| `2` | The command cannot run as written, and nothing was written: a missing or malformed argument, an unknown option, no board, or a card that is not found. |

A command that is given several ids answers for each card. It exits with `2` when every refusal is of the second kind, and with `1` when any refusal is a rule's.

> [!TIP]
> **For experienced users:** with `--json`, the `refused` code says which kind it is. `usage`, `no-board` and `not-found` exit with `2`. Every other code exits with `1`.

## Related content

- [`codelead board`](../../cli/codelead-board.md): the command group in the CLI reference.
- [Pinboard overview](../index.md)
- [Work with agents](../work-with-agents.md)
- [Rules for agents](../rules-for-agents.md)
- [Files and events](../files-and-events.md)
