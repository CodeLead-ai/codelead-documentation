---
title: codelead board add-ready
description: Give a board made before the Ready column that column; no card moves.
kind: reference
area: pinboard
date: 2026-11-25
---

# codelead board add-ready

**Applies to:** CodeLead 0.1 and later

`codelead board add-ready` adds the column **Ready** to a board that was made before it. A board that `codelead board init` makes today has it already.

## Name

`codelead board add-ready` - Gives an older board its Ready column.

## Synopsis

```console
codelead board add-ready [--board <DIR>] [--json]
codelead board add-ready -h|--help
```

## Description

Ready is the person's column: you put a card there when an agent may pick it up alone. The card must say how to tell it is done, what it may touch, and its limit. See [Work with agents](../work-with-agents.md#hand-a-card-to-codelead-through-ready).

The command puts Ready between Pinned and In progress, and moves no card. It writes `AGENTS.md` again when the file is still the package's own text. When a person changed `AGENTS.md` by hand, the file is left as it is, and the command prints the lines to add.

A board that has Ready already is not written.

## Options

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints what was done: `added`, the column, its place, and what happened to `AGENTS.md`.

## Examples

These outputs come from a board in a scratch folder.

- Add the column:

  ```console
  codelead board add-ready
  ```

  ```output
  Added the column Ready to board.json, between Pinned and In progress. It is yours: put a card there that an agent may pick up by itself, once the card says how to tell it is done, what it may touch and its limit.
  AGENTS.md is written again, with the Ready rules.
  No card was moved.
  ```

- Run it again:

  ```console
  codelead board add-ready
  ```

  ```output
  The board has Ready already (the column "ready", between Pinned and In progress); nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The column was added, or the board had it already. |
| `1` | Refused: `board.json` has a column `ready` that is not the person's. |
| `2` | No board. |

## Files

`add-ready` writes `board.json`, and `AGENTS.md` when it is still the package's text.

## See also

- [`codelead board take`](board-take.md)
- [`codelead board init`](board-init.md)
- [`codelead implement --ready`](../../cli/codelead-implement.md)
