---
title: The Sessions view
description: The session logs of each project, newest first, and the Session tab that shows one session as cards in five colours, with filters, a search and the model's prompts and responses.
kind: concept
area: extension
date: 2026-11-20
---

# The Sessions view

The **Sessions** view lists the session logs of each project, newest first. A [session](../concepts/sessions.md) is one conversation with CodeLead, from the chat or from a terminal, and its log records every line of it. A click on a session opens the **Session tab**: the commands on the left, and everything that happened on the right, as cards.

**In short**
- Each row is one folder `.codeleadsessions/session-<time>/`, labelled with its start time in UTC and its last command.
- The Session tab shows the commands, the events in five colours, filter chips, a search, and the model's prompts and responses.
- **Session log** in the chat opens the same tab for the chat's session.

![The Sessions view with two sessions, beside the Session tab of one of them. The tab's header gives the session's id, the status ended, and its counts. The Commands column lists /implement Add a wishlist service with a check mark. On the right are the five filter chips, the search box and the event cards in their colours.](../media/extension/sessions-tab.png)

## Why it exists

A model's answer can be checked only if its question was kept. The session log keeps both. The Sessions view lets you find a session without a terminal, and the Session tab lets you read it without opening raw JSON. A person who did not watch the session can follow it afterward.

## What the view shows

With one project, the sessions are at the top of the view. With several projects, each project is a row, and its sessions are under it.

Each session row shows:

- **The label**: the start time, in UTC, for example `2026-11-20 05:10Z`.
- **The description**: the last command, the number of commands and of model calls, and the status when the session did not end. For example: `/validate · 3 commands · 2 model calls`.
- **The tooltip**: the folder, relative to the project, and the description.

The row's file icon opens `session.jsonl` in the editor. When a project has no session, the view says:

```output
No sessions yet. Start one:
[Create an app]
[Implement a change]
```

The view lists only the project's own `.codeleadsessions/` folder. A new app built inside the project is a project of its own, with its own sessions.

## The Session tab

A click on a session opens the tab `Session <time>`. Its layout follows the Session Log Viewer. See [Read a session in the Session Log Viewer](../how-to/read-a-session.md).

### The commands

The left column lists what was typed in the session: each command, when it ran, how long it took, and its outcome as a mark. A click scrolls the timeline to that command and highlights it.

### The timeline

The right side shows every event of the session as a card. Each card has a colour and a label for its direction:

| Label | What it holds |
|---|---|
| **CodeLead → You** | What CodeLead wrote and asked. |
| **You → CodeLead** | What you typed, and your answers to its questions. |
| **CodeLead → Model** | Each request to a model. |
| **Model → CodeLead** | Each response, and each model error. |
| **System / Other** | The session's own events, such as its start and its end. |

The colour is never the only sign: each card also carries its label.

- **Filter chips** at the top show or hide each kind. **Reset** shows every kind again and clears the search.
- **Search the session…** keeps the cards whose text matches.
- **The content** of each card is rendered: Markdown, diffs, code, JSON and terminal output.
- **A prompt or a response** loads when you select its card. The extension reads the file only from inside the session's folder.
- **Raw JSON** on each card shows the event as CodeLead wrote it.
- **A long session** shows 300 rows at a time. **Show earlier** and **Show more** move through it.

When the session built a new app, a block above the timeline shows the plan's steps, their status and the patch attempts of each step.

> [!NOTE]
> **New to this?** A *prompt* is the text that CodeLead sends to the model. A *response* is what the model answers. CodeLead keeps both in the session's `model/` folder.

> [!WARNING]
> A session log holds the prompts sent to models, and so it holds parts of your code. Treat the folder as you treat the repository. See [Export a session](../how-to/export-a-session.md) before you share one.

## What you can do

- **Read the session of a chat.** In the chat's header, select **Session log**. The Session tab opens on the chat's session, live or older.
- **Find why a command failed.** Select the command on the left. Read the **Model → CodeLead** cards and the result of the command.
- **Export a session.** In the chat, type [`/session --export`](../chat-commands/session.md). See [Export a session](../how-to/export-a-session.md).

## Details

- **The folder.** `.codeleadsessions/session-<time>/` holds `session.jsonl` (one JSON event per line), `summary.json`, and the model calls under `model/`. See [Sessions](../concepts/sessions.md).
- **The order.** The view sorts by `startedAt` in `summary.json`, or by the time in the folder's name.
- **Refresh.** The view refreshes when CodeLead writes under `.codeleadsessions/`. The refresh button in the title bar reads it again.
- **Not committed.** CodeLead adds `.codeleadsessions/` to `.gitignore`.

## Related content

- [Sessions](../concepts/sessions.md)
- [The chat](chat.md)
- [`/session`](../chat-commands/session.md) and [`codelead sessions`](../cli/codelead-sessions.md)
- [Read a session in the Session Log Viewer](../how-to/read-a-session.md)
- [Export a session](../how-to/export-a-session.md)
- [Session protocol](../reference/session-io-protocol.md)
