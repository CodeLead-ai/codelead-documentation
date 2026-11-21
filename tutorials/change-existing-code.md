---
title: Change existing code with evidence
description: Add one business rule to the Basic Store sample, from its conventions to a pull request, through a failing gate and one repair round.
kind: tutorial
area: tutorials
date: 2026-11-20
---

# Change existing code with evidence

In this tutorial, you change the Basic Store sample, an ASP.NET MVC 5 application on .NET Framework 4.5. You add one business rule to its order form. CodeLead writes the change as a [patch](../concepts/requests-plans-and-patches.md), checks it, applies it after your yes, and runs [the gate](../concepts/validation-and-the-gate.md): the project's own build. You follow the change from the request to a pull request, and you read the evidence that CodeLead keeps at each step.

**In short**
- You read the conventions of the code, and you add one rule of your own.
- You run one change, answer its questions, and read the patch check with a note.
- You see the gate fail, and you fix the change with one repair round.
- You read the evidence records, and you open a pull request whose body is the evidence.

In this tutorial, you:

> [!div class="checklist"]
> - Prepare the sample for CodeLead
> - Read the conventions and add a team rule
> - Check the request before the run
> - Run the change and answer the questions
> - Read the patch check and apply the patch
> - See the gate refuse the change
> - Repair the change
> - Read the evidence records
> - Open a pull request

## Prerequisites

What you must know:

- You know git: clone, commit and push.
- You can read a little C#. You do not need to know ASP.NET MVC.

What you must have:

- Windows, with Visual Studio 2019 or later, or the Visual Studio Build Tools with the web development build tools workload. The gate builds the sample with Visual Studio's MSBuild. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).
- CodeLead installed. See [Install CodeLead](../get-started/install.md).
- A model. That is a local model server, or a key for a remote provider. See [Set up local models](../how-to/set-up-local-models.md) or [Set up remote models](../how-to/set-up-remote-models.md).
- A copy of the Basic Store sample, as a git repository: the project `webapp` and the solution `sampleapp.slnx`.
- For the last step: an empty repository on GitHub that you can push to, and the GitHub command-line tool `gh`, signed in with `gh auth login`.
- About 45 minutes.

> [!NOTE]
> On macOS or Linux, the gate cannot build this sample. CodeLead stops before the build and says why: `Validation failed: MSBuild could not run: Visual Studio's MSBuild is needed and was not found`. The way on is to run it on Windows, or to declare the commands in `.codelead/validation.json`.

## Prepare the sample for CodeLead

[`codelead init`](../cli/codelead-init.md) prepares a folder for CodeLead. It writes the settings, keeps your `.env` file out of git, and reads the repository into its [repository map](../concepts/repository-map.md). It calls no model.

1. Open a terminal in the folder of the sample:

   ```console
   cd basic-store
   ```

1. Check that git has nothing waiting:

   ```console
   git status
   ```

   The last line says `nothing to commit, working tree clean`.

1. Prepare the folder:

   ```console
   codelead init
   ```

   You see output similar to the following:

   ```output
   ✔ Created <your project>/.codelead/config.json
   <your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
   Repository map written: 1 module, 89 files; docs/codelead/repository-map.md
   Encyclopedia: 4 entries from the map (4 new, 0 updated, 0 downgraded); evidence record ev-20261120045758-skyvwz.
   ```

   The map also seeds the [knowledge base](../concepts/knowledge-base.md). The read gets its first [evidence record](../concepts/evidence-records.md): a file that records what happened.

1. Check the machine:

   ```console
   codelead doctor
   ```

   The line that starts with `MSBuild` must say `ok`. The line `Model server at …` must say `ok` too. See [`codelead doctor`](../cli/codelead-doctor.md).

1. Commit the setup, so that the pull request at the end holds only your change:

   ```console
   git add -A
   git commit -m "Set up CodeLead"
   ```

## Read the conventions and add a team rule

