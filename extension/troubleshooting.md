---
title: Troubleshoot the extension
description: The real failures of CodeLead for VS Code - the message you see, what it means, and the way out for each, from Initialize to the chat, the board, the plan and the Analyst.
kind: how-to
area: extension
date: 2026-11-20
---

# Troubleshoot the extension

This page lists the failures of CodeLead for VS Code by where they happen, each with the message that you see and the way out. Most failures come from three causes: the `codelead` command is not found, the folder is not trusted, or the model server does not answer.

**In short**
- Read the message first. CodeLead's messages name the cause and the way out.
- The output panel **CodeLead** holds the extension's log and CodeLead's error output. **CodeLead Analyst** holds the Analyst's.
- In the chat, a failure is a card with **Retry**. When the session itself stops, select **Restart** in the chat's header.

## Before you begin

- Run `codelead --version` in a terminal of the same folder. When it fails, CodeLead is not installed, or not on the PATH. See [Install CodeLead](../get-started/install.md). When it says `the build is older than the source`, CodeLead runs from a source checkout, and the extension runs an older build than the checkout's code. Run `npm run build` in the checkout.
- Run [`codelead doctor`](../cli/codelead-doctor.md) in the folder. It checks the machine, the configuration and the model server.
- Open the output panel: **View** > **Output**, then choose **CodeLead** in the list.

> [!NOTE]
> **New to this?** VS Code asks whether you trust a folder's authors. In an untrusted folder, the extension runs no `codelead` command. Select **Manage Workspace Trust** in the Command Palette to change it.

## Fix a folder that shows no project

| What you see | What it means | The way out |
|---|---|---|
| The welcome "This folder has no CodeLead files yet." | The folder holds no `.codelead/`, `.codeleadsessions/` or `docs/codelead/`. | Select **Initialize CodeLead in this folder**. Or open the folder that holds the files. |
| "CodeLead: trust this workspace before initializing it." | The folder is not trusted. | Trust the folder, then select **Initialize** again. |
| "CodeLead: open the folder you want to initialize first." | The window has no folder. | **File** > **Open Folder…**. |
| "CodeLead: init failed (exit 1): …" with "Config already exists at <your project>/.codelead/config.json." | The folder has its configuration already. | Nothing to do: the project is ready. Run `codelead init --force` only to write the defaults again: your changes to the file are lost. |
| A view stays empty after a change | The view did not refresh. | Select the refresh button in the view's title bar. |

A new app built inside a repository is found when it holds its own `.codelead/` or `.codeleadsessions/`, up to two folders deep.

## Fix a chat that does not start

The chat shows a bar above the box with CodeLead's words and **Restart**.

| Message | What it means | The way out |
|---|---|---|
| "CodeLead could not start: the command was not found: install CodeLead or set codelead.command" | The extension cannot find the `codelead` command. | Install CodeLead, or set `codelead.command` to the command's path. See [Commands and settings of the extension](commands.md#settings). Then select **Restart**. |
| "CodeLead stopped (exit 1): …" | CodeLead ended with an error. The last two lines of its error output follow. | Read the output panel **CodeLead**. Fix the cause, then select **Restart**. |
| "A remote model is configured but the privacy settings block remote models, so the session does not start. …" | A role uses a remote provider, and **Allow remote models** is off. | When the code's owner agrees, turn on **Allow remote models** on the Settings page, then **Restart**. Or set the roles to a local model. See [Privacy and remote models](../concepts/privacy-and-remote-models.md). |
| "The session ended." | The session ended normally, for example after `/exit`. | Select **Restart** for a fresh session. |
| "Not sent: the session is not running. Restart it to send." | You sent a line after the session ended. | Select **Restart**, then send it again. |

On Windows, the chat starts the `codelead` command through its npm file. Two more messages can come:

- "Chat needs an npm Node shim or an executable. Set codelead.command to the CodeLead npm command." Set `codelead.command` to the `codelead` command that npm installed.
- "The CodeLead npm shim target is missing. Reinstall CodeLead." Install CodeLead again with npm.

