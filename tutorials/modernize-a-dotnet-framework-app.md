---
title: Modernize a .NET Framework application
description: Take the Basic Store sample, an ASP.NET MVC 5 application on .NET Framework 4.5, through the eight stages of a modernization, from the dossier to the audit pack.
kind: tutorial
area: tutorials
date: 2026-11-20
---

# Modernize a .NET Framework application

This tutorial takes the Basic Store sample through all eight stages of a modernization. The sample is an ASP.NET MVC 5 application on .NET Framework 4.5, with clients, products and orders. At the end, the old application has tests and an upgraded data library. A new API and web application stand beside it, and an audit pack proves every step.

**In short**
- You run the eight stages on one real application, with the wizard or with commands.
- You see what each stage asks of you, and what it says when it worked.
- You know which file to open after each stage.

In this tutorial, you:

> [!div class="checklist"]
> - Read the application, and narrate its code
> - Record the journeys that matter
> - Pin the application's behaviour with tests
> - Upgrade the data layer in place
> - Read the application into its System Model
> - Set up the rebuild with two answers
> - Rebuild the application, unattended
> - Hand over the audit pack

## Prerequisites

**Tools**

- **Windows**, with **Visual Studio 2022 or later** and the workload **ASP.NET and web development**. It brings Visual Studio's MSBuild, which builds the old project, and IIS Express, which runs it. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).
- **The .NET SDK** 8 or later, **Node.js** 20 or later, and **git**.
- **CodeLead**, installed. See [Install CodeLead](../get-started/install.md). For the wizard, also **CodeLead for VS Code** 0.7.16 or later. See [Install the extension](../extension/install.md).
- **A model**: a local model server, or a key for a remote provider. A stronger model on the repair role helps the pins land. See [Set up local models](../how-to/set-up-local-models.md), [Set up remote models](../how-to/set-up-remote-models.md) and [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md).
- **A licence of the Pro plan**, in the folder of the sample: `docs/codelead/licence.json`. The whole modernization program is in the Pro plan: the Explorer, the Analyst, the wizard, and the commands `codelead modernize` and `/modernize`. Without it, `codelead modernize start` is refused before any work. See [Licence and plans](../concepts/licence-and-plans.md).
- **The Basic Store sample** on your machine: the folder that holds `sampleapp.slnx` and `WebApplication1/`.

**Knowledge**

- You know how to open a terminal and run a command.
- You do not need to know CodeLead. You do not need to know ASP.NET either: the tutorial says what to look at.

**Time and cost**

- Plan a day. Most of it is waiting: the pins take one to two hours, and the rebuild takes hours.
- With a paid provider, the model calls cost money. One recorded run of the pins on this sample cost about 13 US dollars. The rebuild stops at 20 US dollars by default.

> [!NOTE]
> This tutorial needs Windows and a model, so its outputs from the Pin stage on were not captured for this page. Blocks marked as described come from CodeLead's code and tests, and from recorded runs on this sample. Your numbers differ.

## Prepare the sample

CodeLead's changes, and your way back, both need git. CodeLead also reads the built application to list its pages and actions, so you build it first.

1. Open a terminal in the sample's folder, the folder that holds `sampleapp.slnx`.
2. Check that the folder is a git repository with no change waiting:

   ```console
   git status
   ```

   You see output similar to the following:

   ```output
   On branch main
   nothing to commit, working tree clean
   ```

3. Open `sampleapp.slnx` in Visual Studio. Select **Build** > **Build Solution**. Visual Studio restores the NuGet packages and builds `webapp`. Then close Visual Studio.
4. Initialize CodeLead in the folder, in one of two ways:
   - **In Visual Studio Code**: open the folder, then select **Projects** > **Initialize CodeLead in this folder**.
   - **In a terminal**: run [`codelead init`](../cli/codelead-init.md).

   You see output similar to the following:

   ```output
   ✔ Created <your project>/.codelead/config.json
   <your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
   Repository map written: 1 module, 89 files; docs/codelead/repository-map.md
   Encyclopedia: 4 entries from the map (4 new, 0 updated, 0 downgraded); evidence record ev-20261120050520-ydk0pi.
   ```

5. Copy your licence file to `docs/codelead/licence.json`.
6. Set up your models, then check the machine:

   ```console
   codelead doctor
   ```

   Check two rows. The model server's row is `ok`, and `MSBuild (Visual Studio, for .NET Framework projects)` is `ok`.

