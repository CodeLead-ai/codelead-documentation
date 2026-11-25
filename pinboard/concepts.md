---
title: Cards, columns and participants
description: The ideas of the board - cards and their types, columns and whose they are, threads, tags, relations, decisions, mentions, the cursor and the check, and the event log.
kind: concept
area: pinboard
date: 2026-11-25
---

# Cards, columns and participants

The board is made of a few simple ideas. A **card** is one piece of work. A **column** says where the work stands. A **participant** is a person or an agent that takes part. Every change is an **event** in a log, and each participant has a **cursor** that says how far it has read.

**In short**
- A card has a type, a title, a description, tags, relations and a thread of comments. It lives in one column.
- Three columns, **Pinned**, **Ready** and **Done**, belong to the person. No agent moves a card into them.
- A decision card asks you a question in its **ask**. Only a person records the answer, in its `decided` field.
- A **check** lists what changed since your cursor: what is addressed to you first, then the decisions, then the rest.

## Why it exists

Work between a person and an agent often lives in chat. A question from the agent scrolls away, and a decision is lost when the session ends. The board keeps each question, task and decision as a card in the repository. A new session, or a second agent, reads the same cards.

The board also sets limits. An agent can propose work and report on it. It cannot decide for you, close your work, or write a secret into a card. See [Rules for agents](rules-for-agents.md).

## Participants

The participants are listed in `board.json`. Each has an id, a kind (`person` or `agent`) and a name:

```json
"participants": [
  { "id": "sam", "kind": "person", "name": "Sam" },
  { "id": "claude", "kind": "agent", "name": "Claude" },
  { "id": "codex", "kind": "agent", "name": "Codex" },
  { "id": "codelead", "kind": "agent", "name": "CodeLead" }
]
```

This list is the whole account system: there are no passwords and no server. A command that writes a card names who acts, with `--as <id>`. The page always writes as the board's first person. A board that [`codelead board init`](cli/board-init.md) makes has CodeLead on it as the agent `codelead`.

## Cards and their types

A card is one markdown file under `items/`. It has a number (its id, such as `#7`), a type, a title and a description in markdown. The six types, and the fields that each type adds:

| Type | Use it for | Its fields |
|---|---|---|
| `task` | Work to do. | `definitionOfDone`, `size` (S, M or L), `landed` |
| `question` | Something one participant asks another. | `askedBy`, `answer` |
| `decision` | A choice that the person makes. | `ask`, `options`, `recommendation`, `decided`, `consequences` |
| `idea` | Something worth trying later. | `hypothesis`, `worthItIf` |
| `review` | Work to look at. | `toReview`, `verdict` |
| `note` | Something to know. | none |

A card also has these optional parts:

- **A status**: one line beside the column, such as "the stock query works; the view is next".
- **A colour mark**: one of five colours (red, orange, yellow, green, blue). The colour is the person's own coding. The commands refuse it from an agent.
- **Attachments**: paths in the repository, with lines when needed, such as `docs/plan.md:40-60`.
- **A session**: the id of the agent session that the card came from.

> [!NOTE]
> **New to this?** *Markdown* is plain text with light marks: `**bold**`, `- a list item`, a line starting with `#` for a heading. The page shows it formatted.

## Columns and whose they are

A new board has eight columns. Each column says where the work stands:

| Column | Meaning | Who moves cards into it |
|---|---|---|
| Inbox | New cards land here. | Anyone |
| Backlog | Planned, not started. | Anyone |
| Ideas | Parked for later. | Anyone |
| Pinned | Waits on you. | The person only |
| Ready | An agent may work it alone: it says how to tell it is done, what it may touch, and its limit. | The person only |
| In progress | Someone works on it. | Anyone |
| For review | Done by an agent, waiting for your look. | Anyone |
| Done | Accepted by you. | The person only |

A column with `"owner": "person"` in `board.json` belongs to the person. An agent that tries to move a card there gets a refusal, and nothing changes. An agent moves a card out of it only when the person asked for that in the session. This is a rule of the agents' guide, and no command checks it. The columns that an agent may use are `agents.mayMoveTo` in `board.json`.

