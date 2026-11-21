---
title: "Quickstart: build a new app"
description: Build a small task manager app from one sentence, in an empty folder, with one command in a CodeLead session.
kind: quickstart
area: get-started
date: 2026-11-20
---

# Quickstart: build a new app

In this quickstart, you build a small task manager app from one sentence. CodeLead plans the app, then builds it step by step. Each step is a patch that passes the patch check and the step's validation before the next step starts. With a fast model, it takes about ten minutes.

**In short**
- You prepare an empty folder with [`codelead init`](../cli/codelead-init.md).
- You type one command, [`/create-app --auto`](../chat-commands/create-app.md), in a CodeLead session.
- At the end, the app builds, its checks pass, and `npm run dev` opens it.

In this quickstart, you:

> [!div class="checklist"]
> - Prepare an empty folder
> - Start a session
> - Build the app with one command
> - Open the app

## Prerequisites

- CodeLead installed, and `codelead doctor` with no failed required check. See [Install CodeLead](install.md).
- A model that answers: a local model server, or a remote provider with your consent.
- You know how to open a terminal. You do not need to know React or any other framework.

## Prepare an empty folder

A new app starts in an empty folder, so that nothing of another project is in the way.

1. Make the folder and go into it:

   ```console
   mkdir task-manager
   cd task-manager
   ```

2. Prepare it for CodeLead:

   ```console
   codelead init
   ```

   You see output similar to the following:

   ```output
   ✔ Created <your project>/.codelead/config.json
   <your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
   Repository map: nothing to read yet (no source files); it is written when the first build finishes.
   ```

   CodeLead wrote its configuration, and it keeps `.env` out of git. The folder has no code yet, so there is no [repository map](../concepts/repository-map.md) to read.

## Start a session

A [session](../concepts/sessions.md) is a conversation with CodeLead about one folder.

1. Start it:

   ```console
   codelead interactive
   ```

   You see the session's banner and its prompt:

   ```output
   ╭──────────────────────────────────────────────────────────────────────────────╮
   │                                                                              │
   │   CodeLead Interactive                                                       │
   │                                                                              │
   │   Repo: <your project>                                                       │
   │   Strategy: auto                                                             │
   │   Project type: greenfield                                                   │
   │   Ask about the code in plain words (the chat never changes code), or use    │
   │   /help for commands.                                                        │
   │   Use /exit or /quit to leave.                                               │
   │                                                                              │
   ╰──────────────────────────────────────────────────────────────────────────────╯
   codelead>
   ```

   "Project type: greenfield" means a new app in an empty folder.

## Build the app with one command

1. Type the request after `/create-app --auto`:

   ```console
   /create-app --auto a task manager app with a list of tasks and a done checkbox
   ```

   With `--auto`, CodeLead plans the app, approves the plan and builds every step, without stopping. It answers its own questions. The checks still run at every step.

2. Watch the plan. A model reads the request and writes a project specification (what the app does) and an execution plan (the steps that build it):

   <!-- described, not captured -->
   ```output
   Create App Plan
   Request: a task manager app with a list of tasks and a done checkbox

   Step 1/4: Analyzing request...
   Step 2/4: No clarifications required.
   Step 3/4: Project specification generated.
   Step 4/4: Execution plan generated.
   Project Specification created.
   Execution Plan created.
   …
   ```

3. Watch the steps. The first step usually runs a scaffold command, such as `npm create vite`, and installs the dependencies. Each later step writes a patch, checks it, applies it and runs its validation:

   <!-- described, not captured -->
   ```output
   …
   ✅ 👍 Step 2 of 6 completed. Validation passed.
   …
   ```

4. Read the end. When every step is done, the session says so:

   <!-- described, not captured -->
   ```output
   Automated build completed: all plan steps are done. Run `npm run dev` in the target folder to open the app.
   Repository map written: …; docs/codelead/repository-map.md
   ```

   CodeLead also wrote the repository map of the new app.

> [!TIP]
> When the model server does not answer, the plan stops at once. Nothing is created:
>
> ```output
> Step 1/4: Analyzing request...
> Failed to create greenfield plan
> Could not connect to lmstudio at http://127.0.0.1:9/v1.
> If this is LM Studio, make sure it is open and the Local Server is running.
> ```
>
> The address is the one that your configuration names. Start the model server, then send the same command again. For other stops, see [Fix a stop](../how-to/fix-a-stop.md).

### See where the build stands

While the build runs, and after it ends, you can ask where it stands:

- [`/greenfield --status`](../chat-commands/greenfield.md) shows the plan, the progress of each step and the next command.
- [`/build-step --show`](../chat-commands/build-step.md) shows the current step, its plan and its validation.

`/create-app` builds by steps, and the session prints each step. To build by increments, with a git commit after each verified increment and a live page that shows the build, use [`/build-incremental`](../chat-commands/build-incremental.md). That page is the run report, `.codelead/run-report.html`. The [tutorial](../tutorials/build-a-new-app.md) builds the same app that way.

## Open the app

1. Leave the session:

   ```console
   /exit
   ```

2. Start the app:

   ```console
   npm run dev
   ```

3. Open the address that the command prints, for example `http://localhost:5173`, in a browser. Add a task, and mark it done.

## Clean up

To remove the app, delete the `task-manager` folder. Nothing outside the folder was changed.

## Next steps

- [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md): the same app in increments, with the gates, a repair and the run report.
- [Quickstart: change existing code](quickstart-existing-code.md)
- [Tutorial: start a new app from a template](../tutorials/start-from-a-template.md)
- [How CodeLead works](../concepts/how-codelead-works.md)
