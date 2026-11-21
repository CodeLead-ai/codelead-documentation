---
title: Work with agents on the board
description: Start a board in the Basic Store repository, write a card on the page, let a coding agent answer and propose a decision, record your decision, and queue the card for CodeLead.
kind: tutorial
area: tutorials
date: 2026-11-20
---

# Work with agents on the board

In this tutorial, you work with a coding agent through the **board**: a set of cards with threads, kept as files in the repository. You work on the board's page. The agent runs the board's commands in its own session. At the end, you hand a card to CodeLead, which works it like any other request.

**In short**
- You start a board with one command, and you open its page.
- You write a card on the page. The agent checks the board, answers you, and proposes a decision card.
- You record the decision on the page, and you send the card to CodeLead.
- CodeLead's queue takes the card, and writes what landed on it.

In this tutorial, you:

> [!div class="checklist"]
> - Start the board
> - Open the board's page
> - Write a card
> - Let the agent answer and propose
> - Record your decision
> - Send the card to CodeLead
> - Run CodeLead's queue

## Prerequisites

What you must know:

- You know how to open a terminal, and what a git commit is.
- You know how to start your coding agent and give it a sentence. You do not need to know any board command: the agent runs them.

What you must have:

- CodeLead installed. See [Install CodeLead](../get-started/install.md).
- The Basic Store sample as a git repository, with nothing waiting to commit.
- A coding agent that can run commands in a terminal of the repository, such as Claude Code or Codex.
- A browser.
- For the last step: a model, and a machine that builds the sample. On this sample, that is Windows with Visual Studio's MSBuild. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).
- About 20 minutes.

> [!NOTE]
> **New to this?** A *card* is one task, question, decision or idea, with a thread of comments. A *column* is where a card stands, from **Inbox** to **Done**. The *check* is how an agent sees what changed since it last looked. See [Cards, columns and participants](../pinboard/concepts.md).

## Start the board

1. Open a terminal in the folder of the sample.
2. Create the board with [`codelead board init`](../pinboard/cli/board-init.md). Name yourself with `--person` and the agent with `--agent`. Use your own short id and name in place of `sam:Sam`:

   ```console
   codelead board init --person sam:Sam --agent claude:Claude
   ```

   You see output similar to the following:

   ```output
   Board created in docs/codelead/board: board.json, AGENTS.md, events.jsonl, .gitignore, items/, cursors/.
   Participants: sam (person), claude (agent). Agents read docs/codelead/board/AGENTS.md.

   Tell the agent, once, in the file it reads at the start of every session (CLAUDE.md, a root AGENTS.md or the like):
     This repository has a Pinboard in docs/codelead/board. Read docs/codelead/board/AGENTS.md once per session, take part as `claude`, and run `codelead board check --as claude` at the start of the session and whenever asked; check again every 30 minutes while the session runs where you have a timer (Claude Code: `/loop 30m codelead board check --as claude`).
   Before that file says it, the first session starts with one sentence to the agent: "Read docs/codelead/board/AGENTS.md and check the board as claude."
   CodeLead joined the board as the participant codelead: tag a card for:codelead to hand it to CodeLead (codelead implement --queue takes those, top of Backlog first).
   ```

3. Copy the indented paragraph into the file that your agent reads at the start of every session, for example `CLAUDE.md`.
4. Commit the board:

   ```console
   git add docs/codelead/board CLAUDE.md
   git commit -m "board: init"
   ```

The board is now three kinds of file: `board.json`, one file for each card under `items/`, and the event log `events.jsonl`. `AGENTS.md` teaches the agent the board's rules.

## Open the board's page

1. Start the page with [`codelead board serve`](../pinboard/cli/board-serve.md), and leave it running:

   ```console
   codelead board serve --open
   ```

   ```output
   Pinboard: http://127.0.0.1:7414/
   The board in docs/codelead/board, as Sam. The page writes the board's files through the same rules as these commands; Ctrl+C stops it.
   ```