Ready is stricter: the commands refuse an agent that moves a card out of it too. An agent gets a card from Ready only from the board, with [`take`](cli/board-take.md). A board made before Ready gets the column with [`add-ready`](cli/board-add-ready.md). See [Ready is the person's](rules-for-agents.md#ready-is-the-persons).

Archiving takes a card off the board. The card stays in its file, and the archived filter shows it again. Only a person archives a card or brings it back.

## Threads

The thread is the list of comments under a card's description. Each comment has its author and its time in UTC. Comments are only added, never edited or removed. This rule keeps the record honest: what an agent said stays as it said it.

A comment can move the card in the same step. On the page this is **Comment & move**. With the commands it is `move --comment`. A comment whose first line starts with `Landed:` also records what landed on a task, in its `landed` field.

## Tags

A tag is a short word on a card, such as `area:orders`. A tag with a colon has a namespace before it. Three namespaces are reserved:

- `repo:<name>` names the project (the repository) a card belongs to. The page's project selector uses it. See [Projects, themes and bulk operations](projects-themes-and-bulk.md).
- `area:<name>` names a part of the work, such as `area:orders`.
- `by:<id>` names who proposed the card. A card that an agent creates gets `by:<agent>`.

Two more tags have a meaning for the tools. `for:codelead` hands a card to CodeLead. Only the person adds it: the commands refuse it from another agent. `source:conversation` marks cards that an agent made from a conversation. See [Work with agents](work-with-agents.md).

## Relations

A relation links one card to another. The card that states a relation keeps it in its own file, and the other card shows it too. The kinds are `parent-of`, `child-of`, `related`, `blocks`, `blocked-by`, `duplicate-of` and `follows`. Seen from the other card, `blocks` reads as `blocked-by`, and `follows` reads as `followed-by`.

## Decisions and the ask

A decision card asks the person to choose. Its `ask` field says what is decided, the options and the recommendation, in one to three sentences. The page shows the ask on the card's face, so you can answer without opening it.

The person records the answer in the `decided` field: the option chosen, by whom, and on which day. The card file keeps it like this:

```yaml
fields:
  ask: "Show dates as 2026-11-20 or as 20 Nov 2026? I recommend 2026-11-20."
  options: [2026-11-20, "20 Nov 2026"]
  decided:
    option: 2026-11-20
    by: sam
    "on": 2026-11-20
```

Only a person sets `decided` or removes it. An agent writes its recommendation in the ask. The next check of every agent lists the answer under "Decided".

## Addressed to you

Write `@<id>` or `@<name>` in a comment or a description to address a participant, such as `@claude` or `@sam`. Case does not matter. A name of several words is written joined, such as `@ClaudeCode`. A word in code, an email address or a package name such as `@types/node` is not a mention.

The event of that write records who it addresses, in `data.addressed`. That participant's next check lists the card first, under "Addressed to you". On the page, the card shows **@you**.

## The cursor and the check

Each participant has a cursor file, `cursors/<id>.json`. It holds three things. `lines` is how many lines of the event log the participant has read. `seen` is the time of its last `--ack`. `opened` says when it last opened each card. A **check** lists the events by others after the cursor, grouped by card:

```output
sam: 3 new on 3 cards since the board began (no cursor yet), 1 addressed to you.

Addressed to you (1)

#2 Must a client see the orders of other clients?  (question · Inbox)
  claude · 2026-11-20T06:39:24Z  created it (question in Inbox)

Also new

#4 Export the orders as CSV  (idea · Ideas)
  codex · 2026-11-20T06:39:26Z  created it (idea in Ideas)
…
Run again with --ack once these are dealt with. Next check in 30 minutes.
```

The check leaves the cursor where it is. With `--ack` it moves the cursor to the end of the log. Opening a card on the page also marks its events as seen. Cursors stay on each machine: git ignores them. See [`codelead board check`](cli/board-check.md).

## The event log

Every write appends one line or more to `events.jsonl`. A line says who did what to which card, and when:

```json
{"at":"2026-11-20T06:39:33Z","by":"sam","item":2,"kind":"comment.added","data":{"excerpt":"Only their own orders. @claude, the admin role still sees all of them.","addressed":["claude"]}}
```

The card file is the truth. The log line is what a check reports. When a card file changes without an event, for example after an edit by hand, the next check adds an event by `unknown`. See [Files and events](files-and-events.md).

## What it is not

- **Not a task runner.** A card never starts work by itself. An agent works on a card when you ask for it in the agent's session. CodeLead works on a card when you run [`codelead implement`](../cli/codelead-implement.md) or [`/implement`](../chat-commands/implement.md).
- **Not a model.** The board never calls a model and never runs code.
- **Not a server in the cloud.** The files are in your repository. The page runs on your machine and answers only there.

## Related content

- [Use the board](use-the-board.md)
- [Work with agents](work-with-agents.md)
- [Rules for agents](rules-for-agents.md)
- [Files and events](files-and-events.md)
- [Board commands](cli/index.md): [`check`](cli/board-check.md), [`new`](cli/board-new.md), [`reply`](cli/board-reply.md), [`move`](cli/board-move.md), [`set`](cli/board-set.md), [`archive`](cli/board-archive.md)
- [The Board view](../extension/board-view.md)
- [Glossary](../concepts/glossary.md)
