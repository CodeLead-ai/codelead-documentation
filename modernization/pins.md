---
title: Pins
description: What a pin is, why there are two levels, how CodeLead counts what the pins prove, and what to do with each result.
kind: concept
area: modernization
date: 2026-11-20
---

# Pins

A pin is a test that records what the code does today. CodeLead writes the pins before it changes anything. Then it runs them again after every later change. When a change alters what the code does, a pin fails, and the change does not land: CodeLead does not keep it.

**In short**
- You know what a pin is, and why CodeLead writes pins at two levels: in memory and on a database.
- You can read a pin's result: what landed, what is covered, what waits, and what to do next.
- You know that the database pins use a database of their own, never the application's.
- You know what CodeLead refuses, and what each message asks of you.

## Why pins exist

An old application often has no tests. The Basic Store sample has none. Without tests, nobody can show that a change kept the behaviour. Pins are tests written for that purpose: they do not judge whether the behaviour is right. They record it, so that any change to it is seen.

This is why CodeLead pins first. [The gate](../concepts/validation-and-the-gate.md) is the build and the tests that every change must pass. A [transform](transforms.md) is refused while the gate holds no pins: `Pin first: the declared gate (.codelead/validation.json) carries no characterization suite, so no transform can be shown safe.`

## What a pin proves: facts