New code must look like the code around it. CodeLead reads the habits of the code into a convention profile, and sends them with every change. Your own written rules come first. See [Conventions](../concepts/conventions.md).

1. Read the conventions of the sample:

   ```console
   codelead conventions
   ```

   You see output similar to the following:

   ```output
   PROJECT CONVENTIONS

   Extracted from 31 sampled source files (C#, JavaScript); .codelead/conventions.json holds the evidence per rule.

   Formatting:
   - C# files are indented with 4 spaces (e.g. WebApplication1/App_Start/BundleConfig.cs)
   - Statements end with semicolons [97% of 309] (e.g. WebApplication1/Scripts/app/order-form.js)

   Naming:
   - Mixed: private fields are `_camelCase` in 50% of cases, private fields are `camelCase` with no underscore otherwise — follow the file you edit (e.g. WebApplication1/Controllers/AccountController.cs)
   - Async methods do not carry an `Async` suffix [94% of 49] (e.g. WebApplication1/Controllers/OrdersController.cs)
   …
   Idioms:
   - Namespaces are block-scoped (`namespace X { … }`) (e.g. WebApplication1/App_Start/BundleConfig.cs)
   - Locals are declared with `var` [97% of 88] (e.g. WebApplication1/App_Start/IdentityConfig.cs)
   - Members use block bodies, not `=>` (e.g. WebApplication1/App_Start/BundleConfig.cs)
   …
   - Control statements always take braces, on their own line (e.g. WebApplication1/App_Start/IdentityConfig.cs)
   …
   16 rules from 31 sampled files. Written to <your project>/.codelead/conventions.json.
   ```

   Each rule names a file that shows it. A share such as `[97% of 88]` says how many cases follow the rule. A rule marked `Mixed` asks the model to follow the file that it edits.

1. Create the file `.codelead/CONVENTIONS.md` with one rule of your team:

   ```markdown
   # Conventions

   - Error messages that a person reads are full sentences that end with a period.
   ```

1. Read the conventions again:

   ```console
   codelead conventions
   ```

   The rule of your team now comes first:

   ```output
   PROJECT CONVENTIONS

   Extracted from 31 sampled source files (C#, JavaScript); .codelead/conventions.json holds the evidence per rule.

   Team rules from .codelead/CONVENTIONS.md (these outrank everything extracted below):
     # Conventions

     - Error messages that a person reads are full sentences that end with a period.

   Formatting:
   …
   ```

1. Commit the rule:

   ```console
   git add .codelead/CONVENTIONS.md
   git commit -m "Add a team convention"
   ```

> [!TIP]
> **For experienced users:** the patch check tests the added lines against the extracted rules that are not mixed. It writes a note for each rule that a line breaks. Your written rules reach the model as words. See [Write your conventions](../how-to/write-conventions.md).

## Check the request before the run

A precise request gives a precise patch. Name the file, the method and the exact words of the error. [`codelead lint`](../cli/codelead-lint.md) checks a request against the repository before any model call.

1. Create the file `request.txt` beside the folder of the sample, with this text:

   ```text
   In WebApplication1/Controllers/OrdersController.cs, in ValidateOrderAsync, refuse an order that has the same product on two lines. Show the error "Each product can appear on one line only." on the second line's product field.
   ```

1. Check the request:

   ```console
   codelead lint --request ..\request.txt
   ```

   You see output similar to the following:

   ```output
   Request lint (the request, the repository, the System Model and the journey spec; no model call)

   Files the request names:
   - WebApplication1/Controllers/OrdersController.cs: exists

   Test files it names, and those beside the files it names:
   - none

   Libraries it assumes:
   - none named

   Model nodes it touches: no System Model (docs/codelead/rebuild/system-model.json).

   Spec steps that express its ask: no journey spec (docs/codelead/rebuild/journeys.json); the ask can be judged by observation only.

   Lines the request should carry:
   - none
   ```

   The file that the request names exists. The sample has no tests, so the gate of this change is the build.

