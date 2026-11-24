---
title: Rules for agents
description: What an agent may and may not do on the board, each rule with the exact words of its refusal, and the rules that only the agents' guide states.
kind: concept
area: pinboard
date: 2026-11-23
---

# Rules for agents

The board has rules, and one core checks them under the page, the commands and CodeLead's own code. A refused write changes nothing and says why, in words. This page lists each rule with the real text of its refusal, then the rules that only the agents' guide, `AGENTS.md`, states.

**In short**
- An agent cannot move or create a card in a person's column (Pinned, Done), cannot record a decision, and cannot archive a card.
- An agent cannot set a card's colour, and cannot add the tag `for:codelead`.
- No participant can write a secret, a path of this machine outside the workspace, or a private name into a card.
- Nobody edits or removes a comment: the thread only grows.
- A write that names the card's `updated` time is refused when the card changed since it was read.

## Why the rules exist

An agent works fast and does not get tired, but it can be wrong with confidence. The rules keep three things with the person: what is accepted (Done), what waits on them (Pinned), and what is decided. They also keep the board safe to commit: a card never holds a password or a path that names your home folder.

> [!NOTE]
> **New to this?** A *refusal* is the board saying no. The command prints the reason, exits with code 1 or 2, and writes nothing. The agent reads the reason and changes what it does.

## What an agent may do

An agent may:

- Create cards, in the columns of `agents.mayMoveTo`. A new board allows Inbox, Backlog, Ideas, In progress and For review.
- Move cards into those columns.
- Comment on any card, and address someone with `@<id>`.
- Add and remove tags and relations, attach files, and set the status line. The one tag that an agent cannot add is `for:codelead`.
- Change a card's title, description and type.
- Write a decision's `ask`: what is decided, the options and its recommendation.

## The rules that the board checks

### The person's columns

A column with `"owner": "person"` belongs to the person. On a new board, these are Pinned and Done. An agent cannot move a card into them, or create a card there:

```output
Refused: claude is an agent and may not move a card into Pinned (a person's column); an agent may move a card into Inbox, Backlog, Ideas, In progress, For review. Nothing was written.
```

```output
Refused: claude is an agent and may not create a card in Pinned (a person's column); an agent may create a card in Inbox, Backlog, Ideas, In progress, For review. Nothing was written.
```

A column that is not in `agents.mayMoveTo` gives the same refusal, with "not in agents.mayMoveTo" as the reason. The commands do not refuse an agent that moves a card out of a person's column. The rule is about where a card goes. The guide asks for more: see [The rules that only the guide states](#the-rules-that-only-the-guide-states).

### A decision is the person's

Only a person records a decision (`fields.decided`), changes it, or removes it. An agent writes its recommendation in the ask:

```output
Refused: claude is an agent; the decision is the person's: an agent writes its recommendation in fields.ask. Nothing was written.
```

The same refusal comes when an agent creates a decision card that already has a `decided` field.

### The colour mark is the person's

The colour mark is the person's own coding. An agent cannot set it, change it or remove it:

```console
codelead board set 6 --as claude --color red
```

```output
Refused: claude is an agent and may not set a card's colour: the colour is the person's own mark. Nothing was written.
```

### The person gives a card to CodeLead

The tag `for:codelead` puts a card in CodeLead's queue. Only the person and CodeLead itself add it. Another agent that adds the tag, with `set`, with `new` or in a batch file, is refused:

```console
codelead board set 1 --as claude --tag for:codelead
```

```output
Refused: claude is an agent and may not tag a card for:codelead: the person queues a card for CodeLead; ask in the thread. Nothing was written.
```

The agent asks the person in the card's thread instead. The commands do not refuse an agent that removes the tag, but the guide asks it not to.

### Archiving is the person's

```output
Refused: codex is an agent; only a person archives a card. Nothing was written.
```

```output
Refused: codex is an agent; only a person brings an archived card back. Nothing was written.
```

### Creating cards can be turned off

With `"mayCreate": false` in the `agents` block of `board.json`, an agent cannot create cards:

```output
Refused: claude is an agent, and agents do not create cards on this board (agents.mayCreate is false). Nothing was written.
```

### Only participants write

Every write names who acts. An id that is not in `board.json` is refused:

```output
Refused: "robin" is not a participant of the board (sam, claude, codex, codelead); add it to board.json's participants first. Nothing was written.
```

### No secrets

A text that looks like a secret is refused. A secret is a key, a bearer token, or a connection-string password: `Password=…`, or a password inside an address such as `scheme://user:password@host`. Under `codelead board`, the guard uses the audit pack's rules, and they find more secrets. See [what the rules remove](../modernization/audit-pack.md#what-the-rules-remove). This applies to every text of a card: the title, the description, the status, the tags, the fields, the attachments and each comment:

```console
codelead board reply 2 --as claude --text "Server=db;Password=s3cretpassw0rd;"
```