2. Look at the page in your browser. It shows the columns **Inbox**, **Backlog**, **Ideas**, **Pinned**, **In progress**, **For review** and **Done**. The dot in the top bar says **live**.

The page writes as you, the board's first person. In VS Code, the **Board** view opens the same page in a tab. See [Use the board](../pinboard/use-the-board.md).

## Write a card

1. Press `n`. A small form opens in **Inbox**.
2. Choose the type **task**.
3. Type the title: `Refuse an order with the same product on two lines`.
4. Type the description. Type `@claude` to address the agent:

   ```text
   Today the order form accepts the same product twice. @claude, what do you need to know before you start?
   ```

5. Select **Create card**, or press ⌘↵ (Ctrl+Enter on Windows and Linux).

The card lands in **Inbox** as `#1`. Its file is `docs/codelead/board/items/0001-refuse-an-order-with-the-same-product-on.md`.

## Let the agent answer and propose

1. Start a session of your agent in the repository.
2. Give it the one sentence that `init` printed, unless its start file says it already:

   ```text
   Read docs/codelead/board/AGENTS.md and check the board as claude.
   ```

3. Watch the agent's session. The agent checks the board. Your card comes first, because it names the agent:

   ```output
   claude: 1 new on 1 card since the board began (no cursor yet), 1 addressed to you.

   Addressed to you (1)

   #1 Refuse an order with the same product on two lines  (task · Inbox)
     sam · 2026-11-20T07:55:19Z  created it (task in Inbox)

   Run again with --ack once these are dealt with. Next check in 30 minutes.
   ```

4. Read what the agent does next. It answers in the card's thread, and it asks you a question as a decision card, with its recommendation. In its session, you see commands and lines such as these:

   ```console
   codelead board reply 1 --as claude --text "@sam, one question first: must the rule also apply when a person edits an order? I asked it on card #2, with my recommendation."
   ```

   ```output
   Commented on #1 as claude (2026-11-20T07:55:20Z).
   ```

   ```console
   codelead board new --as claude --type decision --title "Apply the duplicate-product rule to edits too?" --field ask="Refuse a duplicate product when a person edits an order, or only when they create one? I recommend both: the same form serves Create and Edit." --field options='["Create and Edit", "Create only"]' --relate related:1 --tag repo:webapp
   ```

   ```output
   Created #2 (decision) in Inbox: Apply the duplicate-product rule to edits too?
   docs/codelead/board/items/0002-apply-the-duplicate-product-rule-to.md
   ```

   Your agent writes its own words. The steps are the same.

5. Look at the page. Card `#1` has a flag: the last comment is the agent's, so the card waits on you. Card `#2` has a dashed border and says **proposed**. The bell shows the new changes.

> [!TIP]
> When the agent says that `claude` is not a participant, it used another id. Tell it the id that `init` named, or add the agent to `participants` in `board.json`. See [Work with agents](../pinboard/work-with-agents.md).

## Record your decision

An agent never answers a decision. Only a person records it.

1. Open card `#2`. Its question is at the top, under **Asked of you**.
2. In the **Decided** row, select **Record the decision**.
3. Choose the option `Create and Edit`. Check that **by** is you, and **on** is today.
4. Select **Record the decision**.

The card's face now shows **Decided:** and your answer. Check the card's file with a read-only command, [`codelead board show`](../pinboard/cli/board-show.md):

```console
codelead board show 2
```

```output
#2 · decision · Inbox
Apply the duplicate-product rule to edits too?
tags: repo:webapp, by:claude
by claude · created 2026-11-20T07:55:21Z · updated 2026-11-20T07:55:23Z
file: items/0002-apply-the-duplicate-product-rule-to.md
relations:
  related #1 (Inbox) Refuse an order with the same product on two lines
fields:
  ask: Refuse a duplicate product when a person edits an order, or only when they create one? I recommend both: the same form serves Create and Edit.
  options: ["Create and Edit","Create only"]
  decided: {"option":"Create and Edit","by":"sam","on":"2026-11-20"}

Thread (0)
```