## Run the change and answer the questions

[`/implement`](../chat-commands/implement.md) runs the whole path of one change: the preflight, the patch, the patch check, the apply and the validation. It stops to ask you before each step that changes something. See [Requests, plans and patches](../concepts/requests-plans-and-patches.md).

1. Start a session:

   ```console
   codelead interactive
   ```

   The session starts and shows the prompt `codelead>`.

1. Type `/implement`, a space and the request, and press **Enter**:

   ```console
   /implement In WebApplication1/Controllers/OrdersController.cs, in ValidateOrderAsync, refuse an order that has the same product on two lines. Show the error "Each product can appear on one line only." on the second line's product field.
   ```

1. Read the first question of the preflight. A preflight is the step where a model reads the request and the code, and writes a plan. When something is not clear, it asks. You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Implement
   Request: In WebApplication1/Controllers/OrdersController.cs, in ValidateOrderAsync, refuse an order …
   Step 1/6: Running preflight...
   Clarification 1/2

   Should the rule also apply when a person edits an existing order?

   Why it matters:
   ValidateOrderAsync runs for both Create and Edit.

   Default:
   Yes, for both Create and Edit.

   Answer, press Enter to use default:
   ```

   Your questions differ, because the model writes them. The `Default` is the answer that CodeLead recommends.

1. Press **Enter** to take the default.

1. Read the assumptions. CodeLead lists the choices that it makes unless you say otherwise:

   <!-- described, not captured -->
   ```output
   Clarification 2/2

   CodeLead assumes:
   1. The error goes on the second line's ProductId field, through LineField(line, "ProductId"). (risk: low)
   2. No file other than OrdersController.cs changes. (risk: low)

   Press Enter to accept them all; answer no to reject them all; or name the number and what you want instead (for example "1: resume the plan").

   Risk:
   low

   Default:
   yes

   Press Enter to accept, answer no, or say what you want instead:
   ```

1. Press **Enter** to accept the assumptions.

   CodeLead writes the patch and checks it. You see the lines `Step 2/6: Generating patch from preflight...` and `Step 3/6: Checking patch...`.

> [!TIP]
> When the model server does not answer, the run stops after step 1: `The change did not land: unable to complete preflight`. Run `codelead doctor`, start the server, and run the same `/implement` again. See [Fix a stop](../how-to/fix-a-stop.md).

## Read the patch check and apply the patch

[The patch check](../concepts/the-patch-check.md) is CodeLead's own reading of the patch against your files, with no model. A problem blocks the patch. A note does not block it: it is something for you to read.

1. Read the apply block. You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Patch check passed.
   Apply checked patch

   Files: 1
   Hunks: 2
   Additions: 7
   Removals: 0

   Notes (they do not block applying):
   - [conventions] WebApplication1/Controllers/OrdersController.cs: a control statement without braces: `if (!seenProducts.Add(line.ProductId.Value)) ModelState.AddM` — the code base's rule: Control statements always take braces, on their own line (e.g. WebApplication1/App_Start/IdentityConfig.cs)

   This will modify files in your working tree.
   The patch check passed with 1 note. CodeLead recommends applying.

   Apply this patch? [Y/n]
   ```

1. Read the note. The patch adds an `if` without braces, and the code of the sample always uses braces. The note names the rule and a file that shows it.

1. Press **Enter** to apply the patch. A note never changes the recommended yes.

   You see `Patch applied successfully.`, then the question `Run validation now? [Y/n]`.

> [!NOTE]
> **New to this?** "Apply" means that CodeLead writes the changes into your files. Before this moment, nothing in your folder changed. The note stays in the evidence record, so a reviewer sees it too.

## See the gate refuse the change

The patch check does not compile the code. [The gate](../concepts/validation-and-the-gate.md) does: it runs the project's own build. A change is done only when the gate passes.

1. Press **Enter** to run the validation.

