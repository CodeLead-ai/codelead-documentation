---
title: "Quickstart: change existing code"
description: Make one small change to an existing repository with CodeLead, from the request to the evidence record, in about ten minutes.
kind: quickstart
area: get-started
date: 2026-11-20
---

# Quickstart: change existing code

In this quickstart, you make one small change to an existing application with [`/implement`](../chat-commands/implement.md). You see each step of the loop: the preflight, the patch, the patch check, your approval, the gate, and the evidence record. It takes about ten minutes.

**In short**
- You prepare the repository with [`codelead init`](../cli/codelead-init.md), which also reads it into its map.
- You run one `/implement` and answer its questions with Enter.
- At the end, the change has landed, and an evidence record says what happened.

In this quickstart, you:

> [!div class="checklist"]
> - Prepare the repository
> - Read its map
> - Make one change with /implement
> - Read the evidence record

## Prerequisites

- CodeLead installed, and a model that answers. See [Install CodeLead](install.md).
- A git repository to change. The examples use the Basic Store sample: an ASP.NET MVC 5 application on .NET Framework 4.5, with clients, products and orders.
- For the Basic Store sample: Windows with Visual Studio 2019 or later, or its Build Tools with the web development build tools workload. The sample's build needs Visual Studio's MSBuild. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).
- You know how to open a terminal and use git. You do not need to know C# for this quickstart.

> [!TIP]
> You can use your own repository instead, on any system. Choose a repository whose build and tests run from the command line, and write a request of the same size.

## Prepare the repository

CodeLead keeps its settings and its records inside the repository, so each project is prepared once.

1. Open a terminal in the repository's folder. Make sure that git has no uncommitted changes: `git status`.
2. Run:

   ```console
   codelead init
   ```

   On the Basic Store sample, you see output similar to the following:

   ```output
   ✔ Created <your project>/.codelead/config.json
   <your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
   Repository map written: 1 module, 89 files; docs/codelead/repository-map.md
   Encyclopedia: 4 entries from the map (4 new, 0 updated, 0 downgraded); evidence record ev-20261120045758-skyvwz.
   ```

   CodeLead wrote its configuration and read the repository into its [repository map](../concepts/repository-map.md). It also seeded the [knowledge base](../concepts/knowledge-base.md) and wrote a first [evidence record](../concepts/evidence-records.md), of the read.

## Read the map

The map is CodeLead's summary of the repository. It is read from the code, with no model.

1. Run:

   ```console
   codelead map
   ```

   ```output
   Repository map is fresh: 1 module, 89 files; docs/codelead/repository-map.md
   Entry points: WebApplication1/webapp.csproj
   Stamp: git bd30c0ac3c5f (uncommitted changes), 89 files scanned, written 2026-11-20T04:57:58.426Z
   ```

2. Open `docs/codelead/repository-map.md`. It names the stack, the validation commands and the modules. The validation commands are the [gate](../concepts/validation-and-the-gate.md): the commands that decide whether a change is done.

## Make one change

1. Start a session:

   ```console
   codelead interactive
   ```

2. Type the request. It names the file and the exact words to change:

   ```console
   /implement In WebApplication1/Controllers/OrdersController.cs, change the message after an order is created from "Order {0} was created." to "Order {0} was saved."
   ```

3. The preflight runs. When it asks a question, read it, then press Enter for the recommended answer, or type your own.

4. The patch and the patch check follow. CodeLead shows the check's result and asks you:

   <!-- described, not captured -->
   ```output
   Step 2/6: Generating patch from preflight...
   Step 2/6: Patch generated.
   Step 3/6: Checking patch...
   Step 5/6: Patch check passed.
   …
   The patch check passed. CodeLead recommends applying.

   Apply this patch? [Y/n]
   ```

   The capital `Y` is the recommended answer. Press Enter to apply the patch. See [The patch check](../concepts/the-patch-check.md).

5. CodeLead asks to run the gate:

   <!-- described, not captured -->
   ```output
   Run validation now? [Y/n]
   ```

   Press Enter. On the Basic Store sample, the gate builds `sampleapp.slnx` with Visual Studio's MSBuild.

6. Read the result, on the last line:

   <!-- described, not captured -->
   ```output
   Evidence record: <your project>/.codelead/evidence/ev-….json
   ✔ The change landed: applied to 1 file and validation passed
   ```

   The change is a **landing**: it was applied, and the gate passed.

> [!NOTE]
> On macOS or Linux, the gate of the Basic Store sample cannot run, and the result says so: `✖ Validation failed: MSBuild could not run: Visual Studio's MSBuild is needed and was not found`. The patch is applied, but the gate could not build the project. Use Windows for this sample, or your own repository. See [Validation and the gate](../concepts/validation-and-the-gate.md).

## Read the evidence record

The evidence record is the file that says what happened, without the session.

1. Open the file that the result names, in `.codelead/evidence/`. Its main fields look like this:

   <!-- described, not captured -->
   ```json
   {
     "schema": "codelead.evidence/1",
     "command": "implement",
     "request": "In WebApplication1/Controllers/OrdersController.cs, change the message …",
     "outcome": "landed",
     "surface": { "files": ["WebApplication1/Controllers/OrdersController.cs"], "additions": 1, "removals": 1 },
     "patch_check": { "status": "can_apply", "problems": [], "warning_count": 0 },
     "apply": { "status": "applied", "files_changed": 1 },
     "validation": { "status": "passed", "commands": [ { "command": "…", "status": "passed", "exitCode": 0 } ] }
   }
   ```

2. Read it from top to bottom: the request, the outcome, the file that changed, the check's verdict, and the gate's commands with their exit codes.
3. Leave the session with `/exit`, and look at the change with `git diff`.

## Clean up

To undo the change, run `git checkout -- WebApplication1/Controllers/OrdersController.cs`. To remove CodeLead from the repository, delete the `.codelead` folder and `docs/codelead`, and remove the lines that `codelead init` added to `.gitignore`.

## Next steps

- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md): a larger change, a failing gate, a repair and a pull request.
- [How CodeLead works](../concepts/how-codelead-works.md)
- [Evidence records](../concepts/evidence-records.md)
- [Quickstart: CodeLead in VS Code](quickstart-vscode.md)
