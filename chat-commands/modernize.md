---
title: /modernize
description: Every modernization action of the chat, from the dossier to the audit pack, with its options, its questions, its results and its stops.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /modernize

**Applies to:** CodeLead 0.1 and later

## Name

`/modernize` - Runs the modernization actions: the dossier, the pins, the transforms, the rebuild, the program and the audit pack.

## Synopsis

```console
/modernize --plan [--out <FOLDER>] [--intake] [--eol-refresh] [--redacted]
/modernize --narrate [--project <NAME>] [--limit <N>]
/modernize --testimony <DOCUMENT>... [--assess]
/modernize --pin <PROJECT> [--level memory|database] [--facts <WHAT_TO_PIN>] [--units <TYPE>,...] [--again]
/modernize --transform T1 --scope <PROJECTS> [--target <FRAMEWORK>] [--without-pins] [--again]
/modernize --transform T5 [--scope <PROJECTS>] [--without-pins] [--again]
/modernize --transform T5D --scope <WEB_PROJECT> [--library <NAME>] [--again]
/modernize --transform T10 --dependency <PACKAGE> [--again]
/modernize --transform S --security all|<CLASS>|<IDS> [--again]
/modernize --transform T9 --feature <ID> [--part endpoint|screen]
/modernize --transform T9 --next [--scope <PREFIX>] [--dry-run]
/modernize --transform T9 --judge-again <PART_ID> --reason <WHY>
/modernize --transform T9 --reopen <PART_ID> --reason <WHY>
/modernize --transform T9 --intervene <PART_ID> --reason <WHY>
/modernize --waive <FINDING> --reason <WHY>
/modernize --audit-pack [--since <COMMIT>] [--out <FOLDER>] [--secret-file <PATH>]
/modernize --rebuild-map [--project <NAME>]
/modernize --rebuild-init --out <FOLDER> [--source <FOLDER>] [--template <NAME|PATH>] [--framework net10.0|net8.0] [--identity keep|drop] [--database <CONNECTION_NAME>|sample-seed] [--no-install] [--no-commit]
/modernize --rebuild-plan [--scope <PREFIX>]
/modernize --review channel
/modernize --run [<STAGE>] [--as <ROLE>=<ACCOUNT>...]
/modernize --continue
/modernize --answer <KEY>=<VALUE>...
/modernize --skip <STAGE> --reason <WHY>
/modernize --source --command <EXE> [--args <WORDS>...] --url http://localhost:<PORT>/ [--ready <PATH>] [--cwd <FOLDER>] [--start-ms <MS>] [--env NAME=value...] [--no-allow] [-- <ARGUMENTS>]
/modernize --source --preset iis-express --url http://localhost:<PORT>/ [--project <NAME>]
/modernize --source --show
/modernize --source --clear
```

## Description

Modernization moves an old application to a current platform, and records the application's behaviour before anything changes. `/modernize` runs every part of it from the chat. Each action is the first option after `/modernize`: `--plan`, `--pin`, `--transform` and the others. See [Modernization](../modernization/index.md) and [The eight stages](../modernization/the-eight-stages.md).

The actions follow an order. The program (`--run`) runs the stages in this order for you, and stops when a stage waits on you:

| Stage | What happens | The action by hand |
|---|---|---|
| `read` | Read the application: the survey and the dossier, and the Explorer's crawl. | [`--plan`](#--plan), [`--narrate`](#--narrate), [`--testimony`](#--testimony) |
| `recordings` | A person records the journeys with [the Analyst](../analyst/index.md). | [`--continue`](#--run---continue---answer---skip) when they are made |
| `pins` | Pin the contracts: tests that record what the code does today. | [`--pin`](#--pin) |
| `in-place` | Transform in place, under the pins. | [`--transform`](#--transform) |
| `model` | Extract the System Model of the old application. | [`codelead model`](../cli/codelead-model.md) |
| `rebuild-init` | Make the rebuild beside the old application. | [`--rebuild-init`](#--rebuild-init) |
| `rebuild` | Build the rebuild part by part, and verify each part. | [`--rebuild-plan`](#--rebuild-plan), [`--transform T9`](#--transform-t9) |
| `audit` | Write the audit pack. | [`--audit-pack`](#--audit-pack) |

Two rules hold for the actions that change code:

- **Pin first.** A transform runs under the gate of the pins. A transform with no pins in the gate is refused, unless you say `--without-pins`.
- **The composed actions are `/implement` runs.** `--pin` and `--transform` compose a request from the survey and run it through [`/implement`](implement.md): preflight, patch, patch check, apply, validation. They ask no preflight question: they assume the answers, and show what they assumed once, in a card titled `CodeLead assumed`. They repair themselves when the gate refuses them. The limit, `modernization.repair.rounds` (the default is 2), counts the rounds that did not move the failure. A round that ends with a new failure earns one more round, up to twice the limit. See [Pins](../modernization/pins.md#the-repair-rounds).

> [!NOTE]
> **New to this?** Start with `/modernize --plan` on the Basic Store sample. It reads the repository and changes nothing. Then read `docs/codelead/dossier/DOSSIER.md`. The [tutorial](../tutorials/modernize-a-dotnet-framework-app.md) walks the rest, and the [Modernize wizard](../extension/modernize-wizard.md) runs it with buttons.

Every action of `/modernize` needs the Pro plan in the folder. On the free plan, the chat refuses it before anything runs, with the outcome `failed` and a problem card: `The modernization program is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.` See [Licence and plans](../concepts/licence-and-plans.md).

> [!TIP]
> **For experienced users:** the command line runs the same program: [`codelead modernize`](../cli/codelead-modernize.md) with `start`, `next`, `run`, `continue`, `answer`, `skip`, `status`, `stop` and `source`. The program's state is `.codelead/modernization.json`. See [Modernization state file](../reference/modernization-state-file.md).

## Options

Each option belongs to one action. The action's section below says more.

| Option | Goes with | What it does |
|---|---|---|
| `--plan` | | Surveys the repository and writes the dossier. |
| `--out <FOLDER>` | `--plan`, `--audit-pack`, `--rebuild-init` | The output folder. |
| `--intake` | `--plan` | Asks the twelve intake questions again. |
| `--eol-refresh` | `--plan` | Refreshes the end-of-life dates from the network. |
| `--redacted` | `--plan` | Also writes a copy of the dossier without source lines. |
| `--narrate` | | Narrates the code: one plain statement per fact. |
| `--project <NAME>` | `--narrate`, `--rebuild-map`, `--source --preset` | The project to work on. |
| `--limit <N>` | `--narrate` | Narrates the first N units only. |
| `--testimony <DOCUMENT>` | | Matches people's documents against the code. |
| `--assess` | `--testimony` | A model judges each matched pair. |
| `--pin <PROJECT>` | | Pins a project's behaviour. |
| `--level memory\|database` | `--pin` | The level of the pins. The default is `memory`. |
| `--facts <WHAT_TO_PIN>` | `--pin` | Pins what these words say. |
| `--units <TYPE>,...` | `--pin` | Pins exactly these types. |
| `--again` | `--pin`, in-place `--transform` | Composes again although an increment is applied and not validated, or its patch was declined. |
| `--transform <TYPE>` | | Composes a transform: `T1`, `T5`, `T5D`, `T10`, `S` or `T9`. |
| `--scope <SCOPE>` | `--transform`, `--rebuild-plan` | The projects, or the feature ids and prefixes, joined by commas. |
| `--target <FRAMEWORK>` | `--transform T1` | The framework to add, for example `net8.0`. |
| `--dependency <PACKAGE>` | `--transform T10` | The package to put behind an adapter. |
| `--library <NAME>` | `--transform T5D` | The name of the new class library. |
| `--security <SELECTOR>` | `--transform S` | `all`, a class of findings, or finding ids. |
| `--without-pins` | in-place `--transform` | Runs without pins in the gate, on your responsibility. |
| `--feature <ID>` | `--transform T9` | Rebuilds one feature of the map. |
| `--next` | `--transform T9` | Composes the plan's next part. |
| `--dry-run` | `--transform T9 --next` | Composes every planned part with no model call, and checks it. |
| `--judge-again <PART_ID>` | `--transform T9` | Judges a landed part's acceptance again. |
| `--reopen <PART_ID>` | `--transform T9` | Plans a parked part again. |
| `--intervene <PART_ID>` | `--transform T9` | Runs the intervention rung on one part. |
| `--reason <WHY>` | `--waive`, `--skip`, `--judge-again`, `--reopen`, `--intervene` | Why. It is required with each of them. |
| `--waive <FINDING>` | | Records a "no" to a security finding. |
| `--audit-pack` | | Builds the audit pack. |
| `--since <COMMIT>` | `--audit-pack` | The commit before the work started. |
| `--secret-file <PATH>` | `--audit-pack` | A file whose contents are secrets to remove from the pack. |
| `--rebuild-map` | | Writes the feature map of a web project. |
| `--rebuild-init` | | Makes the rebuild repository. |
| `--source <FOLDER>` | `--rebuild-init` | The old application's folder. |
| `--template <NAME\|PATH>` | `--rebuild-init` | The template of the rebuild. |
| `--framework net10.0\|net8.0` | `--rebuild-init` | The rebuild's .NET version. |
| `--identity keep\|drop` | `--rebuild-init` | Keeps or drops the sign-in of the old application. |
| `--database <NAME>\|sample-seed` | `--rebuild-init` | The connection to use, or sample rows. |
| `--no-install` | `--rebuild-init` | Copies the template without installing its packages. |
| `--no-commit` | `--rebuild-init` | Does not commit what was made. You commit it yourself. |
| `--rebuild-plan` | | Writes the rebuild's plan. |
| `--review channel` | | Reviews the drafted rows of the channel translation table. |
| `--run [<STAGE>]` | | Runs the program's next stages, or one stage again. |
| `--as <ROLE>=<ACCOUNT>` | `--run` | A role that the read stage's Explorer signs in as, and its account. Repeat it for each role. |
| `--continue` | | Says that the recordings are made, then runs the next stages. |
| `--answer <KEY>=<VALUE>` | | Records the program's answers. |
| `--skip <STAGE>` | | Goes past a stage of the program that failed or waits, with `--reason`. |
| `--source` | | Records, shows or clears how CodeLead starts the old application. |
| `--command <EXE>` | `--source` | The program that starts the application. |
| `--args <WORDS>` | `--source` | Its arguments that do not start with `-`. |
| `--url <URL>` | `--source` | The address where the application answers. |
| `--ready <PATH>` | `--source` | The page that must answer before the application is ready. |
| `--cwd <FOLDER>` | `--source` | The folder to start it in. |
| `--start-ms <MS>` | `--source` | How long to wait for it to start. |
| `--env NAME=value` | `--source` | Environment variables for it. |
| `--no-allow` | `--source` | Records the profile, but CodeLead does not start the application. |
| `--preset iis-express` | `--source` | Starts it with IIS Express, on Windows. |
| `--show` | `--source` | Prints the run profile. |
| `--clear` | `--source` | Removes the run profile. |

## Actions

### `--plan`

```console
/modernize --plan [--out <FOLDER>] [--intake] [--eol-refresh] [--redacted]
```

Surveys the repository read-only and writes the dossier bundle to `docs/codelead/dossier/`. The survey reads the projects, their frameworks and packages, the readiness for each kind of modernization, the opportunities and the security exposure. The first time, CodeLead asks the intake: twelve questions about goals and constraints. Enter takes each default, and the dossier marks it as assumed. Then CodeLead derives the options and a recommended order. See [The dossier](../modernization/dossier.md).

- `--out <FOLDER>`: the bundle's folder.
- `--intake`: asks the intake again; the stored answers are the defaults.
- `--eol-refresh`: downloads the end-of-life dates of runtimes and packages (from endoflife.date).
- `--redacted`: also writes `redacted/`, a copy without source lines, for a writer who must not see the code.

> [!TIP]
> Build the projects before the survey when you can. The survey reads the built assemblies to list the actions and handlers, and [`--narrate`](#--narrate) and [`--rebuild-map`](#--rebuild-map) need that list.

Survey the Basic Store sample and take every default of the intake:

```console
/modernize --plan
```

<!-- described, not captured -->
```output
Modernization dossier
Surveying <your project> into docs/codelead/dossier (read-only; D1–D15)...
…
Dossier: <your project>/docs/codelead/dossier/DOSSIER.md — 66% tool-driven, 10 knowledge entries (20 observations)
Intake: twelve questions about goals and constraints; press Enter to take a default (the dossier marks it as assumed).

Clarification 1/12

What should the modernization achieve? (one or two sentences: the business reason, not the technology)

Why it matters:
Every option is judged against this; without it the dossier can only rank by technical risk.

Default:
Keep the system supportable and changeable: supported runtimes, a test safety net, and a build any developer can run.

Answer, press Enter to use default:
…
Dossier bundle: docs/codelead/dossier/ (34 files)
Survey: complete. Intake: asked (all defaults).
Options: 4 of 12 fit (T1, T2, T5, T8); recommended sequence: T5 → T1 → T2 → T8.

Next: to write the dossier's documents, paste CodeLead's writer's brief (docs/dossier-writer-brief.md, installed with CodeLead) with the bundle into a Claude session (--redacted writes a copy without source lines, for code that may not leave the machine).
```

The result is `done`: `Dossier bundle: docs/codelead/dossier/ (34 files)`, with the survey and the options as its facts. A second run says `Intake: stored (all defaults).` and asks nothing.

### `--narrate`

```console
/modernize --narrate [--project <NAME>] [--limit <N>]
```

Narrates the code. The survey found the actions, handlers and data contexts. For each of them, a model writes one plain statement per fact, anchored to its lines. It writes `functional-narration.json` and `functional-narration.md` into the bundle. [`--pin`](#--pin) then turns the rules that it read into tests. There is one model call per unit.

- `--project <NAME>`: narrates one project.
- `--limit <N>`: narrates the first N units. A number below 1 is refused: `--limit takes a positive number of units.`

When the survey listed no unit, there is nothing to narrate:

```console
/modernize --narrate --limit 2
```

```output
Narration: 0 units from the structural inventory, first 2; one model call each.
Nothing to narrate: the structural inventory lists no action, handler or EF context with a declaring source file.
```

### `--testimony`

```console
/modernize --testimony <DOCUMENT>... [--assess]
```

Reads documents from the people who know the application: `.md` or `.txt` files, or folders of them. CodeLead records every sentence as reported testimony and matches it to the narration. It writes `exception-queue.json` and `exception-queue.md`: the contradictions, and what the code does not show. It needs the narration first.

- `--assess`: a model judges each matched pair.

Match a store manager's notes, before the narration exists:

```console
/modernize --testimony manager-notes.md
```

```output
No narration at docs/codelead/dossier/functional-narration.json: run /modernize --narrate first; the documents are matched against what the code was read to do.
■ No testimony read: there is no narration to match the documents against; nothing was written; next: /modernize --narrate
```

The outcome is `stopped`: the narration must come first. When the narration exists and the documents give no statement, the outcome is `nothing`, with `No testimony read: the documents named give no statement; nothing was written`. CodeLead reads only `.md` and `.txt` documents.

### `--pin`

```console
/modernize --pin <PROJECT>
/modernize --pin <PROJECT> --level database
/modernize --pin <PROJECT> --facts <WHAT_TO_PIN>
/modernize --pin <PROJECT> --units <TYPE>,...
/modernize --pin <PROJECT> --again
```

Pins a project's behaviour. A pin is a test that records what the code does today, so that a later change that alters it fails. CodeLead composes the request from the survey and the narration, and runs it as an [`/implement`](implement.md). When an increment lands, its tests join the gate in `.codelead/validation.json`, and every later change runs them. See [Pins](../modernization/pins.md).

> [!IMPORTANT]
> `/modernize --pin` is not [`/pin`](pin.md). `/pin` keeps a list of what waits on a person. `/modernize --pin` writes tests.

**Increments.** Each increment pins a few units (types), chosen so that the model's reply fits. After a landed increment, CodeLead offers the next one by itself: `Pin the next: ItemsController, CustomersController (3 facts)? [Y/n]`. It stops when the level is covered, when you say no, at a stop, or after `modernization.pins.maxIncrementsPerRun` increments (the default is 6).

**Levels.** CodeLead decides for each fact which level can prove it, with no model call, and writes this in the ledger `pin-levels.json` beside the narration:

- `memory`: the in-memory level, with fakes and no database. This is the default.
- `database`: the facts that reach a data context or a repository. These pins write rows and delete them, on a database of their own.
- The walk: facts such as `[Authorize]` that only the running application can prove. The facts whose code needs the web runtime, such as a call of `FormsAuthentication.SignOut()`, are the walk's too. No pin level spends its budget on them.

**The pins' own database.** Before any model call, `--level database` settles the database. CodeLead copies a file-attached database into the pins' folder, under a catalog name of its own, and asks nothing. For a server database, CodeLead writes a git-ignored connections file with a placeholder, and asks one question. The question shows the application's own connections: never give those. The recommended answer is `done`: you put the copy's connection string in that file. The other answers are `env:<NAME>`, a connection string without a password, the path of a copy, or `stop`. A password typed into the chat is refused. A guard refuses any pin that reads the application's configuration or connects to the application's database.

- `--level memory|database`: the level. Anything else is refused: `--level must be memory or database.`
- `--facts <WHAT_TO_PIN>`: pins what these words say. The words go up to the next option.
- `--units <TYPE>,...`: pins exactly these types, names from the survey's structural inventory, joined by commas.
- `--again`: composes the increment again, even though one is applied and not validated, or its patch was declined.

A pin of a .NET Framework web project needs the MSBuild of Visual Studio. On a machine without it, CodeLead stops before any model call:

```console
/modernize --pin webapp
```

```output
The pins of webapp need Visual Studio's MSBuild, and it is not on this machine.

Reason:
webapp is a .NET Framework web project that the .NET SDK cannot build (webapp: non-SDK, v4.5, imports Microsoft.WebApplication.targets). MSBuild was not found (…). No built assembly of webapp to reference in its place (looked for WebApplication1/bin/WebApplication1.dll). Nothing was asked of the model: the gate's dotnet build of the pins could only fail with MSB4019.

Next:
- Run it on Windows with Visual Studio or its Build Tools (the web development build tools workload): …
- Or copy the output of a Windows build of webapp to WebApplication1/bin/: the pins then reference WebApplication1.dll in place of the project
- Or declare this scope's commands in .codelead/validation.json: …
```

A landed increment says what it added and the coverage so far, and never claims more. These words come from CodeLead's own tests, on a small project named `Ledger.Web`:

<!-- described, not captured -->
```output
Increment 1 landed: 4 tests on SessionController, 2 on InvoicesController. So far: 2 of 5 units with provable facts have tests; 6 of 19 provable facts pinned (in memory 6 of 9, database 0 of 10). Rules pinned: 2 of 7; guards 2 of 6; outputs 2 of 6. 3 facts wait for the in-memory level; 10 facts wait for the database level; 8 attribute facts are the walk's.
Pin the next: ItemsController, CustomersController (3 facts)? [Y/n]
…
✔ The pins of Ledger.Web are complete at the in-memory level (9 tests this run); next: /modernize --pin Ledger.Web --level database — the 10 facts that need a database; it runs on a database of its own, never the application's
```

The results of `--pin`:

| Outcome | Headline | The way on |
|---|---|---|
| `passed` | `The pins of webapp: this increment landed (3 tests)` | The next increment, as a command. |
| `passed` | `The pins of webapp are complete at the in-memory level (5 tests this run)` | `/modernize --pin webapp --level database — …` |
| `nothing` | `The pins of webapp are complete at the in-memory level` | `/modernize --pin webapp --level database` |
| `failed` | `The pins of webapp did not land: MSBuild exited 1` | `Repair it in this session: /repair …`, or run it again |
| `failed` | `The pins of webapp did not land: 11 of 11 tests failed: StoreContextTests.OrderLines_…: System.InvalidOperationException: No Entity Framework provider found …` | `Repair it in this session: /repair …`, or run it again |
| `failed` | `The pins of webapp need Visual Studio's MSBuild, and it is not on this machine` | The three ways above. |
| `failed` | `The database pins need a database that is not the application's` | Put the copy's connection string in the file, answer `done`, then run it again. |

The facts of a landed increment are the coverage: `This increment: 3 tests on OrdersController; 2 of the 3 facts it named pinned`, `So far: …`, `Rules pinned: …`, and what waits. A database pin names the database it ran against first, never its credentials. The other stops are in [The stops of a pin or a transform](#the-stops-of-a-pin-or-a-transform).

When a test cannot run in the test process and its repair left no patch, CodeLead offers to set it aside: `Set the 1 failing pin aside and land the 7 that pass? [Y/n]`. A yes removes the failing test method, and the rest land: `7 pins landed; 1 set aside: <test>: <reason>; the walk or the database level may prove it`. See [Pins](../modernization/pins.md#a-pin-that-cannot-run-in-the-test-process).

### `--transform`

```console
/modernize --transform T1 --scope <PROJECTS> [--target <FRAMEWORK>]
/modernize --transform T5 [--scope <PROJECTS>]
/modernize --transform T5D --scope <WEB_PROJECT> [--library <NAME>]
/modernize --transform T10 --dependency <PACKAGE>
/modernize --transform S --security all|<CLASS>|<IDS>
```

Composes an in-place transform from the survey and runs it as an `/implement`, under the gate of the pins. See [Transforms](../modernization/transforms.md).

| Type | What it does |
|---|---|
| `T1` | Makes the projects SDK-style and multi-target, for example .NET Framework and `net8.0`. |
| `T5` | Puts a seam between the code and its data access. |
| `T5D` | Moves the data layer (the data context, its entities, their enums and the initializer) into a new class library, `<project>.Data`, with the same framework and namespaces. The controllers compile unchanged. `T1` then upgrades the library. |
| `T10` | Puts one dependency behind an adapter. Never a package of the framework that the project is built on, such as `Microsoft.AspNet.Mvc`. Never an ORM (a library that maps tables to classes) whose query types the code uses directly, such as `EntityFramework`. CodeLead refuses such a package with the reason and the way on: T5D or the rebuild. No option forces it. See [T10](../modernization/transforms.md#t10-a-dependency-behind-an-adapter). |
| `S` | Fixes security findings of the dossier, with the pins as the gate. The re-run of the security scan is the proof. |

- `--scope <PROJECTS>`: the projects, joined by commas.
- `--target <FRAMEWORK>`: the framework that `T1` adds.
- `--dependency <PACKAGE>`: the package of `T10`.
- `--library <NAME>`: the library of `T5D`. It goes with `T5D` only.
- `--security <SELECTOR>`: the findings of `S`: `all`, a class, or ids such as `S-004`. `all` and a class take only the open findings, never a waived one. An id names one finding on purpose, waived or not.
- `--without-pins`: runs with no pins in the gate, on your responsibility.
- `--again`: composes the transform again, as for `--pin`.

Without pins in the gate, the transform stops before anything is composed:

```console
/modernize --transform T1 --scope webapp
```

```output
Pin first: the declared gate (.codelead/validation.json) carries no characterization suite, so no transform can be shown safe. Run /modernize --pin <project> for the projects in scope, or pass --without-pins to proceed on your own responsibility.
```

| Outcome | Headline |
|---|---|
| `passed` | `Transform T5D landed with the parity gate green` |
| `failed` | `Transform T1 did not land: MSBuild exited 1` |
| `stopped` | `Pin first: …` |

### `--transform T9`

```console
/modernize --transform T9 --feature <ID> [--part endpoint|screen]
/modernize --transform T9 --next [--scope <PREFIX>]
/modernize --transform T9 --next --dry-run
/modernize --transform T9 --judge-again <PART_ID> --reason <WHY>
/modernize --transform T9 --reopen <PART_ID> --reason <WHY>
/modernize --transform T9 --intervene <PART_ID> --reason <WHY>
```

`T9` builds the rebuild, part by part, in the rebuild repository that [`--rebuild-init`](#--rebuild-init) made. A part is an endpoint of the new API or a screen of the new front end, for example `orders.close:endpoint`. Each part runs as an increment: preflight, patch, validation, a checkpoint commit, then the acceptance walk on its own steps. See [The rebuild](../modernization/rebuild.md) and [The walk and parity](../modernization/walk-and-parity.md).

- `--feature <ID>`: rebuilds one feature of the map: its endpoint, then its screen. `--part endpoint|screen` picks one.
- `--next`: composes the part that the plan says comes next. `--scope <PREFIX>` limits it to some features.
- `--dry-run`: with `--next`, composes every planned part with no model call, and checks each quote and name against the code.
- `--judge-again <PART_ID>`: judges a landed part's acceptance again on the same tree, with no model call, when the walker was wrong or changed.
- `--reopen <PART_ID>`: plans a parked part again, with no model call.
- `--intervene <PART_ID>`: runs the model ladder's intervention rung on a part that was not accepted or is parked. An agent works with CodeLead's own tools, within limits of turns, minutes and dollars. The gate then judges the tree; when it does not accept, CodeLead puts the tree back.

`--reason <WHY>` is required with `--judge-again`, `--reopen` and `--intervene`: the evidence record says why. `--again` does not go with `T9`.

```console
/modernize --transform T9 --judge-again orders.close:screen
```

```output
--judge-again orders.close:screen needs --reason "<why>": the evidence record says why the landing is judged again. Nothing was walked or written.
```

A `T9` command ends with its own words as its result.

### `--waive`

```console
/modernize --waive <FINDING> --reason <WHY>
```

Records a "no" to a security finding of the dossier: you decide not to fix it, and you say why. The finding is an id such as `S-001`, or a key (`class|file|title`) from `security-exposure.json`. CodeLead writes it to `docs/codelead/security-waivers.json`, and the audit pack lists it. A waiver without a reason is refused.

```console
/modernize --waive S-001 --reason the page is internal only
```

```output
Waived S-001 Debug compilation enabled — the page is internal only. Recorded in docs/codelead/security-waivers.json (1 waiver(s)); the audit pack lists it under 5-security/.
```

### `--audit-pack`

```console
/modernize --audit-pack [--since <COMMIT>] [--out <FOLDER>] [--secret-file <PATH>]
```

Builds the audit pack in `docs/codelead/audit-pack/`. It holds every landed increment with its request, its evidence record, its commit and its attempts. It also holds the gate, the survey before and after, and every session, sanitized. A manifest lists the SHA-256 of every file. Read its `README.md` first. See [The audit pack](../modernization/audit-pack.md).

- `--since <COMMIT>`: the commit before the work started. The default is the parent of the first checkpoint commit.
- `--out <FOLDER>`: the pack's folder.
- `--secret-file <PATH>`: a file whose contents are secrets, removed from every copied text. You can give it more than once.

```console
/modernize --audit-pack
```

```output
Audit pack
Audit pack: docs/codelead/audit-pack/ (83 files, 0.7 MB)
Increments: 0 landed; sessions: 2; stops: 0. Before `e41a806` → after `bd30c0a`.
Note: No landed evidence record; layer 2 is empty.

Read README.md first; manifest.json lists every file with its SHA-256.
```

### `--rebuild-map`

```console
/modernize --rebuild-map [--project <NAME>]
```

Writes the feature map of a web project to `docs/codelead/rebuild/`. The map lists every controller action in the order of the increments. Each action has its old route, its view, its callers, its facts, its tests, and the endpoint or screen that replaces it. It needs the survey's structural inventory, or a System Model from [`codelead model`](../cli/codelead-model.md).

```output
No structural inventory at docs/codelead/dossier/functional-structure.json that read an action (the survey reads built assemblies): run /modernize --plan first (the projects must be built). Nor is there a System Model at docs/codelead/rebuild/system-model.json to make the map of: `codelead model --source <repo>` reads the application into one.
■ No rebuild map: there is no structural inventory and no System Model to make it of; nothing was written; next: /modernize --plan
```

The outcome is `stopped`, and the way on is `/modernize --plan` or `codelead model --source <repo>`. When the System Model is of another project than the one that `--project` names, the outcome is `failed`.

### `--rebuild-init`

```console
/modernize --rebuild-init --out <FOLDER> [--source <FOLDER>] [--template <NAME|PATH>] [--framework net10.0|net8.0] [--identity keep|drop] [--database <CONNECTION_NAME>|sample-seed] [--no-install] [--no-commit]
```

Makes the rebuild repository beside the old application, after `codelead model`. CodeLead copies and installs the template, and chooses ports that the old application does not use. It writes the rebuild manifest and its run profile, and initializes CodeLead there. It merges the journeys from the model's draft, the Analyst's recordings and the Explorer's stories. Then it writes the plan, runs the dry run, and commits what it made. See [`codelead rebuild init`](../cli/codelead-rebuild-init.md) and [Rebuild manifest](../reference/rebuild-manifest.md).

- `--out <FOLDER>`: the rebuild's folder. It is required.
- `--source <FOLDER>`: the old application. The default is this repository.
- `--template <NAME|PATH>`: the template.
- `--framework net10.0|net8.0`: the .NET version of the new API.
- `--identity keep|drop`: keeps the old application's sign-in, or drops it.
- `--database <CONNECTION_NAME>|sample-seed`: the connection of the old application to use, or sample rows.
- `--no-install`: copies the template without installing its packages.
- `--no-commit`: does not commit what was made. By default, CodeLead commits it with this machine's git user.

```console
/modernize --rebuild-init --no-commit
```

```output
--rebuild-init needs --out <folder>: the folder the rebuild is made in, beside the old application.
```

### `--rebuild-plan`

```console
/modernize --rebuild-plan [--scope <PREFIX>]
```

Runs in the rebuild repository. It writes the rebuild's plan from the old application's System Model to `docs/codelead/rebuild/rebuild-plan.json` and `.md`. The plan lists the parts that are not done yet, in order. Each part has its needs, its steps, its facts and its target files. Blockers come first. It calls no model, except once per project to propose how to split a C# API into partial classes.

- `--scope <PREFIX>`: feature ids or prefixes, such as `orders` or `orders.close`, joined by commas.

In the old repository, there is no manifest, so nothing is planned:

```output
No rebuild manifest at docs/codelead/rebuild/rebuild.json: the plan's commands run in the rebuild repository, whose manifest names the old repository, its feature map and the targets. Nothing is planned.
```

### `--review`

```console
/modernize --review channel
```

Runs in the rebuild repository. It shows each drafted row of the channel translation table beside its old lines, one by one. You accept it, rewrite it or skip it. CodeLead writes the reviewed rows into `decisions.channel` of the manifest, and never drafts them again. `channel` is the only value: `--review` alone is refused.

### `--run`, `--continue`, `--answer`, `--skip`

```console
/modernize --run
/modernize --run <STAGE>
/modernize --run [<STAGE>] --as <ROLE>=<ACCOUNT>...
/modernize --continue
/modernize --answer <KEY>=<VALUE>...
/modernize --skip <STAGE> --reason <WHY>
```

The modernization program in the chat. It is the same program as [`codelead modernize`](../cli/codelead-modernize.md), and its state is `.codelead/modernization.json`, the file that the wizard reads. See [The modernization program](../modernization/the-program.md).

- `--run`: runs the next stages in order, until one waits on you, one fails, or all are done. The same as [`codelead modernize next`](../cli/codelead-modernize-next.md).
- `--run <STAGE>`: runs one stage again: `read`, `recordings`, `pins`, `in-place`, `model`, `rebuild-init`, `rebuild` or `audit`.
- `--as <ROLE>=<ACCOUNT>`, after `--run`: a role that the read stage's Explorer signs in as, and its account, as [`codelead modernize start --as`](../cli/codelead-modernize-start.md#sign-in-accounts-for-the-crawl) does. Repeat it for each role. The state file keeps the roles and the accounts, never a password, for every run of the stage. Each password is read when the stage runs, from `CODELEAD_WALK_PASSWORD_<ROLE>`, else from the file that `CODELEAD_WALK_CREDENTIALS` names. A missing password makes the stage wait, and its problem names the variable. A sign-in that the application refuses for every role, or no screen at all, makes the stage wait too.
- `--continue`: the recordings are made; CodeLead marks them done and runs the next stages. It takes nothing after it.
- `--answer <KEY>=<VALUE>`: records answers, and runs nothing. The same as [`codelead modernize answer`](../cli/codelead-modernize-answer.md), with the same eight keys: `rebuildFolder`, `template`, `framework`, `identity`, `database`, `pinsDatabase`, `security` and `pins`. `pinsDatabase=done` (or `env:<NAME>`) answers the pins' database question. `security=decided`, `fix-all` or `fix-none` answers the security findings that wait for a person. `pins=enough` ends the pins stage with the coverage as it stands, only while the stage waits or failed.
- `--skip <STAGE> --reason <WHY>`: goes past a stage that failed or waits for you, as [`codelead modernize skip`](../cli/codelead-modernize-skip.md) does. The reason is required. The recordings are never skipped.

In unattended mode, `--run` and `--continue` ask you for the consent for each remote provider at their start, before the first model call. A no ends the command before any stage runs, with the outcome `stopped`. See [The modernization program](../modernization/the-program.md#approve-mode-and-unattended-mode).

The pins stage runs both levels and waits while provable facts remain. When the web project has no narration, the stage first asks `Narrate webapp now? [Y/n]`, in this session. A stage that waits on you shows its question as a problem, with the answer to give and the command that runs it again.

```console
/modernize --answer framework=net8.0
```

```output
Answers recorded: framework=net8.0.
```

`pins=enough` before the pins stage ran is refused:

```console
/modernize --answer pins=enough
```

```output
Refused: pins=enough: Pin the contracts (pins) has not run (it is pending): pins=enough is taken only while the stage waits for you or failed; run it first (codelead modernize run pins), then answer again if you choose. Nothing was recorded.
```

The chat checks the keys before it asks the program:

```console
/modernize --answer colour=blue
```

```output
/modernize --answer takes <key>=<value> pairs, the keys rebuildFolder, template, framework, identity, database, pinsDatabase, security, pins; not one: colour=blue.
```

A skip without its reason, and a skip of a stage that is skipped already:

```console
/modernize --skip in-place
/modernize --skip in-place --reason again
```

```output
--skip needs --reason <why>: the stage's record and the audit pack keep why the program went on without it. /modernize --skip <stage> --reason <why> (the stages: read, recordings, pins, in-place, model, rebuild-init, rebuild, audit).
Refused: Transform in place is skipped already (by the person: the transforms run on the Windows machine later): codelead modernize next goes on. Nothing was recorded.
```

```console
/modernize --run nonsense
```

```output
nonsense is not a stage of the program: read, recordings, pins, in-place, model, rebuild-init, rebuild, audit.
```

### `--source`

```console
/modernize --source --command <EXE> [--args <WORDS>...] --url http://localhost:<PORT>/ [--ready <PATH>] [--cwd <FOLDER>] [--start-ms <MS>] [--env NAME=value...] [--no-allow] [-- <ARGUMENTS>]
/modernize --source --preset iis-express --url http://localhost:<PORT>/ [--project <NAME>]
/modernize --source --show
/modernize --source --clear
```

Records how CodeLead starts the old application: its run profile, `modernization.source` in `.codelead/config.json`. The read stage starts the application for the Explorer's crawl, and stops it after. Recording the profile is your yes to CodeLead starting it; `--no-allow` records it without that yes. The folder must be initialized first. See [Start the old application for CodeLead](../modernization/source-run-profile.md).

- `--command <EXE>` and `--args <WORDS>`: the program and its arguments. An argument that starts with `-` goes after `--`.
- `--url <URL>`: where the application answers.
- `--ready <PATH>`: the page that must answer before the application counts as ready. The default is `/`. The application is ready when the page answers with a page, a redirect or a sign-in challenge: HTTP 200 to 399, 401 or 403. A server error (5xx) is not ready.
- `--cwd <FOLDER>`, `--start-ms <MS>`, `--env NAME=value`: the folder, the time to wait (the default is 120 s), and environment variables.
- `--preset iis-express`: IIS Express on Windows, for a .NET Framework web project; `--project <NAME>` names the web project.
- `--show`: prints the profile. `--clear`: removes it.

Record a profile for an application that you start yourself:

```console
/modernize --source --command dotnet --url http://localhost:5000/ --no-allow -- run --urls http://localhost:5000
```

<!-- described, not captured -->
```output
The old application's run profile is recorded in .codelead/config.json (modernization.source):
  command: dotnet run --urls http://localhost:5000
  answers at: http://localhost:5000/ (ready when / answers with a page, a redirect or a sign-in challenge (HTTP 200 to 399, 401 or 403), within 120 s)
  allowed: no, CodeLead does not start it: start it by hand before the read stage, or record the profile again without --no-allow
Then: codelead modernize start (or codelead modernize run read): the read stage waits for you to start the application.
```

The preset needs Windows:

```output
Refused: IIS Express runs on Windows, and this machine is macOS: run the preset on the Windows machine the application runs on, or record another way to start it here (…).
```

## What CodeLead asks

| Question | Action | CodeLead recommends | Enter gives |
|---|---|---|---|
| The twelve intake questions, each `Answer, press Enter to use default:` | `--plan` | The default. | The default. |
| `Apply this patch? [Y/n]` | `--pin`, `--transform` | Yes, when the check passed. | Yes. |
| `The gate refused the pins: <failure>. Repair it? [Y/n]` | `--pin`, `--transform` | Yes. | Yes. |
| `Pin the next: <units> (<n> facts)? [Y/n]` | `--pin` | Yes. | Yes. |
| `Narrate <project> now? [Y/n]` | `--run`, at the pins stage, when the web project has no narration | Yes: "the pins take their facts from it". | Yes. |
| `Set the 1 failing pin aside and land the 7 that pass? [Y/n]` | `--pin`, after a repair round that left no patch | Yes: "the pins that pass are pinned; the text of each one set aside is kept on the evidence record". | Yes. |
| `An increment is applied and not validated: validate it first? [Y/n]` | `--pin`, `--transform` | Yes. | Yes. |
| `An increment is applied and its gate refused it: run its gate again first? [Y/n]` | `--pin`, over an increment whose gate refused it and whose repair rounds ended | Yes: "composing again over it would write the same tests a second time". | Yes. |
| `… apply it, look at it, or compose again?` then `Choose [1-3], or press Enter for 2:` | `--pin`, `--transform`, after a declined patch | 2: look at it; nothing changes. | 2. |
| The pins' database: `Your answer:` | `--pin --level database` | `done`: the password stays in the file. | A text answer is required. |
| `Remove the database pins' gate entries now, so that no later increment runs them on the application's database? [Y/n]` | `--pin --level database` | Yes: "they would run on the application's database". | Yes. |
| `Unapply it, putting those files back as they were before the patch? [y/n]` | `/exit` after a `T9` part whose gate did not pass | Nothing: the step is hard to undo. | Nothing: type `y` or `n`. |

## What you see

Every action ends with a result. For `--plan` and `--audit-pack`, the result names the folder that was written. For `--pin` and the in-place `--transform`, the result has a verdict, and the terminal prints a closing line. Inside a stage of the program, that closing line gives the outcome and the reason, with no `; next: …`. The stage's own lines then give the way out. The other actions end with their own last statement as the headline. `--source` takes its first line instead: what it did, such as `There was no run profile to take out: nothing changed`, or its refusal.

### The stops of a pin or a transform

When a pin or a transform does not land, its result says why, by the class of its stop. A choice of yours is `stopped`, never `failed`.

| Class | Outcome | Headline | The way on |
|---|---|---|---|
| The model server did not answer | `failed` | `The model server of the preflight row did not answer: lmstudio at http://localhost:1234/v1` | `Check the model server: codelead doctor`, then `Then run it again: /modernize --pin webapp` |
| You declined the patch | `stopped` | `You declined the patch; nothing changed` | `Apply it as it is: /edit (the patch is kept)`, `Look at it: /patch --show`, or compose it again |
| You declined the validation | `stopped` | `The pins were applied and not validated: the gate did not run` | `Validate them: /validate (a pass lands the increment and declares the pins' gate entry)` |
| Applied, not validated for another reason | `stopped` | `Transform T1 was applied and not validated: the gate did not run` | `Validate it: /validate (a pass lands the transform)` |
| The model's reply was cut at its cap | `failed` | `… did not land`, with the counts | `Pin fewer units in this increment: /modernize --pin webapp --units <unit>` |
| The patch check refused the patch | `failed` | `… did not land: the patch check refused …` | `/patch --retry`, or run it again |
| The gate refused it, and the repair rounds stopped | `failed` | `The pins of webapp did not land: 1 of 8 tests failed: <test>: <message>`, or `…: dotnet test exited 1` when the build failed | `Repair it in this session: /repair …` |
| A precondition | `failed` | For example, `The pins of webapp need Visual Studio's MSBuild, and it is not on this machine` | The precondition's own ways. |
| The preflight objected | `failed` | The objection's words. | Change the request. |

When tests failed, the reason is the count and the first failed test, with its message. It is never a line of a stack. For example:

<!-- described, not captured -->
```output
1 of 8 tests failed: AccountControllerTests.SignOut_RedirectsToSignIn_Lines51_51: System.NullReferenceException: Object reference not set to an instance of an object
```

A run that comes back over an applied increment asks to validate it first. A pin that comes back over an increment whose gate refused it asks to run that gate again first. A run after a declined patch offers to apply it, show it, or compose it again. `--again` composes anew on purpose.

## Examples

- Survey the repository and write the dossier, changing nothing in the code:

  ```console
  /modernize --plan
  ```

- Pin a project, then the facts that need a database:

  ```console
  /modernize --pin webapp
  /modernize --pin webapp --level database
  ```

- Run the program's next stages, as the wizard does:

  ```console
  /modernize --run
  ```

- Refuse an action without its required word, before anything runs:

  ```console
  /modernize --pin
  ```

  ```output
  --pin requires a project name from the survey (see modernization-readiness.md).
  ```

- `/modernize` alone lists what it needs:

  ```console
  /modernize
  ```

  ```output
  /modernize requires --plan, --pin <project>, --transform <type>, --narrate, --rebuild-map, --rebuild-plan, --rebuild-init --out <folder>, --review channel, --testimony <documents>, --waive <finding> --reason <why>, --audit-pack, or the program's --run [<stage>], --continue, --answer <key>=<value>, --skip <stage> --reason <why> or --source ….
  ```

## Files

- `--plan`: writes `docs/codelead/dossier/` (the bundle, `DOSSIER.md`, `intake.md`, `options.md`, `security-exposure.json`, and `redacted/` with `--redacted`).
- `--narrate` and `--testimony`: write `functional-narration.*` and `exception-queue.*` in the bundle.
- `--pin`: writes a test project, adds its entries to `.codelead/validation.json`, and writes `pin-levels.json`. The database level writes a git-ignored connections file and a copy of a file-attached database in the pins' folder.
- `--pin` and `--transform`: write one evidence record per increment to `.codelead/evidence/`.
- `--waive`: writes `docs/codelead/security-waivers.json`.
- `--audit-pack`: writes `docs/codelead/audit-pack/`.
- `--rebuild-map`, `--rebuild-plan`, `--review`: read and write `docs/codelead/rebuild/`.
- `--run`, `--continue`, `--answer`, `--skip`: read and write `.codelead/modernization.json`. `--skip` also writes an evidence record. `--source` writes `modernization.source` in `.codelead/config.json`.
- `--rebuild-init`: writes the rebuild folder, and commits it unless `--no-commit`.

## See also

- [Modernization](../modernization/index.md)
- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)
- [Pins](../modernization/pins.md) and [Transforms](../modernization/transforms.md)
- [The Modernize wizard](../extension/modernize-wizard.md)
- [`codelead modernize`](../cli/codelead-modernize.md)
- [Results and stops](../concepts/results-and-stops.md)
- [`/pin`](pin.md)