1. Read the verdict. You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Validation
   Validation scope: repo root
   Running 1/1: MSBuild sampleapp.slnx -restore -nologo -v:minimal
   Validation

   Status: failed
   Scope: repo root

   Failed command:
     MSBuild sampleapp.slnx -restore -nologo -v:minimal

   Exit code: 1

   Output preview:
     <your project>\WebApplication1\Controllers\OrdersController.cs(258,13): error CS8370: Feature 'target-typed object creation' is not available in C# 7.3. Please use language version 9.0 or greater. [<your project>\WebApplication1\webapp.csproj]
   …
   Implementation applied, but validation failed.

   Next:
   - Review validation output.
   - Run /repair to generate a minimal fix patch.
   - If you manually verified the change, use /validate --override.
   …
   Evidence record: <your project>/.codelead/evidence/ev-….json
   ✖ The change was applied, but validation failed: MSBuild exited 1; next: /repair
   ```

1. Read the cause. The patch wrote `HashSet<int> seenProducts = new();`. That form needs C# 9. A .NET Framework project compiles with C# 7.3 by default.

1. Read the last line. It is the [result](../concepts/results-and-stops.md) of the command: the outcome, the headline and the way on. Here the way on is `/repair`.

## Repair the change

A repair round is a second, smaller change that fixes what the first one broke. It has three commands: [`/repair`](../chat-commands/repair.md) asks for the patch, [`/edit`](../chat-commands/edit.md) applies it, and [`/validate`](../chat-commands/validate.md) judges it. See [Repair](../concepts/repair.md).

1. Ask for the repair:

   ```console
   /repair
   ```

   CodeLead sends the model the failure, the file that the failure names and the change so far. Then it checks the repair's patch. You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Repair
   Using last failed validation result and current changed files.
   Generating minimal repair patch...
   …
   ✔ Repair round 1 — the repair for MSBuild is ready to apply (1 file); nothing is changed yet; next: /edit
   ```

1. Look at the repair's patch:

   ```console
   /patch --show
   ```

   <!-- described, not captured -->
   ```output
   …
   Patch preview:
   --- a/WebApplication1/Controllers/OrdersController.cs
   +++ b/WebApplication1/Controllers/OrdersController.cs
   @@ -255,7 +255,7 @@
   -            HashSet<int> seenProducts = new();
   +            var seenProducts = new HashSet<int>();
   …
   ```

   The new line also follows the convention `Locals are declared with var`.

1. Apply the repair:

   ```console
   /edit
   ```

1. Press **Enter** at `Apply this patch? [Y/n]`. The result says:

   <!-- described, not captured -->
   ```output
   ✔ Repair round 1 — applied the repair to 1 file; not validated yet; next: /validate
   ```

1. Run the gate again:

   ```console
   /validate
   ```

1. Press **Enter** at `Run these commands? [Y/n]`. The result says:

   <!-- described, not captured -->
   ```output
   ✔ Validation passed after repair round 1: 1 command
   ```

   The change has landed: it is applied, and the gate passed.

> [!TIP]
> **For experienced users:** a repair that brings a file back to a state that an earlier round tried is a cycle. CodeLead stops the loop at the second cycle and says what each round tried. The `repair` role can send repairs to a stronger model. See [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md).

## Read the evidence records

An evidence record is a file that says what was asked, what changed and what the checks said. CodeLead writes it from what happened, never from a model's words. One record follows the other, so a reviewer can follow the whole change.

1. Close the session:

   ```console
   /exit
   ```

1. List the records:

   ```console
   Get-ChildItem .codelead\evidence
   ```

   You see one file for each run, named `ev-<date and time>-<six characters>.json`.