See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md) for the other Windows notes.

## Fix a command that fails in the chat

A command that stops short shows a red card with the reason, the ways out and **Retry**. See [The chat](chat.md#the-problem-card).

| Card or result | What it means | The way out |
|---|---|---|
| **The model call failed**, "Could not connect to lmstudio at http://localhost:1234/v1." | The model server does not run, or its address is wrong. | Start the model server. Check the address in **Settings** > **Model**. Then select **Retry**. |
| **The model call failed**, with a model's name that the provider does not know | The model name is not one of the provider's ids. | Select **Open Settings**, choose a model from the list, and select **Save**. Then **Retry**. |
| A result **Failed**, "Validation failed: npm test exited 1" | The change was applied, and your build or tests failed. | Select the **/repair** button, or fix the code by hand and send `/validate`. See [Repair](../concepts/repair.md). |
| A result **Stopped**, "The change was not applied: the answer was no; nothing is changed" | You answered no. | Send `/edit` to apply the checked patch later, or change the request. |

For each stop and its way out, see [Results and stops](../concepts/results-and-stops.md) and [Fix a stop](../how-to/fix-a-stop.md). For the full list of CodeLead's messages, see [Messages and what to do](../reference/messages.md).

## Fix a question that cannot be answered

| What you see | What it means | The way out |
|---|---|---|
| "This session ended with the window's reload. Its last question was declined; nothing was applied." | VS Code reloaded the window while CodeLead asked a yes-or-no question. No answer could come, so nothing was done. | Select **Open a new chat**, then run the command again. |
| "This session ended with the window's reload. Its last question was answered with the default." | VS Code reloaded the window while CodeLead asked. The old session took its default and ended. | Select **Open a new chat**. Check what the old session did in its log: **Session log** in the header. |
| "This session ended with the window's reload. Its last question can no longer be answered here." | The same, and the log does not show the answer. | Select **Open a new chat**, then run the command again. After `/modernize --plan`, run `/modernize --plan --intake` to answer the questions again. |
| "CodeLead: this chat could not be restored. Open its project in a trusted workspace, then open the chat again." | The restored tab's folder is not in the window, or not trusted. | Open the project's folder, trust it, and open the chat again. |
| The box says "Answer the question above" | A question waits for you. | Answer the card with the coloured border. Or select **Stop**: the question gets no answer. |

## Fix the model list in Settings

The **Model** field of the Settings page says why it has no list:

| Status line | The way out |
|---|---|
| "The server did not answer. Type the model's name." | Start the model server, check the **Address**, then reopen the page. Or type the name. |
| "The key is not set, so the server was not asked. Type the model's name." | Paste the key in **API key** and select **Save**. |
| "This folder is not trusted, so CodeLead is not run here. Type the model's name." | Trust the folder. |
| "The codelead command was not found (the codelead.command setting). Type the model's name." | Install CodeLead, or set `codelead.command`. |
| A saved model with "Not in the provider's list; CodeLead will refuse it if the provider does not know it" | Choose a model from the list, unless you know that the provider accepts the name. |

## Fix the board

| What you see | What it means | The way out |
|---|---|---|
| "CodeLead: the board's server could not start: …" | `codelead board serve` did not start. Its words follow. | Check that `codelead --version` works, or set `codelead.command`. |
| The board tab stays blank | The page did not load in the tab. | Reload the window: **Developer: Reload Window** in the Command Palette. Or select **Open in the browser** in the bar above the page. |
| A dash in the board's status bar item | The board check failed. The tooltip says why. | See the tooltip messages below. |
| "This project has no board yet." | The project has its configuration and no board. | Select **Set up the board in this folder**. |

The tooltip messages of the board check:

- "set codelead.boardParticipant or add a person to board.json": the board has no participant of kind `person`. Add yourself on the Settings page, in **Board**, or set `codelead.boardParticipant`.
- "board check timed out; check codelead.command": the check did not answer within 30 seconds.
- "the command was not found: install CodeLead or set codelead.command".

## Fix the plan when the pro views stay locked

The Analyst, Explorer and Modernization views show "… is in CodeLead's Pro plan" on the free plan.

1. Check that the project holds the licence: `docs/codelead/licence.json`.
2. Run [`codelead licence`](../cli/codelead-licence.md) in the folder. It prints the plan and the licence's warnings. `codelead licence --json` prints what the extension reads, for example `{ "state": "valid", "plan": "pro" }`.
3. Trust the folder. The extension does not read the plan in an untrusted folder, and keeps the plan that it had last time.
4. Select **Refresh** in any CodeLead view. The extension reads the plan again.

An older CodeLead without the `licence` command reads as free. [Update CodeLead](../how-to/update-codelead.md). The command **CodeLead: Modernize this application** says "CodeLead: modernization requires Pro." on the free plan. See [Licence and plans](../concepts/licence-and-plans.md).

## Fix the Analyst and the Explorer

| Message | What it means | The way out |
|---|---|---|
| "The Analyst was not found. Install CodeLead (it brings the Analyst), or set codelead.analystCommand." | Neither `codelead analyst` nor `analyst` could be found. | Select **Open the setting**, or install CodeLead. |
| "CodeLead Analyst: trust this workspace before running an Analyst command." | The folder is not trusted. | Trust the folder. |
| "CodeLead Analyst: exploration stopped. See the CodeLead Analyst output for the reason." | The Explorer ended with an error. | Read the output panel **CodeLead Analyst**. The application must run at the address that you gave. |
| "CodeLead Analyst: the Windows command shim cannot safely pass quotes, percent signs, exclamation marks or newlines. …" | An answer holds a character that the Windows `.cmd` file cannot pass. | Change the answer, or set `codelead.analystCommand` to a direct program. |
| A sign-in fails during a recording or an exploration | The role's password is not set, or wrong. | Set it in **Settings** > **Analyst and Explorer**. With **Integrated**, no password is needed for this computer's account. |

## Fix the Modernize wizard

The wizard says what stops it at the top of its page, for example "This CodeLead has no `modernize` command yet. Update CodeLead, or run the steps by hand:". See [The Modernize wizard](modernize-wizard.md#what-the-page-shows-when-it-cannot-work) for each message.

## Fix the other actions

| Message | The way out |
|---|---|
| "CodeLead: <project> has no run report yet. A build writes .codelead/run-report.html." | Build the app with [`/build-incremental`](../chat-commands/build-incremental.md). The step flow of `/create-app` writes no run report. See [New apps: steps, increments and the run report](../concepts/new-apps.md). |
| "CodeLead: the repository map could not be refreshed: exit 1; see the CodeLead output channel." | Read the output panel **CodeLead**, then run [`codelead map`](../cli/codelead-map.md) in a terminal to see the full message. |
| "CodeLead: no project in this workspace has that kind of artifact." | The project has nothing of that kind yet, for example no session or no board. |
| "CodeLead: a spec has a gap to fix; the CodeLead output says which and how (exit 1)." | Read the output panel **CodeLead**. Fix the spec, then run **Check the specs** again. See [`codelead spec`](../cli/codelead-spec.md). |

## Check that it worked

- The chat's header says **Ready**, and the new session's first turn shows CodeLead's welcome, which starts with "CodeLead Interactive".
- The status bar shows the model of the default role, and the board's count without a dash.
- `codelead doctor` in the folder reports no problem.

When a failure stays, export the session with [`/session --export`](../chat-commands/session.md) and read it, or share it inside your team. See [Export a session](../how-to/export-a-session.md).

## See also

- [Install the extension](install.md)
- [The chat](chat.md)
- [Settings](settings.md)
- [Results and stops](../concepts/results-and-stops.md)
- [Fix a stop](../how-to/fix-a-stop.md)
- [Messages and what to do](../reference/messages.md)
- [`codelead doctor`](../cli/codelead-doctor.md)