CodeLead pins **facts**. A fact is one statement about what a unit does, with the numbers of the lines that it describes. A unit is a type of the code, such as a controller class. The [narration](dossier.md#the-narration) writes the facts, with one model call for each unit. There are three kinds that CodeLead pins, in this order:

1. **Rules**: a condition or a calculation that decides what happens. For example, "a product's SKU is unique".
2. **Guards**: a check of an input, answered with an error. For example, "Details without an id answers Bad Request".
3. **Outputs**: what the unit returns or shows.

A fourth kind, the **attribute facts**, are not pinned. These are attributes such as `[Authorize]`, `[HttpPost]` or `[ValidateAntiForgeryToken]`. A test that creates a controller and calls its method directly does not run these attributes. Only the running application can prove them, so [the walk](walk-and-parity.md) proves them.

A fact whose code needs the **web runtime** is not pinned either. For example, a sign-out action calls `FormsAuthentication.SignOut()`. That call reads the web configuration that the ASP.NET host loads. In a test process there is no host, so the call throws, at either level. The walk proves such a fact too.

## The two levels, and why

| Level | What a test calls | What it can prove | On the Basic Store sample |
|---|---|---|---|
| **In memory** (`memory`, the default) | The code, with hand-written fakes in place of what it needs. No database. | Guards, redirects, and outputs that need no stored data. | The missing-id guards of the store's controllers. |
| **Database** (`database`) | The code with its real data layer, on a database of the pins' own. | The rules that depend on stored rows. | "A product's SKU is unique". |
| The walk (no pin level) | The running application, in a browser. | Attribute facts: sign-in, HTTP verbs, anti-forgery tokens. The facts whose code needs the web runtime. | `[Authorize]` on the account pages. |

A rule needs a database when the code cannot be given a fake. Look at the Basic Store's `ProductsController`. It creates its own data context in a field: `private readonly StoreContext db = new StoreContext();`. To refuse a second product with the same SKU, it asks that context: `db.Products.Where(p => p.Sku == sku)`. A test cannot replace the context with a fake, because the controller does not receive it from outside. Only a database with a product in it can show the refusal `Another product already uses this SKU.` So the in-memory level cannot prove this rule, and the database level can.

### How CodeLead decides the level

CodeLead decides the level of each fact before any model call, from the code that the fact rests on:

- A fact whose lines are attribute lines only belongs to **the walk**.
- A fact whose code calls a static service of the web runtime belongs to **the walk** too. Examples are `FormsAuthentication.SignOut()`, `HttpContext.Current` and `Server.MapPath(…)`. Its reason starts with `needs the web runtime:`.
- A fact whose code reaches a data context or a repository that the unit builds itself belongs to the **database** level. So does a rule whose words speak of stored rows: unique, exists, a count or a sum.
- Every other fact belongs to the **in-memory** level.

CodeLead writes each decision, with its reason, to the ledger `pin-levels.json`, beside the narration. A reason reads, for example, `reaches db, the StoreContext OrdersController builds itself (line 12)`.

> [!TIP]
> **For experienced users:** the classification is static. A data store that the code reaches through a container or a static factory counts as an in-memory fact. Read the ledger when a level's choice surprises you.

## Increments, and "Pin the next?"

CodeLead does not pin a whole project in one request. Each **increment** names a few units, six at most by default, so that the model's answer is not too long. It names the units that hold this level's facts first, and it pins rules before guards and guards before outputs.

Each increment is an [`/implement`](../chat-commands/implement.md) run. CodeLead composes the request, the model writes a patch, [the patch check](../concepts/the-patch-check.md) reads it, and you approve it. Then the gate builds and runs the tests. The preflight checks the request before the patch, and it can ask questions. CodeLead answers them itself, and shows what it assumed once, in a card titled `CodeLead assumed`.

When an increment lands, its tests join the gate in `.codelead/validation.json`. CodeLead then offers the next one, for example:

<!-- described, not captured -->
```output
Pin the next: ItemsController, CustomersController (3 facts)? [Y/n]
```

The capital `Y` is CodeLead's recommendation, and Enter gives it. CodeLead stops offering when the level is covered, when you answer no, when a run stops, or after six increments in one run.

## Coverage, and how to read the result

A landed increment says what it added and what is covered so far. It never claims more. These lines come from CodeLead's own tests, on a small project named `Ledger.Web`:

<!-- described, not captured -->
```output
Increment 1 landed: 4 tests on SessionController, 2 on InvoicesController. So far: 2 of 5 units with provable facts have tests; 6 of 19 provable facts pinned (in memory 6 of 9, database 0 of 10). Rules pinned: 2 of 7; guards 2 of 6; outputs 2 of 6. 3 facts wait for the in-memory level; 10 facts wait for the database level; 8 attribute facts are the walk's.
```

Read it part by part:

- `4 tests on SessionController, 2 on InvoicesController`: what this increment added.
- `2 of 5 units with provable facts have tests`: only the units that hold a fact that a pin level can prove are counted.
- `6 of 19 provable facts pinned (in memory 6 of 9, database 0 of 10)`: the facts that a passing test covers, in all and for each level.
- `Rules pinned: 2 of 7`: the rules, counted on their own. They matter most, because they decide what happens.
- `10 facts wait for the database level`: the work that is left, by level.
- `8 attribute facts are the walk's`: facts that no pin proves. The journeys prove them later. When some facts need the web runtime, the line adds them apart, for example `4 facts need the web runtime: the walk's`.

When nothing is left at a level, the headline says so: `The pins of Ledger.Web are complete at the in-memory level (9 tests this run)`. The next step is then the next level: `/modernize --pin Ledger.Web --level database — the 10 facts that need a database; it runs on a database of its own, never the application's`.

> [!NOTE]
> **New to this?** "Complete" means that every provable fact of the level has a passing test. It does not mean that the application is fully tested. The coverage line says exactly what is covered.

## The pins' own database

Database pins write rows and delete them. CodeLead never runs them on the application's own database. A test that stops half-way leaves its rows there, and every later change runs the pins again. Before any model call, CodeLead decides which database the pins use:

- **A database in a file**, for example a LocalDB `.mdf` file under `App_Data`. CodeLead copies the file into the pins' folder, under a name of its own. It asks nothing, and says what it did in three lines. In CodeLead's tests the lines read:

  <!-- described, not captured -->
  ```output
  CodeLead copied the application's database file for the pins: Ledger.Web/App_Data/Ledger.mdf (and its log) → Ledger.Web.Characterization.Db/data/.
  The pins' connection LedgerConnection points at the copy: Ledger.Web.Characterization.Db/connections.local.config (git-ignored, with Ledger.Web.Characterization.Db/data/).
  The application's own database is not touched: the pins write and delete rows in the copy only.
  ```

  The Basic Store sample keeps its databases in such files. Stop the application first: LocalDB keeps a file open while the application runs.

- **A database on a server**: CodeLead asks one question. The question says what the database pins do, and why the application's database will not do. It also says what will do: a restored copy, an empty database with the same schema, or a local instance. It also shows the application's own server and catalog, under "do not give these". CodeLead has already written a git-ignored connections file with a placeholder in the pins' folder. The recommended answer is `done`: you put the copy's connection string in that file, and the password stays there. You can also answer `env:<NAME>`, an environment variable that holds the string.

A password typed into the chat is refused. The result of a landed database pin names the database that it ran against, never its credentials.

**The database provider.** Entity Framework 6 finds its SQL Server provider by itself. It does not find SQLite, SQL Server Compact, Oracle, MySQL or PostgreSQL: the application registers them in its configuration. For a new database pins project on such a provider, the request says how the application registers it. CodeLead reads this from the application's own files, with no model call:

- the `<entityFramework>` section, with its declaration, its providers and its `defaultConnectionFactory`;
- the `<system.data>` section's `<DbProviderFactories>` entries;
- the packages that bring the provider, at their versions;
- the item of the project file that copies the provider's native library, such as `libe_sqlite3.dylib`.

The request quotes no connection string and no password. It says nothing of the provider when the pins project exists already, or when CodeLead can read no registration.

## What CodeLead refuses

CodeLead refuses these before they can do harm. Each refusal says the way out.

- **A pin of a .NET Framework web project, on a machine without Visual Studio's MSBuild**. It stops before any model call, with `The pins of webapp need Visual Studio's MSBuild, and it is not on this machine`. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).
- **A database pin that reads the application's configuration**. The patch check refuses a pins file that reads `Web.config` for a connection, or that names the application's database.
- **A gate entry of database pins that points at the application's database**. The gate does not run it.
- **A request that the preflight objects to**. When the preflight says that the request cannot be done, the pin stops with those words.
- **A wrong level**. `--level must be memory or database.`
- **Pins that cannot run on this machine**. A .NET Framework test project needs the .NET Framework runtime, or Mono. On a machine with neither, the pin stops before any model call, with `The pins of webapp cannot run on this machine`. See [Details](#details).

## The repair rounds

When the gate refuses an increment, CodeLead asks once: `The gate refused the pins: <the failure>. Repair it? [Y/n]`. Then it runs [repair](../concepts/repair.md) rounds by itself: repair, apply, validate. When tests failed, the failure is the count and the first failed test, for example `1 of 8 tests failed: <test>: <message>`.

The limit, `modernization.repair.rounds` (2 by default), counts only the rounds that did not move the failure. A round that ends with a new failure, unlike every earlier one, earns one more round. The rounds never go past twice the limit. For example, with the default of 2:

1. Round 1 fixes a compiler error, and a new failure appears: the failure moved.
2. Round 2 moves the failure again.
3. Neither round counts against the limit, so round 3 runs.

The rounds also stop when the gate passes. They stop when they go in a circle too: a round returns a file, or the gate, to an earlier state. The first circle gets one more round with a wider view, and that round is the last. See [Repair](../concepts/repair.md#cycles). Then CodeLead lists what each round tried. In CodeLead's tests, the lines of such a run read:

<!-- described, not captured -->
```output
Repair round 1 of 2: <the failure after the patch>
Repair round 2 of 2: <the failure after round 1>
Repair round 3 of at most 4 (2 of the 2 before it moved the failure): <the failure after round 2>
Repair rounds: 3 ran (the limit of 2, and 1 more earned by rounds that moved the failure); the gate passed.
```

The closing line says why the rounds ended. With a limit of 1, in CodeLead's tests:

- A round that did not move the failure: `Repair rounds: 1 round ran and the failure did not change: the limit of 1 (modernization.repair.rounds) counts the rounds that did not move it.`
- Rounds that each moved it, up to the cap: `Repair rounds: 2 rounds ran: the cap, twice the limit of 1 (modernization.repair.rounds); each moved the failure.`

A failure of the machine is not repaired: a missing connections file, or a server that does not answer. CodeLead says what is missing instead, because a repair would only hide it.

### A pin that cannot run in the test process

Sometimes a few tests fail because the test process cannot run them, and the repair round says so and leaves no patch. The other tests of the increment pass. CodeLead then offers to set the failing tests aside, so that the rest land:

<!-- described, not captured -->
```output
The gate failed on 1 of the increment's 8 pins, and the repair left no patch: <the repair's reason>.
- AccountControllerTests.SignOut_RedirectsToSignIn_Lines51_51 (<file>): System.NullReferenceException: Object reference not set to an instance of an object
Set the 1 failing pin aside and land the 7 that pass? [Y/n]
```

Yes is recommended, and Enter gives it. CodeLead removes each failing test method, never a whole file unless every test in it failed. Then the gate runs again without them. When it passes, the increment lands, with a line such as `7 pins landed; 1 set aside: <test>: <reason>; the walk or the database level may prove it`.

- **What is kept.** The landed evidence record keeps the text of each test set aside, under `set_aside`. The ledger `pin-levels.json` lists it under `setAside`, and counts a miss for its fact.
- **With nobody present**, an unattended run answers yes, and says so: `The unattended run: the failing pins are set aside, as CodeLead recommends (…)`.
- **A no** sets nothing aside. The pins stay applied and do not land.
- **The limits.** CodeLead offers this once for each increment, and never after a build failure. It never sets aside more than half of the increment's tests: `Not set aside: 5 of the increment's 8 tests failed, more than half: the request was wrong, not a few facts; nothing is set aside.`
- **A second refusal.** When the gate refuses again without them, CodeLead puts the tests back as they were, and nothing lands.

## The Pin stage of the program

In the [program](the-program.md), the Pin stage runs both levels. It runs the in-memory level to its end, then the database level when facts wait for it. When no fact needs a database, it skips the database level and says why. The stage ends `waiting`, never `done`, while provable facts remain at a level. `codelead modernize next` then stops there, so that the transforms do not run on too few pins.

To continue with the pins as they are, answer `codelead modernize answer pins=enough`. CodeLead takes this answer only while the stage waits or failed. The stage then ends `done`, with the coverage it had: `The pins of Ledger.Web: accepted by the person at 3 of 9 facts pinned in memory, 0 of 10 at the database level.`

The stage's summary says what landed in this run, at each level, with the tests that each level added. For example: `3 increments landed in this run: 3 at the database level, 30 tests (…)`. An increment that lands in a repair round counts too.

When the gate refuses an increment and its repair rounds end, the stage fails. Its problem gives three ways out, and each one works from a terminal:

1. Run the stage again: `codelead modernize run pins`. The pins that were applied and did not land stay as they are. The stage runs their gate first, then their repair rounds, then the offer to set aside a pin that cannot run. Nothing is composed over them, so nothing is written twice.
2. Or open the chat in this folder, with `codelead interactive`, and repair there: `/repair`, then `/validate`. A pass lands the increment.
3. Or go on with what is pinned: `codelead modernize answer pins=enough`.

A run of the same pin over such an increment asks first: `An increment is applied and its gate refused it: run its gate again first? [Y/n]`. Yes is recommended. A no composes nothing.

The pins take their facts from the [narration](dossier.md#the-narration). When the web project has none, the stage makes one first. It says what it does, why, and how long it takes. In approve mode it asks once, and yes is recommended:

<!-- described, not captured -->
```output
Narration first: the pins take their facts from it, and webapp has none. CodeLead narrates it (/modernize --narrate --project webapp): a model call for each of its 51 units; how long it takes is not known yet (no narration here has recorded its time per unit).
Narrate webapp now? [Y/n]
```

In unattended mode, the stage narrates without asking. The narration's files then join the stage's outputs. Sometimes the survey lists no unit of the project, for example because the project was not built. Then there is nothing to narrate, and the stage says so.

> [!IMPORTANT]
> When you answer no, or when no answer can come, the stage runs the in-memory pins once. Its summary says `Coverage in facts is not available and the database level cannot be planned: webapp has no narration (you declined it; /modernize --narrate --project webapp makes it)`. The database level, where the store's rules are, then does not run. See [Narrate the code before you pin it](../how-to/narrate-before-pins.md).

## You see, it means, do

| You see | It means | Do |
|---|---|---|
| `Narrate webapp now? [Y/n]` (the Pin stage) | The web project has no narration, and the pins take their facts from it. | Press Enter (yes). The narration makes one model call for each unit. |
| `Apply this patch? [Y/n]` with `The patch check passed with 1 note. CodeLead recommends applying.` | The patch is safe to apply. A note does not block. | Press Enter (yes). |
| `The pins of webapp: this increment landed (3 tests)` | The tests passed and joined the gate. | Read the `So far:` line. Answer the next offer. |
| `Pin the next: <units> (<n> facts)? [Y/n]` | Provable facts remain at this level. | Press Enter, until the level is complete. |
| `The pins of webapp are complete at the in-memory level` | Every provable in-memory fact has a test. | Run `/modernize --pin webapp --level database`. |
| `The database pins need a database that is not the application's` | The database is on a server, and the pins have none of their own yet. | Put a copy's connection string in the file that CodeLead prepared, and answer `done`. |
| `The pins of webapp need Visual Studio's MSBuild, and it is not on this machine` | The project builds only with Visual Studio's MSBuild. | Run it on Windows with Visual Studio or its Build Tools. |
| `The pins of webapp cannot run on this machine` | The project targets .NET Framework, and this machine has neither the .NET Framework runtime nor Mono. Nothing was asked of the model. | Install Mono (on macOS: `brew install mono`), then run the pin again. Or pin on Windows. Or declare the pins' commands in `.codelead/validation.json`. |
| `The gate refused the pins: … Repair it? [Y/n]` | The tests do not build or do not pass yet. | Press Enter. CodeLead runs the repair rounds by itself. See [The repair rounds](#the-repair-rounds). |
| `Set the 1 failing pin aside and land the 7 that pass? [Y/n]` | One test cannot run in the test process, and its repair left no patch. The other tests pass. | Press Enter. The passing pins land, and the text of the test set aside stays on the evidence record. |
| `The pins of webapp did not land: 1 of 8 tests failed: <test>: <message>` | The gate still refuses after the repair rounds. | Run the stage again, or repair in the chat, or answer `pins=enough`. See [The Pin stage of the program](#the-pin-stage-of-the-program). |
| `An increment is applied and its gate refused it: run its gate again first? [Y/n]` | You run the pin again over an increment that did not land. | Press Enter. The gate runs first, and nothing is composed twice. |
| `The repair loop stopped: this repair returns <file> to its state after repair 1, …` | The repairs went in a circle. | Read what each round tried. Fix it by hand, or with [`/repair`](../chat-commands/repair.md). |
| `The model's reply was cut at its cap: …` | The request was too big for one reply. CodeLead already tried once more. | Pin fewer units: `/modernize --pin webapp --units <one unit>`. |
| `You declined the patch; nothing changed` | You answered no. The patch is kept in the session. | Apply it with [`/edit`](../chat-commands/edit.md), or compose again with `--again`. |
| `The pins were applied and not validated: the gate did not run` | You answered no to the validation. | Run [`/validate`](../chat-commands/validate.md). Do not run the pin again. |
| `The model server of the preflight row did not answer: …` | The model server is down. | Run [`codelead doctor`](../cli/codelead-doctor.md), then run the pin again. |
| `6 facts still wait at the in-memory level` (the Pin stage, waiting) | The stage reached its increment cap with work left. | Run the stage again, or answer `pins=enough`. |

Every message is also in [Messages and what to do](../reference/messages.md).

## Details

The configuration keys, in `.codelead/config.json`:

| Key | What it sets | Default |
|---|---|---|
| `modernization.pins.unitsPerIncrement` | The units that one increment names. | 6 |
| `modernization.pins.maxIncrementsPerRun` | The increments that one `/modernize --pin` runs. | 6 |
| `modernization.pins.maxIncrementsPerStage` | The increments of the Pin stage in unattended mode. | 24 |
| `modernization.pins.patchRole` | The role whose model writes the pins' patch. | See below |
| `modernization.repair.rounds` | The repair rounds after the gate refuses that did not move the failure. A round that moves it earns one more, up to twice this number. | 2 |

- **The commands**: [`/modernize --pin <PROJECT>`](../chat-commands/modernize.md#--pin) with `--level memory|database`, `--units <TYPE>,...`, `--facts <WORDS>` and `--again`. In the program: `codelead modernize run pins`.
- **The test projects**: `<project>.Characterization/` (in memory) and `<project>.Characterization.Db/` (database), beside the project. For a project that targets only .NET Framework, they target `net48`.
- **The test form on a machine without the .NET Framework**. CodeLead decides the form of a .NET Framework test project by what the machine has, never by the system's name alone:
  - **The .NET Framework 4 runtime is installed** (its `clr.dll` under the Windows folder's `Microsoft.NET\Framework64\v4.0.30319`, or `Framework\v4.0.30319`). The pins use xunit 2 with the VSTest runner, as before. Windows has it, so nothing changes on Windows.
  - **The runtime is not installed, and Mono is on the `PATH`**, as on macOS or Linux. The pins use xunit.v3 3.2.2 on the Microsoft Testing Platform: the test project is an executable (`OutputType` `Exe`, `TestingPlatformDotnetTestSupport`). The gate runs it with the same `dotnet test <project> -nologo`, and the .NET SDK runs it with Mono. An existing classic pins project is converted to this form.
  - **Neither**. The pins are refused before any model call, with the ways out: install Mono, pin on Windows, or declare the pins' commands in `.codelead/validation.json`. Commands that you declare there are used as they are, and the pin is not refused.
- **The gate entries**: labelled with their project, for example `pins build: webapp.Characterization (MSBuild, net48)`, `pins in-memory: …` and `pins database: …`.
- **The ledger**: `docs/codelead/dossier/pin-levels.json` (`codelead.pin-levels/1`). It holds each fact, its unit, its lines, its level and why, and the tests set aside. When two database increments name a fact and no test for it lands, the fact is left for a person. See [Pin levels file](../reference/pin-levels-file.md).
- **The model**: without `patchRole`, the pins' patch goes to the repair role's model. It does so when that model is remote and remote models are allowed. See [Models and roles](../concepts/models-and-roles.md).
- **The cost**: one recorded run on the Basic Store sample made 14 calls to a remote model, for about 13 US dollars. That run was made before several fixes of 2026-11-20, so read it as a scale, not as a price.
- **The evidence**: each increment writes an [evidence record](../concepts/evidence-records.md), with the database that a database pin ran against.

## What a pin is not

- **A pin is not a judgment**. It records what the code does, right or wrong. A pin that records a defect keeps the defect until a person decides.
- **A pin is not `/pin`**. The chat command [`/pin`](../chat-commands/pin.md) keeps a list of what waits on a person. `/modernize --pin` writes tests.
- **The pins are not the whole proof**. Attribute facts, and anything that only the running application shows, are proved by [the walk](walk-and-parity.md) and the journeys.

## Related content

- [The eight stages](the-eight-stages.md#3-pin)
- [`/modernize --pin`](../chat-commands/modernize.md#--pin)
- [The dossier](dossier.md): the narration and its facts
- [Transforms](transforms.md): they run under the pins
- [Validation and the gate](../concepts/validation-and-the-gate.md)
- [Repair](../concepts/repair.md)
- [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md)
- [Build .NET Framework projects on Windows](../how-to/build-on-windows.md)
- [Glossary](../concepts/glossary.md)
