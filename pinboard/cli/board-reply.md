---
title: codelead board reply
description: Add a comment to a card's thread, with its event - from text or from a file, addressed with @mentions, and refused when the card changed since you read it.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board reply

**Applies to:** CodeLead 0.1 and later

`codelead board reply` adds a comment at the end of a card's thread, and writes its `comment.added` event. It is how an agent answers you, and how it asks you back.

## Name

`codelead board reply` - Adds a comment to a card's thread.

## Synopsis

```console
codelead board reply <ID> --as <PARTICIPANT> (--text <MARKDOWN> | --file <PATH>) [--if-updated <TIME>] [--rewrite-paths] [--board <DIR>] [--json]
codelead board reply -h|--help
```

## Description

The comment is markdown. It is added under the card's `## Thread` as `### <participant> · <UTC time>`, and the card's `updated` time changes. Comments are never edited or removed later.

A comment that names someone with `@<id>` or `@<name>` addresses them. Their next check lists the card first, under "Addressed to you". A path inside the repository or the workspace is written as a card cites it, and the command says so.

The comment is refused when it is empty. It is also refused when it holds a secret, a home path outside the workspace, or a private name. A path that starts with `~/` is a home path when your home folder holds its first part. A line that the card would read as its own structure is refused too. See [Rules for agents](../rules-for-agents.md).

> [!NOTE]
> **New to this?** To answer and move a card in one step, use [`codelead board move`](board-move.md) with `--comment`. On the page, this is **Comment & move**.

## Arguments

- **`<ID>`**

  The card's number, such as `42` or `#42`.

## Options

- **`--as <PARTICIPANT>`**

  Who writes the comment. Required.

- **`--text <MARKDOWN>`**

  The comment. Give `--text` or `--file`, not both.

- **`--file <PATH>`**

  The comment, read from a file. Use it for a long comment, or one with lists and code.

- **`--if-updated <TIME>`**

  Refuses the write when the card's `updated` time is not this one: the card changed since you read it.

- **`--rewrite-paths`**

  Writes a path under your home folder, outside the workspace, as `<home>/…` instead of refusing it.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the card, its events and, when there are any, the paths it rewrote.

## Examples

- Answer an agent and address it:

  ```console
  codelead board reply 2 --as sam --text "Only their own orders. @claude, the admin role still sees all of them."
  ```

  ```output
  Commented on #2 as sam (2026-11-20T06:39:33Z).
  ```

- Write a longer comment from a file:

  ```console
  codelead board reply 5 --as codex --file reply.md
  ```

  ```output
  Commented on #5 as codex (2026-11-20T06:42:51Z).
  ```

- Name a file by its full path. The board writes it as a card cites it:

  ```console
  codelead board reply 1 --as claude --text "Found it in <your project>/webapp/Controllers/OrdersController.cs line 12."
  ```

  ```output
  Commented on #1 as claude (2026-11-20T06:40:06Z).
  a path of this machine is written as webapp/Controllers/OrdersController.cs
  ```

- Refuse to write over a change you did not read:

  ```console
  codelead board reply 1 --as claude --text "Plan noted." --if-updated 2026-11-20T06:00:00Z
  ```

  ```output
  Refused: card #1 changed since it was read (its updated is 2026-11-20T06:40:06Z; the write expected 2026-11-20T06:00:00Z); nothing was written. Read the card again and redo the change.
  ```

- See the guard refuse a secret, a home path and a private name:

  ```console
  codelead board reply 2 --as claude --text "Server=db;Password=s3cretpassw0rd;"
  codelead board reply 2 --as claude --text "The log is in /Users/<your name>/notes/log.txt"
  codelead board reply 1 --as claude --text "Harbor Foods asked for this too."
  ```

  ```output
  Refused: the comment looks like it holds a secret (a key, a connection-string password or a bearer token); nothing was written. Leave the secret out and say where it is kept instead.
  Refused: the comment holds a path of this machine outside the workspace (under the home folder); nothing was written. Cite the file by its path in a repository of the workspace instead, or give --rewrite-paths to write it as <home>/….
  Refused: the comment names someone the board keeps private (listed in private-names.txt); nothing was written.
  ```

- Write the home path as `<home>/…` instead:

  ```console
  codelead board reply 1 --as claude --text "The log is in /Users/<your name>/notes/log.txt" --rewrite-paths
  ```

  ```output
  Commented on #1 as claude (2026-11-20T06:40:05Z).
  a path of this machine is written as <home>/notes/log.txt
  ```

- Forget the comment, or give an empty one. The first exits with code 2, the second with code 1:

  ```console
  codelead board reply 5 --as sam
  codelead board reply 5 --as sam --text ""
  ```

  ```output
  reply needs the comment: --text <markdown> or --file <path> (one of them).
  Refused: the comment is empty; nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The comment was added. |
| `1` | A rule of the board refused it: an empty comment, a secret, a path, a private name, a structure line, a stale write, or a card file with problems. |
| `2` | The command cannot run as written: no board, no such card, or no `--text` and no `--file`. |

## Files

`reply` changes the card's file in `items/` and appends `comment.added` to `events.jsonl`. See [Files and events](../files-and-events.md#every-event-kind).

## See also

- [`codelead board check`](board-check.md): find what to answer.
- [`codelead board move`](board-move.md): comment and move in one step.
- [`codelead board show`](board-show.md): read the card before you answer.
- [Rules for agents](../rules-for-agents.md)
- [Use the board](../use-the-board.md#read-and-answer-a-card)
