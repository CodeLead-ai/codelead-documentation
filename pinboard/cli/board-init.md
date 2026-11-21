---
title: codelead board init
description: Create the board folder in a repository - board.json, AGENTS.md, events.jsonl, .gitignore, items/ and cursors/ - and print what to tell the agent.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board init

**Applies to:** CodeLead 0.1 and later

`codelead board init` creates the board in the repository you are in. It writes the board's files, names the participants, and prints the paragraph that you give to your coding agent.

## Name

`codelead board init` - Creates the board folder in this repository.

## Synopsis

```console
codelead board init [--person <ID[:NAME]>]... [--agent <ID[:NAME]>]... [--name <NAME>] [--board <DIR>] [--json]
codelead board init -h|--help
```

## Description

Run `init` once, in the repository. It creates `docs/codelead/board` with:

- `board.json`: the seven columns, the six card types, the participants and the agents' limits.
- `AGENTS.md`: the guide that teaches an agent the board's rules.
- `events.jsonl`: the event log, empty.
- `.gitignore`: keeps the cursors, the lock, `private-names.txt` and `.local/` out of git.
- `items/` and `cursors/`: empty folders.

`init` never overwrites a file. When `board.json` is there already, it refuses and writes nothing.

> [!NOTE]
> **New to this?** After `init`, two steps remain. Put the printed paragraph in the file that your agent reads at the start of every session, and commit the board folder. See [Work with agents](../work-with-agents.md).

### The participants

Without `--person`, the person is this machine's user name, in lower case. Without `--agent`, the agent is `claude`, named "Claude Code". A participant is written `id` or `id:Name`, such as `sam:Sam`. An id is one word of letters, digits, `.`, `_` and `-`.

### CodeLead joins the board

When the board is in the default folder, CodeLead joins it as the agent `codelead` and says so on the last line. A card tagged `for:codelead` is then work for CodeLead. With `--board` and another folder, CodeLead does not join.

### What to tell the agent

The output ends with a paragraph for the agent's start-of-session file, and the one sentence that starts the first session. With one agent, the paragraph names it. With several agents, it says "take part under your own id".

## Options

- **`--person <ID[:NAME]>`**

  A person of the board. Repeat it for several people. The default is this machine's user.

- **`--agent <ID[:NAME]>`**

  An agent of the board. Repeat it for several agents. The default is `claude:Claude Code`.

- **`--name <NAME>`**

  The board's name. The default is `<repository folder>-workspace`.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints `{ "dir", "written", "tellTheAgent": { "paragraph", "firstSentence" } }`.

## Examples

- Create a board with yourself and the default agent:

  ```console
  codelead board init
  ```

  ```output
  Board created in docs/codelead/board: board.json, AGENTS.md, events.jsonl, .gitignore, items/, cursors/.
  Participants: <your name> (person), claude (agent). Agents read docs/codelead/board/AGENTS.md.

  Tell the agent, once, in the file it reads at the start of every session (CLAUDE.md, a root AGENTS.md or the like):
    This repository has a Pinboard in docs/codelead/board. Read docs/codelead/board/AGENTS.md once per session, take part as `claude`, and run `codelead board check --as claude` at the start of the session and whenever asked; check again every 30 minutes while the session runs where you have a timer (Claude Code: `/loop 30m codelead board check --as claude`).
  Before that file says it, the first session starts with one sentence to the agent: "Read docs/codelead/board/AGENTS.md and check the board as claude."
  CodeLead joined the board as the participant codelead: tag a card for:codelead to hand it to CodeLead (codelead implement --queue takes those, top of Backlog first).
  ```

- Name the person and two agents:

  ```console
  codelead board init --person sam:Sam --agent claude:Claude --agent codex:Codex
  ```

  ```output
  Board created in docs/codelead/board: board.json, AGENTS.md, events.jsonl, .gitignore, items/, cursors/.
  Participants: sam (person), claude (agent), codex (agent). Agents read docs/codelead/board/AGENTS.md.

  Tell the agent, once, in the file it reads at the start of every session (CLAUDE.md, a root AGENTS.md or the like):
    This repository has a Pinboard in docs/codelead/board. Read docs/codelead/board/AGENTS.md once per session, take part under your own id (`claude` for Claude, `codex` for Codex), and run `codelead board check --as <you>` at the start of the session and whenever asked; check again every 30 minutes while the session runs where you have a timer (Claude Code: `/loop 30m codelead board check --as <you>`).
  Before that file says it, the first session starts with one sentence to the agent: "Read docs/codelead/board/AGENTS.md and check the board under your own id."
  CodeLead joined the board as the participant codelead: tag a card for:codelead to hand it to CodeLead (codelead implement --queue takes those, top of Backlog first).
  ```

- Create a second board in another folder, with its own name:

  ```console
  codelead board init --board other-board --name store-notes --person sam:Sam
  ```

  ```output
  Board created in other-board: board.json, AGENTS.md, events.jsonl, .gitignore, items/, cursors/.
  Participants: sam (person), claude (agent). Agents read other-board/AGENTS.md.
  …
  ```

  CodeLead does not join a board outside the default folder.

- Run it where a board exists. Nothing is written:

  ```console
  codelead board init
  ```

  ```output
  A board is already at <your project>/docs/codelead/board (board.json is there); nothing was written.
  ```

- Give an id that is not one word. Nothing is written:

  ```console
  codelead board init --person "sam lee"
  ```

  ```output
  Refused: that board would not be valid: participant 1 must be { "id": <word>, "kind": "person" or "agent", "name": <text> }. Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The board was created. |
| `1` | A board is already there. |
| `2` | The board would not be valid, for example an id that is not one word. |

## Files

`init` writes `board.json`, `AGENTS.md`, `events.jsonl`, `.gitignore`, `items/` and `cursors/` in the board folder. See [Files and events](../files-and-events.md).

> [!TIP]
> **For experienced users:** with `--json`, the output is one JSON document, and a script can parse all of it. CodeLead's line "CodeLead joined the board…" then goes to the error stream.

## See also

- [Work with agents](../work-with-agents.md): give the board to an agent.
- [`codelead board serve`](board-serve.md): open the board's page.
- [`codelead board check`](board-check.md): the agent's first command.
- [Files and events](../files-and-events.md)
- [The Board view](../../extension/board-view.md): **Set up the board in this folder** runs this command.
- [`codelead board`](../../cli/codelead-board.md)
