---
title: Files and events
description: The board's folder file by file - board.json, the cards under items/, events.jsonl with every event kind and its fields, the cursors, .local/, private-names.txt and archived cards - with real examples.
kind: reference
area: pinboard
date: 2026-11-20
---

# Files and events

**Applies to:** CodeLead 0.1 and later

The board is a folder of plain files. A person, the page and every agent read and write the same files, so this page is the board's whole format. The examples come from a board about the Basic Store sample, with one person, `sam`, and the agents `claude`, `codex` and `codelead`.

**In short**
- `board.json` holds the columns, the card types, the participants and the agents' limits.
- Each card is a markdown file under `items/`, with YAML front matter, a description and a thread.
- `events.jsonl` gets one JSON line for each change, and is never edited.
- `cursors/`, `.local/`, `private-names.txt` and the lock stay on each machine. Git ignores them.

## The board folder

The folder is `docs/codelead/board` in the repository, unless a command names another with `--board <dir>`:

```text
docs/codelead/board/
  board.json            the board: columns, types, participants, rules
  AGENTS.md             the guide for agents, written by init
  events.jsonl          one line for each change
  .gitignore            keeps the machine's own files out of git
  items/
    0001-show-the-stock-on-the-order-form.md
    0002-must-a-client-see-the-orders-of-other.md
    …
  cursors/              not in git
    sam.json
    claude.json
    .cards.json
  .local/               not in git
    preferences.json
  private-names.txt     not in git, optional
```

[`codelead board init`](cli/board-init.md) writes `board.json`, `AGENTS.md`, an empty `events.jsonl`, `.gitignore`, `items/` and `cursors/`. It never overwrites a file that is there.

## board.json

```json
{
  "schema": "codelead.board/1",
  "name": "store-workspace",
  "columns": [
    { "id": "inbox", "name": "Inbox" },
    { "id": "backlog", "name": "Backlog" },
    { "id": "ideas", "name": "Ideas" },
    { "id": "pinned", "name": "Pinned", "owner": "person" },
    { "id": "in-progress", "name": "In progress" },
    { "id": "for-review", "name": "For review" },
    { "id": "done", "name": "Done", "owner": "person" }
  ],
  "types": ["question", "decision", "task", "idea", "review", "note"],
  "tagNamespaces": ["repo", "area", "by"],
  "participants": [
    { "id": "sam", "kind": "person", "name": "Sam" },
    { "id": "claude", "kind": "agent", "name": "Claude" },
    { "id": "codex", "kind": "agent", "name": "Codex" },
    { "id": "codelead", "kind": "agent", "name": "CodeLead" }
  ],
  "agents": { "mayMoveTo": ["inbox", "backlog", "ideas", "in-progress", "for-review"], "mayCreate": true },
  "check": { "intervalMinutes": 30 },
  "nextId": 1
}
```

| Key | What it holds |
|---|---|
| `schema` | Always `codelead.board/1`. |
| `name` | The board's name. The default is `<repository folder>-workspace`. |
| `columns` | The columns in their order: `id`, `name`, and `"owner": "person"` for a person's column. A column can also have a `limit`. |
| `types` | The card types. |
| `tagNamespaces` | The reserved tag namespaces. |
| `participants` | Each person and agent: `id`, `kind` (`person` or `agent`), `name`. |
| `agents.mayMoveTo` | The columns that an agent may move a card into, or create one in. It cannot name a person's column. |
| `agents.mayCreate` | `false` stops agents from creating cards. |
| `check.intervalMinutes` | How often an agent checks. The default is 30. |
| `nextId` | The number of the next card. |
| `workspace` | Optional. The folder of your repositories, relative to this repository. The default is `..`, the repository's parent. |
| `imports` | Written by [`codelead board import`](cli/board-import.md): one marker for each document it imported, so that it runs once. |

Unknown keys are kept as they were read. A write that only raises `nextId` changes that number and no other byte. The id `unknown` cannot be a participant: it names the author of a change that came without an event.

## A card: items/NNNN-slug.md

Each card is one file. Its name starts with the id, padded to four digits, then a slug of the title: `0001-show-the-stock-on-the-order-form.md`. The id is the key; the slug only helps a person find the file.