7. Let CodeLead start the old application for the Explorer. Record the run profile with the IIS Express preset:

   ```console
   codelead modernize source --preset iis-express --url http://localhost:5080/
   ```

8. Commit what CodeLead wrote:

   ```console
   git add -A
   git commit -m "CodeLead initialized"
   ```

**What you have now**: a git baseline that you can return to, and a built application. CodeLead is initialized, and it knows how to start the application.

> [!TIP]
> If the folder is not a git repository yet, run `git init`, add a `.gitignore` for `bin/`, `obj/`, `packages/` and the `.mdf` and `.ldf` files under `App_Data/`, and commit. If `codelead doctor` shows a model server that does not answer, see [Set up local models](../how-to/set-up-local-models.md).

## Read the application

Stage 1 of 8, **Read**, surveys the code and changes nothing. Then the Explorer, a CodeLead tool, visits the running application's screens and sends no form. See [The dossier](../modernization/dossier.md).

1. Start the program.
   - **In the wizard**: in the **Modernization** view, select **Modernize this application** (the rocket in the view's title bar). In the step **The application**, type the address `http://localhost:5080/`, choose the mode **Approve each landing**, and select **Start**. In this mode, CodeLead asks you before it keeps each change.
   - **In a terminal**: run [`codelead modernize start`](../cli/codelead-modernize-start.md):

     ```console
     codelead modernize start --base http://localhost:5080/
     ```

   You see output similar to the following:

   <!-- described, not captured -->
   ```output
   The modernization program of webapp (aspnet-mvc), approve mode: started at .codelead/modernization.json; …
   ▸ Read the application (codelead modernize run read)
   Surveying <your project> into docs/codelead/dossier (read-only; D1–D15)...
   …
   Dossier bundle: docs/codelead/dossier/ (34 files)
   Survey: complete. Intake: defaults (all defaults).
   Options: 4 of 12 fit (T1, T2, T5, T8); recommended sequence: T5 → T1 → T2 → T8.
   …
     Done: The dossier: 34 files; the options' sequence T5 → T1 → T2 → T8; the Explorer crawled http://localhost:5080/ into analyst-recordings/<time> (the old application started from its run profile, and stopped after).
   ▸ Record the journeys (codelead modernize run recordings)
     Waiting for you: The recordings are a person's: 0 recordings drafted so far; continue when they are made.
   ```

2. Open `docs/codelead/dossier/DOSSIER.md`, then `options.md`. In `options.md`, find the T5 row: `webapp holds the EF data context StoreContext (4 entities) and no class library does`. That row decides the first change of stage 4.
3. Narrate the code. A model describes what each controller action does, in plain statements, and the pins later prove these statements. Open the chat: select the project in the **Projects** view, or run `codelead interactive` in a terminal. Then type:

   ```console
   /modernize --narrate
   ```

   CodeLead makes one model call for each unit. On this sample, a recorded run narrated 51 units in about 11 minutes on a local model. When you do not narrate now, the Pin stage asks `Narrate webapp now? [Y/n]` before it pins. See [Narrate the code before you pin it](../how-to/narrate-before-pins.md).

**What you have now**: the dossier in `docs/codelead/dossier/`, with the narration in `functional-narration.md`, and the Explorer's site map in `analyst-recordings/<time>/`.

> [!TIP]
> **For experienced users:** open `docs/codelead/dossier/modernization-readiness.md` for the project's name as the commands take it (`webapp`), and `functional-narration.md` for the facts that the pins will prove. The state of the program is in `.codelead/modernization.json`.

## Record the journeys

Stage 2 of 8 is **Observe**. A journey is a path that a user takes, with what each step must show. Your journeys become the checks of the rebuild. Record five, because they cover the store's rules.

1. Start the old application. In PowerShell, run IIS Express on the web project's folder:

   ```console
   & "C:\Program Files\IIS Express\iisexpress.exe" /path:<your project>\WebApplication1 /port:5080
   ```

2. In the **Analyst** view, select **Record a journey**. Give the address `http://localhost:5080/` and the name "Basic Store". The store needs no sign-in, so choose **New users**. See [Record a journey](../analyst/record-a-journey.md) for each control.
3. Record these five journeys:
   1. **Create an order**: Orders, New order, pick a client, add two lines, save. The details page shows the total.
   2. **The inactive product**: New order, try to add the inactive product "HDMI Cable 2 m". Record what the page shows.
   3. **The SKU rule**: Products, New product, with the SKU `ACC-1001`. The page shows "Another product already uses this SKU."
   4. **An empty order**: New order, no line, save. The page shows "Add at least one product to the order."
   5. **Cancel an order**: note a product's revenue, set an order that holds it to Cancelled, and see the revenue drop.
4. Draft each recording, and confirm its journeys. Journeys 2 to 4 show a refusal on purpose: confirm them.
5. Stop IIS Express: press **Q** in its terminal. The database file must be free for the next stage.
6. Say that the recordings are made.
   - **In the wizard**: in the step **Recordings**, select **Continue**.
   - **In a terminal**: run [`codelead modernize continue`](../cli/codelead-modernize-continue.md).

   You see output similar to the following, and the Pin stage starts:

   <!-- described, not captured -->
   ```output
   Record the journeys: done (5 recordings drafted).
   ▸ Pin the contracts (codelead modernize run pins)
   ```

**What you have now**: five recordings, each with its `journeys.json`, in `analyst-recordings/`.

> [!TIP]
> **For experienced users:** before you continue, replay a recording on the old application with [`codelead analyst replay`](../analyst/reference/analyst-replay.md). A replay that passes shows that the journey itself is right.

## Pin the behaviour

Stage 3 of 8, **Pin**, writes tests that record what the code does today. It works at two levels: in memory, then on a copy of the database. See [Pins](../modernization/pins.md).

The stage runs in the terminal **CodeLead: modernize** (in the wizard) or in your terminal. In approve mode, it asks you there. You narrated the code in stage 1, so the stage pins on the narration's facts and does not offer to narrate.

1. Read the card `CodeLead assumed`. It lists what CodeLead decided in place of asking you. There is nothing to answer.
2. At `Apply this patch? [Y/n]`, press **Enter**. The capital `Y` is CodeLead's recommendation. A line such as `The patch check passed with 1 note. CodeLead recommends applying.` means yes: a note does not block.
3. Wait for the gate. It builds the pins with MSBuild, then runs them with `dotnet test`. If it refuses, CodeLead asks `The gate refused the pins: … Repair it? [Y/n]`. Press **Enter**.
4. Read the result. It says what landed and what is covered. These lines come from CodeLead's tests, on a small project:

   <!-- described, not captured -->
   ```output
   Increment 1 landed: 4 tests on SessionController, 2 on InvoicesController. So far: 2 of 5 units with provable facts have tests; 6 of 19 provable facts pinned (in memory 6 of 9, database 0 of 10). Rules pinned: 2 of 7; guards 2 of 6; outputs 2 of 6. 3 facts wait for the in-memory level; 10 facts wait for the database level; 8 attribute facts are the walk's.
   Pin the next: ItemsController, CustomersController (3 facts)? [Y/n]
   ```

5. At `Pin the next: … [Y/n]`, press **Enter**. Repeat until the stage moves to the database level.
6. At the database level, CodeLead copies the sample's database file for the pins, and asks nothing. Check that the three lines name a copy in `webapp.Characterization.Db/data/`, and end with `The application's own database is not touched: the pins write and delete rows in the copy only.`
7. Answer the patches and the offers as before, until the stage ends.
8. See the coverage:

   ```console
   codelead modernize status
   ```

   Under `3. Pin the contracts`, a `Coverage:` line gives the facts pinned at each level, and the rules among them.

9. If the stage shows **waiting** with a title such as `6 facts still wait at the in-memory level`, run it again. In the wizard, select **Retry**. In a terminal, run `codelead modernize run pins`. When the coverage is enough for you, run `codelead modernize answer pins=enough` instead.
10. Commit the pins:

    ```console
    git add -A
    git commit -m "Pins"
    ```

**What you have now**: two test projects, `webapp.Characterization/` and `webapp.Characterization.Db/`, whose tests run in the gate of every later change.

> [!TIP]
> **For experienced users:** open `.codelead/validation.json` for the pins' gate entries, and `docs/codelead/dossier/pin-levels.json` for the level of each fact and why. Each increment's evidence record is in `.codelead/evidence/`. If the pins stop with a message, find it in the table of [Pins](../modernization/pins.md#you-see-it-means-do).

## Upgrade the data layer in place

Stage 4 of 8, **Upgrade in place**, changes the old code in small steps, under the pins. On this sample, it moves the data layer into a library, then upgrades that library. See [Transforms](../modernization/transforms.md).

1. Move the program on with [`codelead modernize next`](../cli/codelead-modernize-next.md). Type it in a terminal, or in the wizard's terminal **CodeLead: modernize**. When the Pin stage ended `done` in the same run, the program is already on this stage.
2. The stage first says which packages T10 leaves out. `Microsoft.AspNet.Mvc` is the framework that the application is built on. `EntityFramework` is the ORM (a library that maps tables to classes) that the code is written in. The code uses its query types directly, such as `DbSet<Order>`. They are the sample's only unmaintained packages, so the stage runs no T10. It also names the security finding that no patch can close: the old runtime. Then it starts with T5D, the data layer into a library:

   <!-- described, not captured -->
   ```output
   ▸ Transform in place (codelead modernize run in-place)
   T10: not on Microsoft.AspNet.Mvc (ASP.NET MVC, webapp): the platform the application is built on is not put behind an adapter (the platform itself: T1 or the rebuild).
   T10: not on EntityFramework (Entity Framework 6, webapp): not for an adapter, an ORM the code is written in (its query types are used directly): the data layer's own transform (T5D) or the rebuild.
   S-002 end-of-life-runtime: closed by the platform upgrade (T1) or the rebuild; not a security patch.
   Transform: /modernize --transform T5D --scope webapp
   Transform T5D on webapp, webapp.Data
   …
   Request: Extract the data layer of webapp into a new class library webapp.Data, with no behaviour change. …
   ```

3. At `Apply this patch? [Y/n]`, read the files that the patch check lists: the seven moved files, the new project `webapp.Data`, and the solution entry. Press **Enter**.
4. Wait for the gate. The pins must pass with the moved files. The result reads `Transform T5D landed with the parity gate green`.
5. The stage goes on with T1 on `webapp.Data`. The library gets the project format of current .NET, and targets current .NET beside .NET Framework 4.5. Approve it as in step 3.
6. Then S fixes the security findings that a patch can close, and whose fix changes nothing for a legitimate caller. On the sample that is one finding: the debug setting. Read the patch before you approve it. The old runtime (S-002) is not patched here: the rebuild closes it.
7. The stage then waits for you. Nine write actions of the store accept a write without `[Authorize]`, findings S-003 to S-011. A fix would refuse the callers that your journeys recorded, so the stage does not apply it. It lists each finding with its two ways on:

   <!-- described, not captured -->
   ```output
     Waiting for you: … transforms landed: …; 9 security findings are decisions for a person.
     9 security findings are decisions for a person: fixing them changes who may do what (the fix adds an authorization requirement: a caller who may write today without signing in is refused): the journeys recorded the application as it is, so the stage does not apply them. Decisions for a person: S-003 (Create accepts a write without [Authorize], WebApplication1/Controllers/ClientsController.cs:63): waive it with a reason: /modernize --waive S-003 --reason "<why it is meant to be so>"; or fix it on purpose: /modernize --transform S --security S-003 — S-004 (…) — …
     The way out: For each, waive it with its reason or fix it on purpose (the two lines above); then record the choice: codelead modernize answer security=decided. Or let the stage fix them all: security=fix-all; or leave them open: security=fix-none.
     Then: codelead modernize run in-place
   ```

8. The store accepts writes without sign-in by design, so waive each of the nine findings. In the chat, type:

   ```console
   /modernize --waive S-003 --reason the store accepts writes without sign-in by design
   ```

   Do the same for S-004 to S-011.
9. Record your choice, then run the stage again:

   ```console
   codelead modernize answer security=decided
   codelead modernize run in-place
   ```

   T5D and T1 landed before, and are not run again. S landed before too, and is not run again: the stage knows it by the findings that it covered. The stage ends `done`.

10. When the stage ends, commit:

    ```console
    git add -A
    git commit -m "Upgrade in place"
    ```

**What you have now**: the data context and its entities in a library of their own, `webapp.Data`. It builds for .NET Framework and for current .NET, and the web project still runs on it. Each of the nine open write actions has a waiver with its reason, in `docs/codelead/security-waivers.json`.

> [!TIP]
> To fix a finding on purpose instead, or to decide all of them with one answer, see [Waive a security finding](../how-to/waive-a-security-finding.md). When a stage fails, the program does not run it again by itself. Fix the cause and run the stage again, or go past it with [`codelead modernize skip`](../cli/codelead-modernize-skip.md) and your reason. See [Fix a stop](../how-to/fix-a-stop.md#unblock-a-modernization-stage).

> [!TIP]
> **For experienced users:** read each landing's evidence record in `.codelead/evidence/`, and the new `webapp.Data/webapp.Data.csproj`. Add a gate entry that builds the whole solution, so that every later change builds it. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md#declare-your-own-commands).

## Model the application

Stage 5 of 8, **Model**, reads the application into its System Model, with no model call. See [The System Model](../modernization/system-model.md).

1. The program runs this stage after Upgrade in place. If the program stopped before it, run [`codelead modernize run model`](../cli/codelead-modernize-run.md). You see output similar to the following:

   ```output
   ▸ Extract the model (codelead modernize run model)
   System Model of webapp (basic-store/WebApplication1, inventory: project-file): 289 nodes, 189 edges; 63 entry points, 154 caller records; 10 not resolved, 0 leading nowhere.
   Recipes: aspnet-mvc, jquery-browser. Passes: server, surfaces, client, channel ran.
   Gate: not passed: 36 findings to resolve (36 unexplained-orphan).
   …
   The extraction gate: 36 findings, every one the ASP.NET Identity scaffold that nothing live reaches (Account/ConfirmEmail, Account/ForgotPassword, …): 36 reasons recorded in docs/codelead/rebuild/extraction-dispositions.json. The model is read again.
   …
   Gate: passed.
   …
     Done: The System Model of webapp: the gate passed (36 Identity scaffold findings recorded as intended); the feature map written.
   ```

   The 36 findings are pages of the ASP.NET Identity template that nothing in the sample links to. CodeLead records a reason for each, and the gate passes.

2. Open `docs/codelead/rebuild/feature-map.md`. It lists the 63 features of the old application, in the order of the increments. Seven of them are Manage pages of the Identity template that nothing live reaches, such as `manage.add-phone-number-get`: their live triggers read `none found`. The rebuild's plan drops them in the next stage.

**What you have now**: the System Model and the feature map in `docs/codelead/rebuild/`.

> [!TIP]
> **For experienced users:** open `docs/codelead/rebuild/extraction-gate.md` for every finding with its key, and `extraction-dispositions.json` for the reasons that CodeLead recorded.

## Set up the rebuild

Stage 6 of 8 is **Set up the rebuild**. The rebuild is a new application in its own folder, beside the old one. CodeLead needs two answers. See [The rebuild](../modernization/rebuild.md).

1. Give the answers: keep the accounts, and start the new API on sample rows.
   - **In the wizard**: in the step **Your answers**, choose **Identity** `keep`, type **Database** `sample-seed`, and select **Save**. The wizard records the answers and runs the next stages.
   - **In a terminal**: run [`codelead modernize answer`](../cli/codelead-modernize-answer.md), then run the stage:

     ```console
     codelead modernize answer identity=keep database=sample-seed
     codelead modernize run rebuild-init
     ```

   You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Answers recorded: identity=keep, database=sample-seed.
   ▸ Initialize the rebuild (codelead modernize run rebuild-init)
   Rebuild of webapp (.) from the template dotnet-next-shadcn (…), in ../basic-store-rebuild.
   Template: 43 files copied.
   Ports: the API on 5081 (the template's 5080 moved), the web on 3000; the old application answers on 5080 (the program's address), 44366 (webapp.csproj's IISUrl), 52895 (webapp.csproj's DevelopmentServerPort), and both run side by side for the walk.
   Git: the folder is a git repository; what is made is committed at the end.
   …
   Journeys: docs/codelead/rebuild/journeys.json: … journeys (model 20, analyst 5, explorer …); … duplicates removed.
   …
   Plan: 80 parts: 68 planned, 12 dropped (10 parts dropped: nothing live reaches them); blockers: none. Written: docs/codelead/rebuild/rebuild-plan.json and docs/codelead/rebuild/rebuild-plan.md.
   Dry run: the checks passed; …
   Evidence: .codelead/evidence/ev-….json (rebuild-init · success).
   Commit: … "CodeLead: the rebuild's scaffold from dotnet-next-shadcn", with this machine's git user: the rebuild starts on a clean tree.
   Ready: run codelead rebuild run in ../basic-store-rebuild.
   ```

   The template's packages install first, which takes a few minutes. Read three lines:

   - `Ports:` the old application answers on 5080, so the new API moves to 5081. The two run side by side during the walk.
   - `Plan:` the plan drops 12 of its 80 parts. Ten of them belong to the seven Manage pages that nothing live reaches.
   - `Commit:` CodeLead committed the new folder with your git user, so you do not commit it yourself.

2. Open `../basic-store-rebuild/docs/codelead/rebuild/rebuild-plan.md`. It lists the parts in order, the lanes, and the next part: `identity-seam`.

**What you have now**: a new folder, `basic-store-rebuild`, beside the sample. It holds the template, the manifest, the merged journeys and the plan, in its first commit.

> [!TIP]
> **For experienced users:** open `docs/codelead/rebuild/rebuild.json` in the rebuild folder. Its `run` block is the run profile, with the caps of the unattended run under `run.unattended`. Lower `maxCostUsd` there before the Rebuild stage if you want a smaller cap.

## Rebuild the application

Stage 7 of 8, **Rebuild**, builds the new application part by part, unattended, then walks it and checks parity. It needs no answer from you. See [The walk and parity](../modernization/walk-and-parity.md).

1. Start the stage. In the wizard, the program goes on from the previous stage by itself. In a terminal, run `codelead modernize run rebuild`. The stage prints one line for each event, starting with:

   <!-- described, not captured -->
   ```output
   ▸ Build and verify (codelead modernize run rebuild)
   The rebuild run: ../basic-store-rebuild, every lane, under the run profile's caps ($20, 480 minutes) …
   ```

2. Follow the progress on the report. In the wizard, the step **Rebuild** shows it. In a browser, open `../basic-store-rebuild/.codelead/rebuild-report.html`.
3. When the stage ends, read its line. It names the parts taken, the parts parked for a person, and whether the walk and parity passed.
4. Open the rebuild's board, and read each parked part's card. Each card says why CodeLead stopped on that part.

**What you have now**: a new API and web application in `basic-store-rebuild`, the walk's report in `docs/codelead/parity/walk.md`, and the parity report in `docs/codelead/parity/rebuild-parity.md`.

> [!TIP]
> Before this stage, you can check every planned part with no model call. In the rebuild's chat, type `/modernize --transform T9 --next --dry-run`. On this sample it composes 68 parts and refuses none: `… 35 mismatches; 0 refused. Not composed: 12 dropped.` See [The rebuild](../modernization/rebuild.md#the-dry-run).

> [!TIP]
> **For experienced users:** the run summary is `.codelead/rebuild-run-<stamp>.md` in the rebuild folder, with each part's outcome and cost. The rebuild is finished only when `codelead parity --rebuild` says `Finished: yes`.

## Hand over the audit pack

Stage 8 of 8, **Hand over**, writes the audit pack: every landing with its proof, in one folder. See [The audit pack](../modernization/audit-pack.md).

1. In the wizard, the program runs this stage last, by itself. In a terminal, run `codelead modernize run audit`. You see output similar to the following:

   <!-- described, not captured -->
   ```output
   ▸ Write the audit pack (codelead modernize run audit)
   Audit pack: docs/codelead/audit-pack/ (… files, … MB)
   Increments: … landed; sessions: …; stops: …. Before `…` → after `…`.
   …
     Done: The audit pack: … landed increments, … sessions, … files.
   ```

2. Open `docs/codelead/audit-pack/README.md`. Check the table of increments: each pin and each transform, with its commit and its attempts.
3. See the whole program:

   ```console
   codelead modernize status
   ```

   Every stage reads `done`, or says why it is not.

**What you have now**: an audit pack that a reviewer can read without CodeLead, and the end of the program.

> [!TIP]
> **For experienced users:** `manifest.json` lists the SHA-256 of every file of the pack. Give `--since <COMMIT>` to `/modernize --audit-pack` with your baseline commit, so that "before" is the sample as you found it.

## Clean up

1. If IIS Express still runs, press **Q** in its terminal.
2. Delete the folder `basic-store-rebuild`, beside the sample.
3. To return the sample to your baseline, run these commands in its folder. They remove every change and every new file that came after the baseline:

   ```console
   git reset --hard <your baseline commit>
   git clean -fd
   ```

4. Delete the copied databases of the pins, if they are still there: the folder `webapp.Characterization.Db/data/`. Git ignores it, so `git clean -fd` keeps it.

> [!CAUTION]
> `git reset --hard` and `git clean -fd` cannot be undone. They remove the pins, the transforms, the dossier and the audit pack from the sample's folder. Copy the audit pack elsewhere first if you want to keep it.

## Next steps

- [Pins](../modernization/pins.md): what the pins proved, and how to read their coverage.
- [The eight stages](../modernization/the-eight-stages.md): each stage's inputs, outputs and messages.
- [The rebuild](../modernization/rebuild.md): the plan, the lanes and the parked parts.
- [The audit pack](../modernization/audit-pack.md): what a reviewer reads.