1. Open the record of the `/implement`. It stopped, because the gate refused the change. Trimmed, it looks like this:

   <!-- described, not captured -->
   ```json
   {
     "id": "ev-20261120101512-k3p9qa",
     "schema": "codelead.evidence/1",
     "created_at": "2026-11-20T10:15:12.204Z",
     "workspace": "<your project>",
     "who": { "name": "<your name>", "hash": "<hash of your git email>" },
     "command": "implement",
     "request": "In WebApplication1/Controllers/OrdersController.cs, in ValidateOrderAsync, refuse an order …",
     "outcome": "stopped",
     "stop_reason": "validation_failed",
     "preflight": { "strategy": "clarify", "clarifications": 2, "defaults_used": 2 },
     "retries_used": 0,
     "surface": { "files": ["WebApplication1/Controllers/OrdersController.cs"], "additions": 7, "removals": 0 },
     "patch_check": { "status": "can_apply", "problems": [], "warning_count": 1 },
     "apply": { "status": "applied", "files_changed": 1 },
     "validation": {
       "status": "failed",
       "commands": [{ "command": "<MSBuild.exe> sampleapp.slnx -restore -nologo -v:minimal", "status": "failed", "exitCode": 1 }]
     },
     "conventions": { "conformance_warnings": ["[conventions] WebApplication1/Controllers/OrdersController.cs: a control statement without braces: …"] },
     "knowledge_ids": ["k-…"]
   }
   ```

1. Open the record of the repair round. It follows the first record, and its outcome is `landed`:

   <!-- described, not captured -->
   ```json
   {
     "id": "ev-20261120102204-m81tzc",
     "schema": "codelead.evidence/1",
     "command": "repair",
     "outcome": "landed",
     "follows": "ev-20261120101512-k3p9qa",
     "repair_round": 1,
     "repair_rounds": [{ "round": 1, "outcome": "patch_applied", "validation": "passed" }],
     "validation": {
       "status": "passed",
       "commands": [{ "command": "<MSBuild.exe> sampleapp.slnx -restore -nologo -v:minimal", "status": "passed", "exitCode": 0 }]
     },
     …
   }
   ```

1. Read the chain: the request, the files of the change, the patch check, the apply, the failed gate, the repair round and the passing gate.

> [!NOTE]
> **For team leads:** the records hold no source code: requests, file names, verdicts and exit codes. See [Evidence records](../concepts/evidence-records.md) and [Evidence record format](../reference/evidence-record-format.md).

## Open a pull request

[`/pr`](../chat-commands/pr.md) hands the landed change to a reviewer. It makes a branch, a commit and a push, then a pull request whose body is the evidence record. Session logs are never part of the commit.

1. Add your empty GitHub repository as the remote `origin`, and push the base branch:

   ```console
   git remote add origin https://github.com/<your account>/basic-store.git
   git push -u origin main
   ```

1. Start a session again:

   ```console
   codelead interactive
   ```

1. See what `/pr` would do, with no change:

   ```console
   /pr --dry-run
   ```

   You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Dry run — nothing was changed. This would run:
     git checkout -b codelead/<request>-<id>
     git add -A (then .codeleadsessions and .codelead/sessions unstaged, if anything there was staged)
     git commit -m "…"
     git push -u origin codelead/<request>-<id>
     gh pr create --base main --head codelead/<request>-<id> --title "…" --body-file <body>

   Title: …
   …
   ```

1. Open the pull request:

   ```console
   /pr
   ```

   The result says `Pull request opened: <address>`, then the branch and `Evidence record <id> now carries the pull request.`

1. Open the address in a browser. The body holds the request, what changed, the gate and its verdict, and the repair round.

## Clean up

1. On GitHub, close the pull request, or delete the repository that you made for this tutorial.

1. Delete the folder of the sample:

   ```console
   cd ..
   Remove-Item -Recurse -Force basic-store
   Remove-Item request.txt
   ```

## Next steps

> [!div class="nextstepaction"]
> [Evidence records](../concepts/evidence-records.md)

- [Build a new app, step by step](build-a-new-app.md)
- [Modernize a .NET Framework application](modernize-a-dotnet-framework-app.md)
- [The patch check](../concepts/the-patch-check.md)
- [Declare validation commands](../how-to/declare-validation-commands.md)