## Send the card to CodeLead

1. Drag card `#1` to **Backlog**. CodeLead's queue takes Backlog first.
2. Open card `#1`, and select **Send to CodeLead**. The page adds the tag `for:codelead`, and a blue **CodeLead** badge shows on the card.
3. Ask the agent to check the board again. Its check lists your decision first:

   ```output
   claude: 3 new on 2 cards since 2026-11-20T07:55:22Z, 1 decided.

   Decided (1)

   #2 Apply the duplicate-product rule to edits too?: decided: Create and Edit (by sam, 2026-11-20)

   Also new

   #1 Refuse an order with the same product on two lines  (task · Backlog)
     sam · 2026-11-20T07:55:26Z  moved it from Inbox to Backlog
     sam · 2026-11-20T07:55:27Z  tagged it for:codelead

   Run again with --ack once these are dealt with. Next check in 30 minutes.
   ```

4. The agent writes the decision into the task's thread, so that CodeLead reads it with the card. It does not take the card: `AGENTS.md` tells it that a card tagged `for:codelead` is CodeLead's.

## Run CodeLead's queue

1. In a terminal in the folder of the sample, see what the queue would do, with [`codelead implement`](../cli/codelead-implement.md) `--dry-run`. Nothing is run or written:

   ```console
   codelead implement --queue --dry-run
   ```

   <!-- described, not captured -->
   ```output
   Dry run of codelead implement --queue: the checks passed; nothing is run or written.
   Board: docs/codelead/board.
   Model: the default role, the preflight role, the fast strategy, the repair role, the clarify strategy, the architect strategy, the deep strategy: lmstudio (the server's loaded model).
   The queue (the cards tagged for:codelead, top of Backlog first), in order: 1 card.
     1. #1 "Refuse an order with the same product on two lines" (Backlog)
   Each card: one session driven with /implement --card <id>, then up to 2 repair rounds, then /exit; the session writes the card's trail as codelead.
   Caps: no caps; up to 2 repair rounds per card; the stop file .codelead/STOP and Ctrl+C stop the run after the card in flight.
   Lock: free: a run started now takes it.
   ```

2. Start the queue:

   ```console
   codelead implement --queue
   ```

   CodeLead reads the card as the request: its title, its description, the comments and the related decision. It writes `Started:` on the card, and moves it to **In progress**. At the end, it writes `Landed:` and moves the card to **For review**. When the change does not land, it writes `Stopped:` with the reason and the ways on, addressed to you.

   <!-- described, not captured -->
   ```output
   #1 "Refuse an order with the same product on two lines": landed, evidence record ev-…, 12 min; now in For review.
   Implement run (the queue): 1 card worked, 1 landed, 12 min. Stopped: no card is tagged for:codelead.
   ```

3. On the page, open card `#1`. Read the `Landed:` comment and the change. When you accept the work, move the card to **Done**. Only a person moves a card to Done.

> [!TIP]
> When the card ends with `Stopped:`, read its reason in the thread. Answer there, move the card back to **Backlog**, still tagged, and run the queue again. For the stops themselves, see [Fix a stop](../how-to/fix-a-stop.md).

## Clean up

1. Stop the page: press **Ctrl+C** in the terminal of `codelead board serve`.
2. Keep the board when the team will use it: commit it with the cards.
3. To remove it, delete the folder `docs/codelead/board`, remove the paragraph from `CLAUDE.md`, and commit the removal.

## Next steps

> [!div class="nextstepaction"]
> [Work with agents](../pinboard/work-with-agents.md)

- [Use the board](../pinboard/use-the-board.md): the page, the marks, the filters and the keys
- [Rules for agents](../pinboard/rules-for-agents.md): what `AGENTS.md` tells an agent
- [`codelead implement`](../cli/codelead-implement.md): the queue's caps and its stop file
- [Change existing code with evidence](change-existing-code.md)
