---
title: The eight stages
description: What each of the eight stages of a modernization does, what you do in it, what it writes, and how you know that it worked.
kind: concept
area: modernization
date: 2026-11-20
---

# The eight stages

A modernization in CodeLead has eight stages: Read, Observe, Pin, Upgrade in place, Model, Set up the rebuild, Rebuild and Hand over. They are the same for every application. Inside a stage, CodeLead decides what runs and what is skipped, and says why.

**In short**
- You know what each stage is for, what CodeLead does in it, and what you do.
- You know which files each stage writes, and which words say that it worked.
- You know the command and the chat line that run each stage by hand.

## Why there are stages

An old application is risky to change. Nobody remembers everything that it does, and it often has no tests. The stages put the work in a safe order: first know the application, then record its behaviour, and only then change it. Each stage reads what the stages before it wrote, so no stage has to guess.

The stages are the steps of the [modernization program](the-program.md). The program keeps their state in `.codelead/modernization.json`.

## The order, and where you act

```text
 1. Read                 CodeLead reads the code. Nothing changes. No model call.
       |
 2. Observe              <-- YOU record the journeys with the Analyst, then select Continue.
       |
 3. Pin                  CodeLead writes tests.           <-- YOU approve each patch (approve mode).
       |
 4. Upgrade in place     CodeLead changes the old code.   <-- YOU approve each landing (approve mode).
       |
 5. Model                CodeLead reads the application into its System Model. No model call.
       |
 6. Set up the rebuild   <-- YOU give two answers: identity and database.
       |
 7. Rebuild              CodeLead builds the new application, unattended, within limits.
       |
 8. Hand over            CodeLead writes the audit pack. YOU hand it to a reviewer.
```

A stage that needs you shows **waiting** with what it waits for. Nothing after it runs until you act.

## The names

The eight names above are the names of the stages. The program's files and commands use a short id for each stage. The words that `codelead modernize status` prints today are older titles of the same stages.

| # | Stage | Stage id | Title in `status` today | Step of the wizard |
|---|---|---|---|---|
| 1 | Read | `read` | Read the application | Read |
| 2 | Observe | `recordings` | Record the journeys | Recordings |
| 3 | Pin | `pins` | Pin the contracts | Rebuild (the stage list) |
| 4 | Upgrade in place | `in-place` | Transform in place | Rebuild (the stage list) |
| 5 | Model | `model` | Extract the model | Rebuild (the stage list) |
| 6 | Set up the rebuild | `rebuild-init` | Initialize the rebuild | Your answers, then Rebuild |
| 7 | Rebuild | `rebuild` | Build and verify | Rebuild |
| 8 | Hand over | `audit` | Write the audit pack | Done |

## 1. Read

