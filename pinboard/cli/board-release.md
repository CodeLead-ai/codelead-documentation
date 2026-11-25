---
title: codelead board release
description: The agent that holds a card from Ready gives it back to Ready, with the reason in its thread.
kind: reference
area: pinboard
date: 2026-11-25
---

# codelead board release

**Applies to:** CodeLead 0.1 and later

`codelead board release` gives a card back to **Ready**. The agent that holds the card uses it when it cannot finish, and says why.

## Name

`codelead board release` - Gives a held card back to Ready.

## Synopsis

```console
codelead board release <ID> --as <AGENT> --reason <MARKDOWN> [--rewrite-paths] [--board <DIR>] [--json]
codelead board release -h|--help
```

## Description

The reason lands in the card's thread, the card goes back to the top of Ready, and its claim ends. Another agent, or the same one, may then get it with [`codelead board take`](board-take.md).

Only the agent that holds the card gives it back. A person moves the card instead: a move by the person ends the claim too.

> [!NOTE]
> **New to this?** Release puts the card back where an agent can take it again. When a person should look at the card first, hand it to For review instead, with [`codelead board move`](board-move.md) and a comment. `codelead implement --ready` does that with every card it cannot finish, so that the card is not taken again in a loop.

## Arguments

- **`<ID>`**

  The card's number.

## Options

- **`--as <AGENT>`**

  The agent that holds the card. Required.

- **`--reason <MARKDOWN>`**

  Why the card goes back, for the person to read in the thread. Required.

- **`--rewrite-paths`**

  Writes a path under your home folder, outside the workspace, as `<home>/…` instead of refusing it.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the card, its comment and its events.

## Examples

These outputs come from a board in a scratch folder.

- Give the card back:

  ```console
  codelead board release 1 --as codelead --reason "The tests need a database this machine does not have."
  ```

  ```output
  Gave #1 back to Ready, with the reason in its thread.
  ```

- A person tries it:

  ```console
  codelead board release 1 --as sam --reason "x"
  ```

  ```output
  Refused: sam is a person; only the agent that holds a card gives it back. Move the card yourself: a move of yours ends its claim. Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Given back. |
| `1` | Refused: not the holder, a card held by no one, a reason with a secret, or `busy` (ask again). |
| `2` | No board, no such card, no `--as`, or an empty reason. |

## Files

`release` appends the reason to the card's thread, writes its column in `items/`, and appends `comment.added`, `item.moved` and `claim.released` to `events.jsonl`.

## See also

- [`codelead board take`](board-take.md), [`codelead board renew`](board-renew.md)
- [`codelead board move`](board-move.md): hand a card to For review.
