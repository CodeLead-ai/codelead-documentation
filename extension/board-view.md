---
title: The Board view
description: The project's board in the side bar and in a tab - its columns and cards, the board page, the cards tagged for CodeLead, and the status bar count of what waits for you.
kind: concept
area: extension
date: 2026-11-20
---

# The Board view

The **Board** view shows the project's board: its columns and the cards in each. The board is [Pinboard](../pinboard/index.md), the board between you and your coding agents, kept as files in `docs/codelead/board/`. A click on a column or a card opens the board's page in a tab, where you read and answer the cards.

**In short**
- The view lists each column with its count, and each card with its number, title and type. A card tagged `for:codelead` says `· CodeLead`.
- A click opens the board page in a tab, `Board — <project>`, at the card that you selected.
- The status bar shows how many cards have something new for you, and how many name you.
- The extension starts the board's local server for you, and stops it with the window.

![The Board view of a sample project beside the tab Board — shop-api. The view lists the columns Inbox with 3 cards, In progress with 1 and For review with 1, and each card's type. Above the board page, a bar gives Open in the browser and the address. The page shows the cards of Inbox and In progress, and For review folded with its count.](../media/extension/board-view-tab.png)

## Why it exists

Work that waits on a person, such as a question, a decision or a review, gets lost in chat histories. The board keeps it in one place, as cards that people and agents both read. The view shows the cards beside the code, and the status bar tells you when something is new for you. See [Cards, columns and participants](../pinboard/concepts.md).

## What the view shows

The view lists each project that holds `docs/codelead/board/board.json`. Under a project:

- **Each column**, with its card count: `Backlog · 4`. A column with no card still shows, so that an empty board has something to select.
- **Each card**: `#<number> <title>`, and `· CodeLead` when the card is tagged `for:codelead`. The description is the card's type: `question`, `decision`, `task`, `idea`, `review` or `note`, each with its own icon. The tooltip is the card's description.
- **A problem** with the board's files, as a row of its own.

A click on the project or on a column opens the board page. A click on a card opens the page at that card. A right-click on the project or on a column offers **Open the board**. The file icon on a card opens its file in the editor.

## The board page

The board page opens in a tab named `Board — <project>`. A bar above it gives the page's address, as **Open in the browser —** `http://127.0.0.1:…`. It also says "If the board stays blank, reload the window".

The page is the board itself. On it, you read a card and its thread, answer, move cards between columns, and set tags. A card has **Send to CodeLead**, which tags it `for:codelead`. See [Use the board](../pinboard/use-the-board.md).

> [!NOTE]
> **New to this?** A *card* is one piece of work or one question, with a thread of replies. A *column* says its state, such as Backlog or Done. A *participant* is a person or an agent that takes part.

### Cards for CodeLead

A card tagged `for:codelead` is in CodeLead's queue. **Implement a change** in the [Projects view](projects-view.md) offers these cards, Backlog first. Cards in In progress, For review or Done are not offered. CodeLead moves a card that it takes to In progress, and to For review when the change lands. See [Work with agents](../pinboard/work-with-agents.md).

In the chat, `/implement --card <id>` takes the request from a card, and the completion row offers the cards. See [`/implement`](../chat-commands/implement.md).

## The status bar

A speech bubble in the status bar shows two numbers, for example `3 · @1`:

- The first number counts the cards with something new for you.
- The second, after `@`, counts the cards that name you.

A click opens the board. The tooltip lists each project, its counts, and up to five card titles. A dash instead of the numbers means that the check failed, and the tooltip says why.

The extension runs `codelead board check --as <you> --json` in each project with a board. It checks again when a board file changes, and at the board's check interval, every 30 minutes by default. The check never marks anything as read.

> [!TIP]
> **For experienced users:** "you" is the setting `codelead.boardParticipant`. When it is empty, the extension takes the first participant of kind `person` in `board.json`. See [Commands and settings of the extension](commands.md).

## The title bar and the welcome

- **Refresh** reads the board again.
- **The gear**, **Board Settings**, opens the Settings page at the **Board** section: the people of the board and the check interval. See [Settings](settings.md#board).

A project with CodeLead's configuration but no board shows this welcome:

```output
This project has no board yet. The board is where you and CodeLead keep the cards: requests, questions, what landed.
[Set up the board in this folder]
```

The button runs [`codelead board init`](../pinboard/cli/board-init.md) in the project.

## Details

- **The server.** The extension runs `codelead board serve --port 0 --json` in the project, and opens the address that it prints. The server answers only on this machine. It lives as long as the window, and the extension stops a server left by an older window.
- **One tab for each project.** Opening the board again shows the same tab, at the new card.
- **Open Card….** In the Command Palette, **CodeLead: Open Card…** asks for a card number, then opens the page at that card.
- **The files.** The board is `docs/codelead/board/`: `board.json`, the cards under `items/`, `events.jsonl`, and the agents' guide `AGENTS.md`. See [Files and events](../pinboard/files-and-events.md).

## Related content

- [Pinboard](../pinboard/index.md)
- [Use the board](../pinboard/use-the-board.md)
- [Work with agents](../pinboard/work-with-agents.md)
- [`codelead board`](../cli/codelead-board.md), [`codelead board check`](../pinboard/cli/board-check.md) and [`codelead board serve`](../pinboard/cli/board-serve.md)
- [`/implement`](../chat-commands/implement.md)
- [Tutorial: work with agents on the board](../tutorials/work-with-agents-on-the-board.md)