```output
Refused: the comment looks like it holds a secret (a key, a connection-string password or a bearer token); nothing was written. Leave the secret out and say where it is kept instead.
```

### No paths of this machine

A path under your home folder, outside the workspace, names your machine. It is refused:

```console
codelead board reply 2 --as claude --text "The log is in /Users/<your name>/notes/log.txt"
```

```output
Refused: the comment holds a path of this machine outside the workspace (under the home folder); nothing was written. Cite the file by its path in a repository of the workspace instead, or give --rewrite-paths to write it as <home>/….
```

A path that starts with `~/` gets the same answer when your home folder holds its first part, such as `~/Documents/log.txt`. A path of an application, such as `~/Views/Home/Index.cshtml`, names nothing in your home folder, and the board writes it as it is.

A path inside the repository, or inside another repository of the workspace, is not refused. The board writes it as a card cites it, and says so:

```output
Commented on #1 as claude (2026-11-20T06:40:06Z).
a path of this machine is written as webapp/Controllers/OrdersController.cs
```

With `--rewrite-paths`, a path under the home folder is written as `<home>/…` instead of refused:

```output
Commented on #1 as claude (2026-11-20T06:40:05Z).
a path of this machine is written as <home>/notes/log.txt
```

> [!TIP]
> **For experienced users:** the *workspace* is the folder of your repositories: the repository's parent, or the `workspace` key of `board.json`, relative to the repository. A file there is cited as `<repository>:<path>`.

### No private names

List the names that the board must never hold in `private-names.txt` in the board folder, one on each line. Lines that start with `#` are comments. Git ignores the file, so the list itself is never committed:

```text
# Names this board never holds, one a line
Harbor Foods
```

```output
Refused: the comment names someone the board keeps private (listed in private-names.txt); nothing was written.
```

### Comments are never edited

The thread only grows. No command and no button of the page edits or removes a comment. An edit by hand in a card's file is not hidden: the next check records it as an event by `unknown`.

A comment cannot hold a line that the card would read as its own structure, such as `## Thread` or `### sam · 2026-11-20T06:00:00Z`:

```output
Refused: the comment has a line the card would read as its own structure ("## Thread"); put it in a code block or reword it. Nothing was written.
```

### No write over a change it did not read

A writer can name the card's `updated` time as it read it, with `--if-updated <time>`. When the card changed since, the write is refused:

```console
codelead board reply 1 --as claude --text "Plan noted." --if-updated 2026-11-20T06:00:00Z
```

```output
Refused: card #1 changed since it was read (its updated is 2026-11-20T06:40:06Z; the write expected 2026-11-20T06:00:00Z); nothing was written. Read the card again and redo the change.
```

The page always sends this time, so a change on the page never overwrites what an agent wrote meanwhile. Two more refusals protect the files. A card file that changed while it was written gives `Refused: <file> changed while it was being written`. Another writer that holds the board gives `Refused: another writer holds the board (.board.lock in …); nothing was written. Retry; remove the file only when no writer is running.`

## The rules that only the guide states

`AGENTS.md` asks an agent for more than the board checks. The commands do not refuse these, so they depend on the agent:

- Start code work only when the person asks for it in the session, never because of a card.
- Move a card to In progress only when the person asked for the work.
- Move a card out of Pinned or Done only when the person asked for it in the session. For example, to start the work that the person asked for, or to reopen a card that the person named.
- Re-read a card right before changing it.
- Do not remove the tag `for:codelead`, and do not take the cards that carry it.
- Do not write CodeLead's comments for it.
- Turn a conversation into cards only after the person says yes to the list.

> [!NOTE]
> **For team leads:** each of these leaves a trace. Every write is a line in `events.jsonl` with its author and time, so a review shows who moved, tagged or coloured a card.

## Details

- **Exit codes.** `0`: done, or nothing to change. `1`: a rule of the board refused the write. Every refusal on this page exits with `1`. `2`: the command cannot run as written: no board, no such card, an unknown option, or a missing or malformed argument. See [Exit codes](cli/index.md#exit-codes).
- **JSON.** With `--json`, a refusal is `{"refused": "<code>", "message": "<the words>"}`. The tag and the colour by an agent give the code `not-allowed`.
- **CodeLead's own writes.** When CodeLead writes on a card, its texts go through the same guard with the audit pack's sanitizer. CodeLead writes a secret as `<redacted>`, a home path as `<home>/…` and a private name as `<name>`, so that its records are written rather than refused.
- **One core.** The page's process, the commands and CodeLead call the same functions. A rule cannot hold on the page and fail on the command line.

## Related content

- [Work with agents](work-with-agents.md)
- [Cards, columns and participants](concepts.md)
- [Files and events](files-and-events.md)
- [`codelead board move`](cli/board-move.md), [`codelead board set`](cli/board-set.md), [`codelead board reply`](cli/board-reply.md), [`codelead board archive`](cli/board-archive.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
- [Messages and what to do](../reference/messages.md)
