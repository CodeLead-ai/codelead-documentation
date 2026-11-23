---
title: Install the extension
description: Install CodeLead for VS Code from its .vsix file, initialize a folder with one click, and update or remove the extension.
kind: how-to
area: extension
date: 2026-11-20
---

# Install the extension

This page installs CodeLead for VS Code from its `.vsix` file, prepares a folder for CodeLead, and says how to update and remove the extension. The extension needs CodeLead itself on the machine, because every button runs the `codelead` command.

**In short**
- You install the `.vsix` file with `code --install-extension`, or with **Install from VSIX…** in the Extensions view.
- **Initialize CodeLead in this folder** runs `codelead init`, then `codelead board init`. Nothing happens in a folder without this click.
- To update, install the newer file in the same way. To remove, uninstall the extension: your project keeps all its files.

## Before you begin

- **VS Code 1.95 or newer.** **Help** > **About** shows the version.
- **CodeLead, installed.** In a terminal, `codelead --version` must print a version. See [Install CodeLead](../get-started/install.md).
- **The extension's file.** It is named like `codelead-vscode-0.7.16.vsix`.
- **A trusted folder.** VS Code asks whether you trust the authors of a folder. The extension runs `codelead` only in a trusted folder.

> [!NOTE]
> **New to this?** A `.vsix` file is a VS Code extension in one file. Teams use it for an extension that is not in the public Marketplace.

## Install the file

Choose one of the two ways.

**From a terminal:**

1. Open a terminal in the folder that holds the file.
2. Run this command:

   ```console
   code --install-extension codelead-vscode-0.7.16.vsix
   ```

   ```output
   Extension 'codelead-vscode-0.7.16.vsix' was successfully installed.
   ```

**From VS Code:**

1. Open the **Extensions** view (the four squares in the activity bar).
2. Select the **…** menu at the top of the view.
3. Select **Install from VSIX…**, and pick the file.

The CodeLead icon, a `[CL]` mark, appears in the activity bar. If VS Code asks you to reload the window, reload it.

## Open the CodeLead views

1. Open a folder: **File** > **Open Folder…**.
2. Select the **CodeLead** icon in the activity bar.

The side bar shows the CodeLead views. See [A tour of the extension](tour.md). In a folder with no CodeLead files, the **Projects** view shows this welcome:

```output
This folder has no CodeLead files yet. Initialize CodeLead to start: it writes `.codelead/config.json`, sets up the board in `docs/codelead/board`, and keeps your `.env` out of git. Nothing happens without this click.
[Initialize CodeLead in this folder]
Already have the files? Open the folder that holds them.
```

The other views say "No CodeLead files in this folder yet. Initialize CodeLead from the Projects view above."

## Initialize a folder

Initialize prepares the folder for CodeLead. You do it once for each project.

1. In the **Projects** view, select **Initialize CodeLead in this folder**.
2. When the window has several folders, choose the folder in the list **Choose the folder to initialize**.

The extension runs two commands in the folder, one after the other: [`codelead init`](../cli/codelead-init.md), then [`codelead board init`](../pinboard/cli/board-init.md). Their words appear in the output panel **CodeLead**. Here is the output for a small task manager app:

```output
✔ Created <your project>/.codelead/config.json
<your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
Repository map written: 2 modules, 2 files; docs/codelead/repository-map.md
Encyclopedia: 6 entries from the map (6 new, 0 updated, 0 downgraded); evidence record ev-20261120061313-skh45o.
Board created in docs/codelead/board: board.json, AGENTS.md, events.jsonl, .gitignore, items/, cursors/.
Participants: <your name> (person), claude (agent). Agents read docs/codelead/board/AGENTS.md.
…
CodeLead joined the board as the participant codelead: tag a card for:codelead to hand it to CodeLead (codelead implement --queue takes those, top of Backlog first).
```

The two commands do these things:

- `codelead init` writes `.codelead/config.json` with the default settings.
- It adds `.env`, `.env.local` and `.env.*.local` to `.gitignore`, so that your keys stay out of git.
- It reads the repository into its [repository map](../concepts/repository-map.md), seeds the [knowledge base](../concepts/knowledge-base.md), and writes one [evidence record](../concepts/evidence-records.md) of the read.
- `codelead board init` sets up the project's board in `docs/codelead/board`. See [Pinboard](../pinboard/index.md).

When both commands end, the views fill. **Set up the board in this folder** runs only `codelead board init`. It is for a project that has its configuration but no board. You find it in the **Board** view and in the Command Palette.

> [!TIP]
> **For experienced users:** **Create an app** in a folder without `.codelead/config.json` asks first: "CodeLead: <folder> has no .codelead/config.json yet. Initialize CodeLead first: it writes that file." Select **Initialize CodeLead** in that message, and the action continues.

## Connect a model

CodeLead needs a model for most commands. Open the Settings page: the gear in the **Settings** view, or the gear in the **Projects** view. Choose the provider, the address and the model, and select **Save**. For a remote provider, paste the key: CodeLead keeps it in the folder's `.env` and never shows it again. See [Settings](settings.md).

## Update the extension

1. Get the newer `.vsix` file.
2. Install it in the same way as the first time. VS Code replaces the old version. You do not uninstall first.
3. If VS Code asks you to reload the window, reload it.

An open chat comes back after the reload with its history. Its session belonged to the old window, so the chat shows a notice and the button **Open a new chat**. See [The chat](chat.md#a-restored-chat).

## Remove the extension

Run this command, or select **Uninstall** on CodeLead in the **Extensions** view:

```console
code --uninstall-extension codelead.codelead-vscode
```

The extension removes nothing from your project. The files that CodeLead wrote stay: `.codelead/`, `.codeleadsessions/` and `docs/codelead/`.

## Check that it worked

- The **CodeLead** icon is in the activity bar.
- After Initialize, the **Projects** view lists your folder with its **Start** row.
- The status bar shows the model of the default role: a robot icon with the provider and the model, for example `lmstudio · qwen/qwen3.6-35b-a3b`. Right after Initialize, it shows the provider `lmstudio` and no model name yet. Without a default role, it says `model not set`.
- The status bar also shows the board's count: a speech bubble with the cards that have something new for you.

When something is missing, see [Troubleshoot the extension](troubleshooting.md).

## Details

- **When the extension starts.** VS Code starts it when the folder holds `.codelead`, `.codeleadsessions`, `docs/codelead` or `analyst-recordings`. It also starts when you open its views, or when it restores a chat tab.
- **The plan.** In a trusted folder, the extension runs `codelead licence --json` in each project, in the background. It reads the plan again on **Refresh**, when the folders of the window change, and when `docs/codelead/licence.json` changes. An answer that takes more than 10 seconds, or a CodeLead that does not know the command, counts as free. The window starts with the plan that it had last time.
- **The command.** The extension runs the command in the setting `codelead.command`, `codelead` by default. On Windows, the global command is a `.cmd` file, and the extension starts it in a way that works with it. See [Commands and settings of the extension](commands.md).
- **Workspace trust.** In an untrusted folder, Initialize, the plan read, the repository map's refresh and the Analyst commands refuse to run.

## See also

- [Quickstart: CodeLead in VS Code](../get-started/quickstart-vscode.md)
- [A tour of the extension](tour.md)
- [Settings](settings.md)
- [Install CodeLead](../get-started/install.md)
- [`codelead init`](../cli/codelead-init.md) and [`codelead board init`](../pinboard/cli/board-init.md)
- [Update CodeLead](../how-to/update-codelead.md)
- [Troubleshoot the extension](troubleshooting.md)
