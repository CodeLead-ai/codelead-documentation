---
title: Commands and settings of the extension
description: Every command that CodeLead for VS Code adds to the Command Palette and to its views, when each one shows, and every codelead.* setting with its default.
kind: reference
area: extension
date: 2026-11-20
---

# Commands and settings of the extension

**Applies to:** CodeLead for VS Code 0.7.16 and later

## Description

CodeLead for VS Code adds commands under the category **CodeLead**. Most of them are in the Command Palette, and also on a button or a menu of a view. Some show only on a view's row, because they act on that row. The extension also adds four settings, `codelead.command`, `codelead.boardParticipant`, `codelead.analystCommand` and `codelead.runIn`.

To open the Command Palette, press Ctrl+Shift+P, or Cmd+Shift+P on macOS. Type `CodeLead` to list the commands. The extension adds no keyboard shortcuts.

> [!NOTE]
> **New to this?** A command shows in the Command Palette only when it can act. For example, **CodeLead: Implement a change** shows only when a folder of the window holds CodeLead files. The column "Shows when" says the condition.

## Commands in the Command Palette

| Command | Shows when | What it does | Also on |
|---|---|---|---|
| **CodeLead: Initialize CodeLead in this folder** | Always | Runs `codelead init`, then `codelead board init`, in the folder that you choose. See [Install the extension](install.md#initialize-a-folder). | The welcome of the Projects view. |
| **CodeLead: Set up the board in this folder** | Always | Runs `codelead board init`. | The welcome of the Board view. |
| **CodeLead: Refresh** | Always | Reads the projects, the views and the open pages again, and reads each project's plan again. | The refresh button of each view. |
| **CodeLead: Open CodeLead chat** | A folder is open | Opens the chat of the project that you choose. See [The chat](chat.md). | A project's click and right-click menu in Projects, **Start** > **Open the chat**, the chat tab's title bar. |
| **CodeLead: Create an app** | A folder is open | Asks what the app must do, and sends `/create-app --auto <words>`. | **+** in Projects, **Start** > **Create an app**. |
| **CodeLead: Implement a change** | A project exists | Offers the cards tagged `for:codelead`, or a box for your words, and sends `/implement`. | **…** in Projects, **Start** > **Implement a change**. |
| **CodeLead: Check the specs** | A project exists | Runs `codelead spec check`. The output goes to the output panel **CodeLead**. | **…** in Projects. |
| **CodeLead: Write the spec report** | A project exists | Runs `codelead spec report`, then opens `.codelead/spec-report.html` in your browser. | **…** in Projects. |
| **CodeLead: Open the run report** | A project exists | Opens `.codelead/run-report.html` in your browser. | **…** in Projects. |
| **CodeLead: Change the model** | A project exists | Opens the Settings page. | **…** in Projects, the model in the status bar. |
| **CodeLead: Open CodeLead Settings** | A project exists | Opens the Settings page at the top. See [Settings](settings.md). | The gear of the Settings view. |
| **CodeLead: Read the repository again** | A project exists, in a trusted folder | Runs `codelead map --refresh`. See [The Knowledge base view](knowledge-base-view.md#the-repository-map-node). | The pencil of **Repository map**. |
| **CodeLead: Open the board** | A project exists | Opens the board page of the project that you choose. See [The Board view](board-view.md). | A project or a column in Board, the board in the status bar. |
| **CodeLead: Open Card…** | A project exists | Asks a card number, and opens the board page at that card. | A card in Board. |
| **CodeLead: Record a journey** | Pro plan | Asks three questions, then runs `codelead analyst record` in a terminal. See [The Analyst view](analyst-view.md). | **+** in Analyst and on its project rows. |
| **CodeLead: Draft from a recording** | Pro plan | Runs `codelead analyst draft` on a recording, and opens the Confirm tab. | A recording's right-click menu, in Analyst and in the VS Code Explorer. |
| **CodeLead: Check journeys** | Pro plan | Runs `codelead analyst check` on a `journeys.json`. | A draft's right-click menu. |
| **CodeLead: Replay** | Pro plan | Asks the address, then runs `codelead analyst replay` in a terminal. | A draft's right-click menu. |
| **CodeLead: Rules** | Pro plan | Runs `codelead analyst rules` on one or more recordings. | A recording's right-click menu. |
| **CodeLead: Facts** | Pro plan | Runs `codelead analyst facts` on one or more recordings. | A recording's right-click menu. |
| **CodeLead: Confirm** | Pro plan | Opens the Confirm tab of a draft. | A draft's or a confirmation's right-click menu. |
| **CodeLead: Explore an application** | A project exists, in a trusted folder, pro plan | Asks four questions, then runs `codelead analyst explore`. See [The Explorer view](explorer-view.md). | **+** in Explorer and on its project rows. |
| **CodeLead: Modernize this application** | Pro plan | Opens the Modernize wizard. See [The Modernize wizard](modernize-wizard.md). | The rocket in Modernization. |

The Analyst commands, **Read the repository again**, Initialize and the wizard refuse to run in an untrusted folder. In an untrusted folder, the **+** of the Analyst and Explorer views, their right-click items, and the pencil of **Repository map** are hidden. **Read the repository again** and **Explore an application** are also hidden in the Command Palette. A hidden command still refuses when you run it another way, such as from a keyboard shortcut.

## Commands on a view only

These commands have no entry in the Command Palette. A view's row or button runs them, with the row as its target.

| Command | ID | Where it is | What it does |
|---|---|---|---|
| **Open Session** | `codelead.openSession` | A session in Sessions. | Opens the Session tab. |
| **Open Evidence Record** | `codelead.openEvidence` | A record in Knowledge base, the document icon on a card's record. | Opens the record's page. |
| **Open Knowledge Ledger** | `codelead.openKnowledge` | An entry of the encyclopedia. | Opens the knowledge ledger page. |
| **Open Run Journal** | `codelead.openJournal` | **Run journal** in Projects. | Opens the run journal page. |
| **Open Modernization Page** | `codelead.openModernizationPage` | A document in Modernization. | Opens its page. |
| **Open Recording** | `codelead.openRecording` | A recording in Analyst or Explorer. | Opens the recording page. |
| **Open Artifact in Editor** | `codelead.openArtifact` | The file icon on a row. | Opens the row's file in the editor. |
| **Edit** | `codelead.editSetting` | The pencil on a row of the Settings view. | Opens the row's edit dialog. |
| **Model Settings** | `codelead.openModelSettings` | The gear of Projects. | Opens the Settings page at **Model**. |
| **Board Settings** | `codelead.openBoardSettings` | The gear of Board. | Opens the Settings page at **Board**. |
| **Analyst Settings** | `codelead.openAnalystSettings` | The gear of Analyst and of Explorer. | Opens the Settings page at **Analyst and Explorer**. |
| **Open Section Settings** | `codelead.openSectionSettings` | Used by the three commands above. | Opens the Settings page at a named section. |

## The IDs of the palette commands

| Command | ID |
|---|---|
| Initialize CodeLead in this folder | `codelead.initialize` |
| Set up the board in this folder | `codelead.initializeBoard` |
| Refresh | `codelead.refresh` |
| Open CodeLead chat | `codelead.openChat` |
| Create an app | `codelead.createApp` |
| Implement a change | `codelead.implement` |
| Check the specs | `codelead.specCheck` |
| Write the spec report | `codelead.specReport` |
| Open the run report | `codelead.openRunReport` |
| Change the model | `codelead.changeModel` |
| Open CodeLead Settings | `codelead.openSettings` |
| Read the repository again | `codelead.refreshRepositoryMap` |
| Open the board | `codelead.openBoard` |
| Open Card… | `codelead.openCard` |
| Record a journey | `codelead.analyst.record` |
| Draft from a recording | `codelead.analyst.draft` |
| Check journeys | `codelead.analyst.check` |
| Replay | `codelead.analyst.replay` |
| Rules | `codelead.analyst.rules` |
| Facts | `codelead.analyst.facts` |
| Confirm | `codelead.analyst.confirm` |
| Explore an application | `codelead.analyst.explore` |
| Modernize this application | `codelead.modernize.wizard` |

Use an ID in a keyboard shortcut of your own, in VS Code's `keybindings.json`.

## Settings

Each setting has the scope `resource`: you can set it for the user, for the workspace, or for one folder of a workspace. Open **File** > **Preferences** > **Settings**, and search for `codelead`.

- **`codelead.command`**

  The CodeLead command that the extension runs. The chat, Initialize, the board, the plan, the model lists, the spec commands, the wizard and the map use it. It is one program, by name on the PATH or by its path. The default is `codelead`.

- **`codelead.boardParticipant`**

  The board participant that the status bar checks for. When it is empty, the extension takes the first participant of kind `person` in `board.json`. The default is empty.

- **`codelead.analystCommand`**

  The Analyst command that the extension runs. Quote paths with spaces. The default is empty. Empty means: run `codelead analyst` with the command of `codelead.command`, then `analyst` on the PATH, then the build of an open Analyst workspace. A value that is not empty is run as you wrote it, `analyst` included. The **Analyst command** row of the [Settings](settings.md#analyst-and-explorer) page shows the command that the extension resolves.

- **`codelead.runIn`**

  Where **Create an app** and **Implement a change** run. `chat`, the default, sends their line into the project's chat. `terminal` types it into a terminal named **CodeLead**, and you press Enter there.

## Context keys

The extension sets two context keys. VS Code reads them in the conditions of menus and the Command Palette. You can use them in your own `keybindings.json`.

| Key | Values | Meaning |
|---|---|---|
| `codelead.hasProjects` | `true`, `false` | A folder of the window holds `.codelead/`, `.codeleadsessions/` or `docs/codelead/`. |
| `codelead.plan` | `free`, `pro` | The window's plan. It is `pro` when any project of the window is on the pro plan. See [Licence and plans](../concepts/licence-and-plans.md). |

> [!TIP]
> **For experienced users:** the views have no condition of their own. On the free plan, all eight views stay listed, and the pro views show a lock row. The condition `codelead.plan == 'pro'` hides their buttons and their Command Palette entries.

## Examples

- Run CodeLead from a path, when `codelead` is not on the PATH. In the workspace's `.vscode/settings.json`:

  ```json
  {
    "codelead.command": "<path to the codelead command>"
  }
  ```

- Type **Create an app** and **Implement a change** into a terminal instead of the chat:

  ```json
  {
    "codelead.runIn": "terminal"
  }
  ```

  The terminal **CodeLead** starts `codelead interactive`, and 2 seconds later the line appears in it, for example `/implement add a due date to tasks`. Press Enter to start it.

- Check the board for a participant other than the first person:

  ```json
  {
    "codelead.boardParticipant": "<your participant id>"
  }
  ```

- Bind a key to the chat, in `keybindings.json`:

  ```json
  [
    { "key": "ctrl+alt+l", "command": "codelead.openChat", "when": "codelead.hasProjects" }
  ]
  ```

## See also

- [A tour of the extension](tour.md)
- [Settings](settings.md)
- [The chat](chat.md)
- [Command-line reference](../cli/index.md)
- [Chat command reference](../chat-commands/index.md)
- [Troubleshoot the extension](troubleshooting.md)