```markdown
---
id: 1
type: task
title: "Show the stock on the order form"
column: done
tags: [area:orders, repo:webapp, by:sam, for:codelead]
author: sam
created: 2026-11-20T06:39:23Z
updated: 2026-11-20T06:41:09Z
fields:
  landed: ["4f2a9c1 on main; webapp/Controllers/OrdersController.cs, webapp/Views/Orders/Create.cshtml (3 tests)."]
status: "the stock query works; the view is next"
archived: 2026-11-20T06:41:09Z
---

When a clerk adds a product to an order, the form must show how many are in stock. See webapp/Controllers/OrdersController.cs.

## Thread

### claude · 2026-11-20T06:40:29Z
Started: Sam asked for this card in the session.

### claude · 2026-11-20T06:40:30Z
Landed: 4f2a9c1 on main; webapp/Controllers/OrdersController.cs, webapp/Views/Orders/Create.cshtml (3 tests).
```

### The front matter

The front matter is YAML between the two `---` lines.

| Key | Required | What it holds |
|---|---|---|
| `id` | Yes | The card's number. |
| `type` | Yes | One of the board's types. |
| `title` | Yes | One line. |
| `column` | Yes | A column's id. |
| `author` | Yes | The participant who created the card. |
| `created`, `updated` | Yes | UTC times, such as `2026-11-20T06:39:23Z`. Every write sets `updated`. |
| `tags` | No | A list of tags. |
| `relations` | No | A list such as `- { kind: related, to: 5 }`. |
| `attachments` | No | A list such as `- { path: "docs/plan.md", lines: "40-60" }`. |
| `fields` | No | The type's own fields. See [Cards and their types](concepts.md#cards-and-their-types). |
| `color` | No | The person's colour mark: `"#b33f40"` red, `"#ed944d"` orange, `"#edd94c"` yellow, `"#48a14d"` green or `"#6577b3"` blue. |
| `status` | No | The one-line status. |
| `session` | No | The agent session that the card came from. |
| `archived` | No | When the card left the board. |

Unknown keys, comments and blank lines are kept. A change rewrites only the lines that it changes.

### The description and the thread

The text before `## Thread` is the description, in markdown. Each comment is a heading `### <participant> · <UTC time>`, then its markdown. Comments are added at the end, never rewritten. The description cannot hold a line `## Thread`, and a comment cannot hold a line that looks like the card's own structure.

### Paths in a card