- **Why it exists**: you cannot change safely what you do not know. Read gives a picture of the whole application before anything changes.
- **What CodeLead does**: it surveys the repository and changes nothing. It reads the projects, their frameworks and packages, the history, the rules in the code, the tests and the security exposure. Then it lists the kinds of modernization that fit, and an order. When the program has the application's address, the [Explorer](../analyst/explore-an-application.md) visits its screens. The Explorer is a CodeLead tool that clicks through a running application and sends no form. With `codelead modernize start --as <role>=<account>`, it signs in as each account, and explores what each role can reach. A missing password makes the stage wait, and its problem names the variable to set. An Explorer that explored nothing makes the stage wait too, for example after a refused sign-in. No model is called.
- **What you do**: nothing, for the survey. Before it, you can [let CodeLead start the old application](source-run-profile.md). After it, you can narrate the code: the pins need the narration to count what they prove. When you do not, the Pin stage offers to narrate first. See [The dossier](dossier.md#the-narration).
- **What it produces**: the dossier, CodeLead's report on the application, in `docs/codelead/dossier/`: `DOSSIER.md`, `options.md`, `modernization-readiness.md`, `security-exposure.md` and their JSON files. With the Explorer: a folder `analyst-recordings/<time>/` with `sitemap.json`.
- **How you know it worked**: the stage ends `done`, with a line such as `Done: The dossier: 34 files; the options' sequence T5 → T1 → T2 → T8; …`.
- **The command and the chat line**: `codelead modernize run read`, or `/modernize --run read`. By hand, without the program: [`/modernize --plan`](../chat-commands/modernize.md#--plan).

## 2. Observe

- **Why it exists**: the code says what the application can do. A person knows what its users actually do. A journey is a path that a user takes through the application, step by step. The journeys that a person records become the checks of the rebuild.
- **What CodeLead does**: the Explorer already listed the screens in Read. In this stage, CodeLead waits for you and counts the recordings.
- **What you do**: you record each important journey with the [Analyst](../analyst/record-a-journey.md), a CodeLead tool that records what a person does in the application. You draft each recording into journeys and confirm them. Then you select **Continue**.
- **What it produces**: for each recording, a folder `analyst-recordings/<time>/` with `recording.json`, `journeys.json` and `journeys.md`.
- **How you know it worked**: a line such as `Record the journeys: done (0 recordings drafted).` The number is the count of drafted recordings that CodeLead found.
- **The command and the chat line**: `codelead modernize continue`, or `/modernize --continue`. The program never runs this stage by itself: the recordings are a person's work.

> [!NOTE]
> **New to this?** You can continue with no recording. The rebuild then has only the journeys that CodeLead drafts from the code. Your recordings are the strongest checks, because they show what real users do.

## 3. Pin

- **Why it exists**: a [pin](pins.md) is a test that records what the code does today. CodeLead runs the pins again after every later change. A change that alters the behaviour makes a pin fail, and CodeLead does not keep that change.
- **What CodeLead does**: when the web project has no narration, it narrates the project first, because the pins take their facts from it. Then it writes the pins in small increments, first at the in-memory level, then at the database level. Each increment is an [`/implement`](../chat-commands/implement.md) run: a patch, the [patch check](../concepts/the-patch-check.md), then [the gate](../concepts/validation-and-the-gate.md), which builds the code and runs the tests. When the gate refuses, CodeLead repairs the patch. After each landed increment, it offers the next one.
- **What you do**: in approve mode, you answer `Narrate webapp now? [Y/n]` when there is no narration, then `Apply this patch? [Y/n]` and `Pin the next: … [Y/n]`. The capital letter is CodeLead's recommendation, and Enter gives it. For a database on a server, you give the pins a database of their own.
- **What it produces**: the test projects `<project>.Characterization/` and `<project>.Characterization.Db/`, and their entries in `.codelead/validation.json`. Also the ledger `docs/codelead/dossier/pin-levels.json`, and one [evidence record](../concepts/evidence-records.md) for each increment.
- **How you know it worked**: the stage ends `done`, and `codelead modernize status` prints a coverage line under it. In CodeLead's own tests, on a small project, the line reads `Coverage: in memory 9 of 9 facts pinned (4 of 4 units with tests; rules 3 of 3); database 10 of 10 facts pinned …`. While provable facts remain, the stage ends `waiting`, with a title such as `6 facts still wait at the in-memory level`.
- **The command and the chat line**: `codelead modernize run pins`, or `/modernize --run pins`. By hand: [`/modernize --pin webapp`](../chat-commands/modernize.md#--pin), then `/modernize --pin webapp --level database`.

> [!IMPORTANT]
> The pins of a .NET Framework web project build with Visual Studio's MSBuild, on Windows. On another system, the stage fails before any model call, with `The pins of webapp need Visual Studio's MSBuild, and it is not on this machine`. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).

## 4. Upgrade in place

- **Why it exists**: some parts of an old application can stay. A small, checked change in the old code is often safer than a rewrite.
- **What CodeLead does**: it runs the [transforms](transforms.md) that the dossier's options report indicates, in this order. First T5D: the data layer moves into a new class library. Then T1 on the class libraries: they get the project format of current .NET, and target a current .NET as well. Then T10, for each unmaintained package that the report names. T10 never takes the framework that the application is built on, such as ASP.NET MVC. It never takes an ORM (a library that maps tables to classes) whose query types the code uses directly, such as Entity Framework 6. That ORM is the work of T5D or the rebuild. On the Basic Store sample, no package is left for T10, so the stage runs none. Then S, for the security findings that a patch can close and whose fix changes nothing for a legitimate caller. A finding that only another transform or the rebuild can close, such as an old runtime, is named and not patched. The gate runs the pins on each transform. A transform that landed in an earlier run is not run again.
- **What you do**: in approve mode, you approve each landing. When a transform fails, you read its problem and act on it. Some security findings change who may do what when they are fixed, such as a write action without `[Authorize]`. You decide each of them: [waive it with a reason, or fix it on purpose](../how-to/waive-a-security-finding.md). Then you answer `security=decided`.
- **What it produces**: the changed code and project files in the old repository, and a new library such as `webapp.Data/`. Each landing writes one evidence record.
- **How you know it worked**: the stage ends `done`, with a line of the form `<n> transforms landed: <each transform with its scope>`. A transform that does not land fails the stage, and its problem says why. While security decisions wait for you, the stage ends `waiting`, with a title such as `9 security findings are decisions for a person`.
- **The command and the chat line**: `codelead modernize run in-place`, or `/modernize --run in-place`. By hand: [`/modernize --transform`](../chat-commands/modernize.md#--transform).

## 5. Model

- **Why it exists**: to rebuild an application, CodeLead must know every entry point, screen and table that it has. The [System Model](system-model.md) is that list, read from the code, with the file and line of each item.
- **What CodeLead does**: it runs [`codelead model`](../cli/codelead-model.md), with no model call. The extraction gate then checks that the map has no hole that nobody explained. Sometimes every open finding is an ASP.NET Identity page that nothing reaches. Then CodeLead records the reasons itself, and reads the model again. Last, it writes the feature map: the list of features that the rebuild carries.
- **What you do**: nothing, unless the gate has other findings. Then you write a reason for each one, under its key, in `docs/codelead/rebuild/extraction-dispositions.json`.
- **What it produces**: in `docs/codelead/rebuild/`: `system-model.json`, `model.md`, `extraction-gate.md`, `journeys.draft.json`, `truth.draft.json`, `schema.json`, `schema.md`, `feature-map.json`, `feature-map.md`, and `extraction-dispositions.json` when reasons were recorded.
- **How you know it worked**: `Done: The System Model of webapp: the gate passed (36 Identity scaffold findings recorded as intended); the feature map written.`
- **The command and the chat line**: `codelead modernize run model`, or `/modernize --run model`.

## 6. Set up the rebuild

- **Why it exists**: the [rebuild](rebuild.md) is a new application in its own folder, beside the old one. CodeLead needs a starting point and two decisions that only you can make.
- **What CodeLead does**: it runs [`codelead rebuild init`](../cli/codelead-rebuild-init.md) with your answers. It copies a template, a starter application, and installs its packages. It chooses ports for the new API and web application that the old application does not use. It writes the manifest, the file that describes the rebuild, with its run profile: how to start the new application. It merges the journeys, initializes CodeLead in the new folder, writes the plan, and checks the plan with a dry run. Last, it commits what it made in the new folder.
- **What you do**: you give two answers. `identity`: keep the old accounts, or drop the sign-in. `database`: the name of a connection string, or `sample-seed` for sample rows. The rebuild folder and the template have defaults.
- **What it produces**: in the rebuild folder, `../<folder>-rebuild` by default: the template's files, `docs/codelead/rebuild/rebuild.json`, `journeys.json`, `rebuild-plan.json` and `rebuild-plan.md`, and `.codelead/validation.json`.
- **How you know it worked**: the plan says `blockers: none`, and the dry run passes. The stage then ends `done` with a line of the form `The rebuild was made at <folder>: 80 parts, no blocker; the dry run passes.`
- **The command and the chat line**: `codelead modernize answer identity=keep database=sample-seed`, then `codelead modernize next`. The stage alone: `codelead modernize run rebuild-init`, or `/modernize --run rebuild-init`.

## 7. Rebuild

- **Why it exists**: the new application is built in small parts. Each part is checked against the old behaviour before the next one starts.
- **What CodeLead does**: it runs [`codelead rebuild run`](../cli/codelead-rebuild-run.md) on every lane (an area of the plan), under the caps of the run profile. For each part, a model writes the code, the gate builds and tests it, and the walk tries it. A part that CodeLead cannot finish is parked: left for a person, with the reason. The run then continues. Then CodeLead starts the new application and runs [the walk](walk-and-parity.md), parity and the report.
- **What you do**: nothing while it runs. After it, you act on what the stage waits for, such as a parked part. Each parked part is a card on the rebuild's board.
- **What it produces**: the new application's code, in the rebuild folder. Also the run summary `.codelead/rebuild-run-<stamp>.md`, the reports `docs/codelead/parity/walk.md` and `docs/codelead/parity/rebuild-parity.md`, and the page `.codelead/rebuild-report.html`.
- **How you know it worked**: the stage's line says what the run did. It names the parts taken and the parts parked, and whether the walk and parity passed. Then CodeLead reads the plan again. The stage ends `done` only when every part of the plan is finished, for example accepted or dropped. Otherwise it ends `waiting`, with one of two problems:
  - `The rebuild waits for you`: a person's turn stands between. The reason names each turn with its line. For example, a parked part comes with its `/modernize --transform T9 --reopen <PART_ID> --reason "<why>"` line.
  - `The rebuild is not finished`: the run stopped at its caps, and nothing in the plan asks a person. Run the stage again.

  A walk that fails does not fail the stage. To go on with what is built, answer `codelead modernize answer rebuild=enough`. The program's evidence record of the answer keeps what the stage waited for. Or go past the stage with `codelead modernize skip rebuild --reason "<WHY>"`.
- **The command and the chat line**: `codelead modernize run rebuild`, or `/modernize --run rebuild`.

> [!WARNING]
> This stage calls models for every part, without asking. It stops at the caps, the limits of the run profile: 12 parts, 20 US dollars and 480 minutes by default. See [The rebuild](rebuild.md#the-unattended-run).

## 8. Hand over

- **Why it exists**: a reviewer who did not watch the work must be able to check it. The [audit pack](audit-pack.md) is the proof in one folder.
- **What CodeLead does**: it gathers every landed increment with its request, its evidence record, its commit and its attempts. It adds the gate, the survey before and after, the security findings, and every session with its secrets removed. A manifest lists the SHA-256 of every file.
- **What you do**: you read `README.md` in the pack, and you give the pack to the person who reviews the work.
- **What it produces**: `docs/codelead/audit-pack/`, with `README.md`, `manifest.json` and its layers.
- **How you know it worked**: a line of the form `Audit pack: docs/codelead/audit-pack/ (47 files, 0.2 MB)`, then the count of the increments, the sessions and the stops. Your numbers differ.
- **The command and the chat line**: `codelead modernize run audit`, or `/modernize --run audit`. By hand: [`/modernize --audit-pack`](../chat-commands/modernize.md#--audit-pack).

## Details

- Each stage that ends writes an evidence record, `modernize · <stage>`, with the outcome success, failure or asked.
- A stage has one of six statuses: `pending`, `running`, `waiting`, `done`, `failed` or `skipped`. A failed stage always carries a problem: its title, its reason, the way out and the command that runs it again.
- [`codelead modernize next`](../cli/codelead-modernize-next.md) runs a waiting stage again. It never runs the recordings or a failed stage, and it stops at a waiting Pin stage or a waiting Rebuild stage.
- [`codelead modernize skip`](../cli/codelead-modernize-skip.md) `<STAGE> --reason <WHY>` goes past a stage that failed or waits. The stage ends `skipped`, with your reason.
- The stages that call models are Pin, Upgrade in place and Rebuild. Read calls none, unless you narrate the code by hand.

## What it is not

- **The stages are not a fixed script**. CodeLead skips what does not apply and says why. For example, Upgrade in place is `skipped` when the options report indicates no transform.
- **The stages do not remove your control**. Each one runs by hand too, in the chat or a terminal, with the same result.

## Related content

- [The modernization program](the-program.md): the state file, the commands and the two modes
- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)
- [`codelead modernize`](../cli/codelead-modernize.md)
- [`/modernize`](../chat-commands/modernize.md)
- [The Modernize wizard](../extension/modernize-wizard.md)
- [Results and stops](../concepts/results-and-stops.md)
- [Glossary](../concepts/glossary.md)
