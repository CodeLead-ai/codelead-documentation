---
title: codelead board take
description: An agent asks the board for its next card from Ready - the board gives one, held for it by a claim, or says why there is none.
kind: reference
area: pinboard
date: 2026-11-25
---

# codelead board take

**Applies to:** CodeLead 0.1 and later

`codelead board take` is how an agent gets a card from the **Ready** column. The agent does not choose the card. It asks, and the board gives it the next card it may work, or says why there is none.

## Name

`codelead board take` - Asks the board for the agent's next Ready card.

## Synopsis

```console
codelead board take --as <AGENT> [--lease <MINUTES>] [--board <DIR>] [--json]
codelead board take -h|--help
```

## Description

The person puts a card in Ready when an agent may pick it up alone. A card there says three things in its fields. It says how to tell it is done (`definitionOfDone`) and what it may touch (`mayTouch`). It names its limit (`limitMinutes`, `limitMoney`, or both).

`take` decides, under the board's lock, so that two agents never get the same card:

1. It first ends every claim whose time has passed. Those cards go back to the top of Ready.
2. It picks the next card for this agent. The cards tagged `for:<agent>` come first, then the cards with no `for:` tag. Each group is in the column's order, top first.
3. It moves the card to **In progress** and records a **claim**: who holds the card, since when, and until when.

A card waits, and is not given, while it is blocked by a card that is not in Done. It also waits while a card that another agent holds may touch the same files. An agent holds at most one card at a time, unless `dispatch.perAgent` in `board.json` says more.

The claim is a lease. The agent renews it while it works ([`codelead board renew`](board-renew.md)). It hands the card to **For review** when the work is done, with [`codelead board move`](board-move.md): that move ends the claim. [`codelead board release`](board-release.md) gives the card back to Ready.

> [!NOTE]
> **New to this?** A lease is like a library loan: the card is yours until a time, and you can extend it. When an agent stops working and does not renew, the claim runs out, and the next take puts the card back in Ready. No card stays stuck.

"Nothing for you" is an answer, not an error: `take` then exits with `0` and says why. Only an agent takes; a person moves cards.

## Options

- **`--as <AGENT>`**

  The agent that asks. It must be an agent of the board. Required.

- **`--lease <MINUTES>`**

  How long the claim lasts before it runs out, unless it is renewed. The default is the board's `dispatch.leaseMinutes` (30). The most is `dispatch.maxLeaseMinutes` (240).

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the answer as JSON: `given`, the card with its three things, the `claim`, and the `events`; or `given: false` with the `reason` and the `message`.

## Examples

These outputs come from a board in a scratch folder. The card numbers and times are examples.

- Ask for a card. The board gives the card that Sam put in Ready:

  ```console
  codelead board take --as codelead
  ```

  ```output
  Took #1 (task): Rate-limit the sign-in form
  codelead holds it until 2026-11-25T12:39:18Z (30 minutes): renew it while you work (`codelead board renew 1 --as codelead`), and hand it to For review when it is done.
  Done when: Five failed sign-ins in a minute lock the form for ten minutes; npm test passes.
  May touch: src/auth/, test/auth.test.ts
  Limit: 30 minutes
  docs/codelead/board/items/0001-rate-limit-the-sign-in-form.md
  ```

- Ask again while you hold that card:

  ```console
  codelead board take --as codelead
  ```

  ```output
  Nothing for codelead: it holds #1 (until 2026-11-25T12:39:18Z) already, and an agent holds at most 1 card at a time on this board (dispatch.perAgent in board.json). Hand it to For review, or give it back, then ask again.
  ```

- Ask on a board made before Ready:

  ```console
  codelead board take --as codelead
  ```

  ```output
  This board has no Ready column, so no card is given: Ready is where the person puts a card an agent may pick up by itself. `codelead board add-ready` gives the board its Ready column.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | A card was given, or the answer says why none was. |
| `1` | Refused: a person asked, a lease longer than the board allows, or another writer holds the board (`busy`: ask again). |
| `2` | No board, or no `--as`. |

## Files

`take` writes the card's `column` and its `claim` in `items/`, and appends `item.moved` and `claim.taken` to `events.jsonl`. A claim that it ends adds `claim.expired`. See [Files and events](../files-and-events.md).

## See also

- [`codelead board renew`](board-renew.md), [`codelead board release`](board-release.md)
- [`codelead board add-ready`](board-add-ready.md): give an older board its Ready column.
- [`codelead implement --ready`](../../cli/codelead-implement.md): CodeLead takes its cards this way.
- [Work with agents](../work-with-agents.md#hand-a-card-to-codelead-through-ready)
- [Rules for agents](../rules-for-agents.md#ready-is-the-persons)