A path in a card is a path in the repository, such as `webapp/Controllers/OrdersController.cs`. A file of another repository of the workspace is written `<repository>:<path>`. See [No paths of this machine](rules-for-agents.md#no-paths-of-this-machine).

### Archived cards

An archived card keeps its file in `items/`, with an `archived` time in its front matter. It leaves the page's columns and `list` until you ask for archived cards. [`codelead board unarchive`](cli/board-unarchive.md) removes the key, and the card returns to its column.

## events.jsonl

Each line is one JSON object, with its keys in this order: `at`, `by`, `item`, `kind`, `data`. Lines are only added. The card file is the truth; the line is what a check reports.

```json
{"at":"2026-11-20T06:40:20Z","by":"sam","item":2,"kind":"comment.added","data":{"excerpt":"Answered: own orders only.","withMove":true}}
{"at":"2026-11-20T06:40:20Z","by":"sam","item":2,"kind":"item.moved","data":{"from":"inbox","to":"done","withComment":true}}
```

| Key | What it holds |
|---|---|
| `at` | The UTC time of the change. |
| `by` | The participant, or `unknown` for a change that came without an event. |
| `item` | The card's id. |
| `kind` | What happened. See the table below. |
| `data` | The details of that kind. |

### Every event kind

| Kind | `data` |
|---|---|
| `item.created` | `title`, `type`, `column`. Also `decided` when the card is created with a decision, `addressed` when the description names someone, `rewritten` for the paths it rewrote, and `source` for a card from an import. |
| `item.updated` | `fields`: the parts that changed, from `title`, `type`, `description`, `status`, `color`, `ask` and `fields`. Also `decided` (`{ option, by, on }`, or `null` when removed), `addressed`, `rewritten`, and `landed` for a `Landed:` comment. |
| `item.moved` | `from`, `to`. Also `after`, the card it was dropped below, and `withComment` when a comment came with the move. |
| `item.archived` | `column`: where the card stood. |
| `item.unarchived` | `column`: where the card returns. |
| `comment.added` | `excerpt`: the first 120 characters on one line. Also `addressed`, `rewritten`, and `withMove` when the comment came with a move. |
| `relation.added` | `kind`, `to`. |
| `relation.removed` | `kind`, `to`. |
| `tag.added` | `tag`. Also `rewritten`. |
| `tag.removed` | `tag`. |
| `attachment.added` | `path`, and `lines` when given. Also `rewritten`. |

Real lines for a decision, a landing and a tag:

```json
{"at":"2026-11-20T06:40:19Z","by":"sam","item":3,"kind":"item.updated","data":{"fields":["fields"],"decided":{"option":"2026-11-20","by":"sam","on":"2026-11-20"}}}
{"at":"2026-11-20T06:40:30Z","by":"claude","item":1,"kind":"item.updated","data":{"fields":["fields"],"landed":"4f2a9c1 on main; webapp/Controllers/OrdersController.cs, webapp/Views/Orders/Create.cshtml (3 tests)."}}
{"at":"2026-11-20T06:40:18Z","by":"sam","item":1,"kind":"tag.added","data":{"tag":"for:codelead"}}
```

### Changes without an event

When a card file changes and no event names it, for example after an edit by hand, the next check adds an event by `unknown`. Its time is the file's time, and `data.synthesized` is `true`:

```json
{"at":"2026-11-20T06:42:15Z","by":"unknown","item":4,"kind":"item.updated","data":{"synthesized":true,"fields":["description"]}}
```

A check prints it as `unknown · 2026-11-20T06:42:15Z  changed its file with no event (description)`. The page's process adds the same event once the file has been quiet for a moment. A new card file with no event gets an `item.created` by `unknown`.

## Cursors

`cursors/<participant>.json` says what a participant has seen:

```json
{ "seen": "2026-11-20T06:41:51Z", "lines": 39, "opened": {} }
```

| Key | What it holds |
|---|---|
| `seen` | The time of the last check with `--ack`. |
| `lines` | How many lines of `events.jsonl` the participant has read: the exact place of the cursor. |
| `opened` | For each card id, when the participant last opened the card. Its events before that time are seen. |

No cursor file means that everything on the board is new to that participant. A cursor that cannot be read is refused, with the advice to fix it or delete it. `cursors/.cards.json` is the machine's own record of each card file, which finds the changes without an event. Cursors stay on each machine.

## The `.local/` folder

`.local/preferences.json` holds this machine's view choices: the theme and the board background, by default and for each project, and the project shown last. The page writes it. See [Keep a theme for one project](projects-themes-and-bulk.md#keep-a-theme-for-one-project). No event is written for it.

## private-names.txt

Optional. One name on each line; lines that start with `#` are comments. No write may put one of these names into a card. See [No private names](rules-for-agents.md#no-private-names).

## .gitignore and the lock

The board's `.gitignore`, as `init` writes it:

```text
cursors/
.board.lock
private-names.txt
.*.tmp
.local/
```

`.board.lock` exists while a writer holds the board. Another writer waits for it up to 3 seconds, then is refused, and nothing is written. A lock older than 30 seconds was left by a process that stopped, and the next writer takes it over.

## AGENTS.md

The guide for agents. `init` writes it from `board.json`: the columns, the agents' columns and the interval. It lists the files, how to check, how to write, what never to do, and how to record work. See [Rules for agents](rules-for-agents.md).

## Imports

[`codelead board import`](cli/board-import.md) marks each document that it imported in `board.json`:

```json
"imports": [
  { "source": "docs/PINNED.md", "at": "2026-11-20T06:43:24Z", "by": "sam", "sha256": "56e88384…", "cards": [8, 9] }
]
```

The import does not write the mapping that it prints to a file. To keep it, save the output yourself, for example in an `imports/` folder in the board.

## See also

- [Cards, columns and participants](concepts.md)
- [Rules for agents](rules-for-agents.md)
- [`codelead board validate`](cli/board-validate.md): check every file of the board.
- [`codelead board check`](cli/board-check.md): read the events after your cursor.
- [`codelead board init`](cli/board-init.md)
- [Files and folders](../reference/files-and-folders.md)
