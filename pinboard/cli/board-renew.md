---
title: codelead board renew
description: The agent that holds a card from Ready extends its claim while it works, so the card is not given back to Ready.
kind: reference
area: pinboard
date: 2026-11-25
---

# codelead board renew

**Applies to:** CodeLead 0.1 and later

`codelead board renew` extends the claim on a card that the agent holds. The claim then runs for its lease again, from now.

## Name

`codelead board renew` - Extends the agent's claim on a card.

## Synopsis

```console
codelead board renew <ID> --as <AGENT> [--lease <MINUTES>] [--board <DIR>] [--json]
codelead board renew -h|--help
```

## Description

A card that [`codelead board take`](board-take.md) gave an agent is held by a claim that runs out at a time. While the agent works, it renews the claim, well before that time. When the claim runs out, the next take puts the card back at the top of Ready.

Only the agent that holds the card renews it. A claim ends when the card leaves In progress. The agent hands it to For review or gives it back with [`release`](board-release.md), or the person moves it. A renewal after that is refused, and the refusal says where the card is now.

> [!TIP]
> **For experienced users:** a refused renewal is the signal that the person's move won. `codelead implement --ready` stops the card's session when that happens, and writes why in the card's thread.

## Arguments

- **`<ID>`**

  The card's number.

## Options

- **`--as <AGENT>`**

  The agent that holds the card. Required.

- **`--lease <MINUTES>`**

  How long from now. The default is the board's `dispatch.leaseMinutes` (30). The most is `dispatch.maxLeaseMinutes` (240).

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the card, the new `claim`, the minutes and the event.

## Examples

These outputs come from a board in a scratch folder.

- Renew for 40 minutes:

  ```console
  codelead board renew 1 --as codelead --lease 40
  ```

  ```output
  #1: codelead holds it until 2026-11-25T12:49:19Z (40 minutes).
  ```

- Another agent tries to renew it:

  ```console
  codelead board renew 1 --as claude
  ```

  ```output
  Refused: card #1 is held by codelead, not by claude; only the agent that holds a card renews it or gives it back. Nothing was written.
  ```

- Ask for a lease that is longer than the board allows:

  ```console
  codelead board renew 1 --as codelead --lease 500
  ```

  ```output
  Refused: a lease on this board is at most 240 minutes (dispatch.maxLeaseMinutes in board.json); 500 is more. Ask for less, and renew it while you work. Nothing was written.
  ```

- Renew a card that went back to Ready:

  ```console
  codelead board renew 1 --as codelead
  ```

  ```output
  Refused: card #1 is held by no one (it is in Ready). A claim that is not renewed runs out, and the next take returns its card to Ready: ask the board for a card again with `codelead board take --as codelead`. Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Renewed. |
| `1` | Refused: not the holder, a card held by no one, a lease too long, or `busy` (ask again). |
| `2` | No board, no such card, or no `--as`. |

## Files

`renew` writes the claim's `until` in the card's file in `items/`, and appends `claim.renewed` to `events.jsonl`.

## See also

- [`codelead board take`](board-take.md), [`codelead board release`](board-release.md)
- [`codelead implement --ready`](../../cli/codelead-implement.md)
