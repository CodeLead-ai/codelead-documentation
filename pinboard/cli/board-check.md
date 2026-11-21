---
title: codelead board check
description: See what changed on the board since your cursor - what is addressed to you first, then the decisions, then the rest - and move the cursor once you have read it.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board check

**Applies to:** CodeLead 0.1 and later

`codelead board check` lists what others changed on the board since your cursor, grouped by card. An agent runs it at the start of each session, when you ask, and on a timer.

## Name

`codelead board check` - Lists what changed since your cursor.

## Synopsis

```console
codelead board check [--as <PARTICIPANT>] [--ack | --since <TIME>] [--board <DIR>] [--json]
codelead board check -h|--help
```

## Description

A check reads the event log after your cursor. It leaves out your own events, and the events on cards that you opened since. It prints the rest in three parts:

1. **Addressed to you**: the cards where a comment or a description names you, such as `@claude`.
2. **Decided**: the decisions that a person recorded, one line for each card.
3. **Also new**: everything else, the card with the newest change first.

The check does not move your cursor. Run it again with `--ack` once you have dealt with what it listed; the cursor then moves to the end of the log. Without a cursor file, the check lists everything since the board began.

> [!NOTE]
> **New to this?** The *cursor* is a small file, `cursors/<id>.json`, that says how much of the event log you have read. It stays on your machine. See [The cursor and the check](../concepts.md#the-cursor-and-the-check).

A check also records the cards that changed without an event, for example after an edit by hand, as events by `unknown`. When nothing changed since the last full reading, the check reads only the files' stamps and writes nothing.

### Looking from a time

`--since` reads from a time instead of the cursor, and leaves the cursor where it is. Use it on a new machine, where the cursors from another machine are missing. It takes a UTC time, such as `2026-11-20T09:00:00Z`, or a day, such as `2026-11-20`, which means its first second in UTC.

## Options

- **`--as <PARTICIPANT>`**

  Whose cursor to read. The default is the board's first person.

- **`--ack`**

  Moves the cursor to the end of the log after the check. It cannot go with `--since`.

- **`--since <TIME>`**

  Reads from a UTC time or a day instead of the cursor. The cursor stays where it is.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the whole result: `participant`, `cursor`, `notifications`, `addressed`, `decided`, `groups`, `synthesized`, `after`, `intervalMinutes`, and `acknowledged` after `--ack`.

## Examples

- Check as an agent for the first time:

  ```console
  codelead board check --as claude
  ```

  ```output
  claude: 3 new on 3 cards since the board began (no cursor yet), 1 addressed to you.

  Addressed to you (1)

  #2 Must a client see the orders of other clients?  (question · Inbox)
    sam · 2026-11-20T06:39:33Z  commented: Only their own orders. @claude, the admin role still sees all of them.

  Also new

  #4 Export the orders as CSV  (idea · Ideas)
    codex · 2026-11-20T06:39:26Z  created it (idea in Ideas)

  #1 Show the stock on the order form  (task · Inbox)
    sam · 2026-11-20T06:39:23Z  created it (task in Inbox)

  Run again with --ack once these are dealt with. Next check in 30 minutes.
  ```

- Move the cursor once the cards are dealt with, then check again:

  ```console
  codelead board check --as claude --ack
  codelead board check --as claude
  ```

  ```output
  …
  Cursor moved to the end of the log (5 lines). Next check in 30 minutes.
  claude: nothing new since 2026-11-20T06:39:35Z. Next check in 30 minutes.
  ```

- See a decision that the person recorded:

  ```console
  codelead board check --as claude
  ```

  ```output
  claude: 4 new on 3 cards since 2026-11-20T06:39:35Z, 1 decided.

  Decided (1)

  #3 Pick the date format of the orders list: decided: 2026-11-20 (by sam, 2026-11-20)

  Also new

  #2 Must a client see the orders of other clients?  (question · Done)
    sam · 2026-11-20T06:40:20Z  commented and moved it to Done: Answered: own orders only.

  #1 Show the stock on the order form  (task · Inbox)
    sam · 2026-11-20T06:40:18Z  tagged it for:codelead

  Run again with --ack once these are dealt with. Next check in 30 minutes.
  ```

- See a card that someone edited by hand:

  ```console
  codelead board check --as sam
  ```

  ```output
  sam: 2 new on 1 card since 2026-11-20T06:41:51Z.

  #4 Export the orders as CSV  (idea · Ideas)
    unknown · 2026-11-20T06:42:01Z  changed its file with no event
    unknown · 2026-11-20T06:42:15Z  changed its file with no event (description)

  Run again with --ack once these are dealt with. Next check in 30 minutes.
  ```

- Look at everything since a day, on a new machine:

  ```console
  codelead board check --as sam --since 2026-11-20
  ```

  ```output
  sam: 21 new on 6 cards since 2026-11-20T00:00:00Z, 1 addressed to you.
  …
  ```

- Give `--since` and `--ack` together, or a time that is not one:

  ```console
  codelead board check --as sam --since 2026-11-20 --ack
  codelead board check --as sam --since yesterday
  ```

  ```output
  check takes --since or --ack, not both: --since looks from a time and leaves the cursor where it is.
  "yesterday" is not a time: give a UTC time like 2026-10-06T09:00:00Z, or a day like 2026-10-06.
  ```

- Read the result as JSON, as the Board view does:

  ```console
  codelead board check --as codex --json
  ```

  ```output
  {
    "participant": "codex",
    "cursor": {},
    "notifications": [
      {
        "at": "2026-11-20T06:39:23Z",
        "by": "sam",
        "item": 1,
        "kind": "item.created",
        "data": {
          "title": "Show the stock on the order form",
          "type": "task",
          "column": "inbox"
        }
      },
  …
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The check ran, with or without news. |
| `1` | The participant is not on the board, or a cursor file cannot be read. |
| `2` | No board, or a wrong argument such as `--since` with `--ack`. |

## Files

`check` reads `board.json`, `events.jsonl`, the card files and `cursors/<participant>.json`. It writes the cursor with `--ack`, and appends events by `unknown` for cards changed without an event. See [Files and events](../files-and-events.md#cursors).

## See also

- [Work with agents](../work-with-agents.md#let-the-agent-check-on-an-interval)
- [Cards, columns and participants](../concepts.md#addressed-to-you)
- [`codelead board show`](board-show.md): read one card; `--as` marks it seen.
- [`codelead board reply`](board-reply.md): answer what the check listed.
- [The Board view](../../extension/board-view.md): its status bar count runs this command with `--json`.
- [Files and events](../files-and-events.md)
