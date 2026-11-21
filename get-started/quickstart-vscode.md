---
title: "Quickstart: CodeLead in VS Code"
description: Install the CodeLead extension, initialize a folder, connect a model, and send one request in the chat, in about ten minutes.
kind: quickstart
area: get-started
date: 2026-11-20
---

# Quickstart: CodeLead in VS Code

In this quickstart, you install CodeLead for VS Code, prepare a folder with one click, and use the chat. The chat runs the same CodeLead session as the terminal, with buttons for the questions and the results. It takes about ten minutes.

**In short**
- You install the extension from a `.vsix` file, and CodeLead's icon appears in the activity bar.
- **Initialize CodeLead in this folder** prepares the folder. Nothing happens without that click.
- In the chat, a question gets an answer from your code. A request for a change gets the command to run, with **Yes** and **No**.

In this quickstart, you:

> [!div class="checklist"]
> - Install the extension
> - Initialize a folder
> - Connect a model
> - Ask a question and make one change in the chat

## Prerequisites

- VS Code 1.95 or newer. **Help** > **About** shows the version.
- CodeLead installed, so that `codelead --version` works in a terminal. See [Install CodeLead](install.md).
- A model: a local model server, or a key for a remote provider.
- A folder to work in: the Basic Store sample, your own repository, or an empty folder for a new app.
- You know how to open a folder in VS Code. You do not need to know CodeLead's commands: the chat offers them.

## Install the extension

The extension comes as one file, named like `codelead-vscode-<version>.vsix`.

1. Open a terminal in the folder that holds the file.
2. Install it:

   ```console
   code --install-extension codelead-vscode-<version>.vsix
   ```

   VS Code says that the extension was successfully installed.

You can also install it from VS Code. Open the **Extensions** view, and select the **…** menu at the top of the view. Select **Install from VSIX…**, and pick the file. See [Install the extension](../extension/install.md).

## Initialize a folder

1. Open the folder in VS Code: **File** > **Open Folder…**.
2. Select the **CodeLead** icon in the activity bar on the left.
3. In the **Projects** view, select **Initialize CodeLead in this folder**.

   The extension runs [`codelead init`](../cli/codelead-init.md), then [`codelead board init`](../pinboard/cli/board-init.md). They write `.codelead/config.json` and keep `.env` out of git. They read the repository into its [repository map](../concepts/repository-map.md), and set up the project's board in `docs/codelead/board`.

> [!NOTE]
> **New to this?** The extension changes nothing in a folder until you select **Initialize**. Before that, the views are empty, and that is expected.

## Connect a model

1. Open **Settings** with the gear in the CodeLead views.
2. Choose the provider, the address and the model.
   - For a local server, keep the default address, `http://localhost:1234/v1`, or type the address of your server.
   - For a remote provider, paste the key once. CodeLead keeps it in the folder's `.env` file and never shows it again. Turn on **Allow remote models** only when the code's owner agrees.
3. Select **Save** for each change. A wrong value is refused next to its field.

See [Settings](../extension/settings.md) and [Privacy and remote models](../concepts/privacy-and-remote-models.md).

## Ask a question and make one change

1. In the **Projects** view, select your project. A tab named **CodeLead · <your project>** opens. This is [the chat](../extension/chat.md).
2. Type a question in plain words, and press Enter. For example, on the Basic Store sample:

   ```text
   how is the order total calculated?
   ```

   The chat answers from your code, and it names the files and lines that its answer rests on. The chat never changes code.

3. Type a request for a change:

   ```text
   In OrdersController.cs, change the message after an order is created from "Order {0} was created." to "Order {0} was saved."
   ```

   The chat answers with the command that it would run, an [`/implement`](../chat-commands/implement.md), and asks **Run it now?** with **Yes** and **No**. Select **Yes**.

4. Answer each question as it comes. Each question shows the recommended answer, and the button for it is highlighted. See [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md).
   - **Apply this patch?** comes after [the patch check](../concepts/the-patch-check.md). Select **Yes** when the check passed.
   - **Run validation now?** runs the [gate](../concepts/validation-and-the-gate.md): your project's build and tests. Select **Yes**.

5. Read the result at the end of the turn. It shows the outcome, one sentence, a few facts and the next commands as buttons. **Show details** opens the full output. When the change landed, the result says "The change landed".

In an empty folder, a request such as `a task manager app with a list of tasks and a done checkbox` starts a new app with [`/create-app --auto`](../chat-commands/create-app.md).

> [!TIP]
> When a command fails, the chat shows a card with the reason, the ways out and a **Retry** button. When the chat says "Session ended", select **Restart** in its header. See [Troubleshooting](../extension/troubleshooting.md).

## Clean up

To remove the extension, run `code --uninstall-extension codelead.codelead-vscode`, or select **Uninstall** in the **Extensions** view. The extension removes nothing from your project. To undo the change, use `git checkout` on the file.

## Next steps

- [A tour of the extension](../extension/tour.md): every view, in one page.
- [The chat](../extension/chat.md)
- [Quickstart: change existing code](quickstart-existing-code.md): the same loop in a terminal, step by step.
- [How CodeLead works](../concepts/how-codelead-works.md)
