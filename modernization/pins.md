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

## The two levels, and why

| Level | What a test calls | What it can prove | On the Basic Store sample |
|---|---|---|---|
| **In memory** (`memory`, the default) | The code, with hand-written fakes in place of what it needs. No database. | Guards, redirects, and outputs that need no stored data. | The missing-id guards of the store's controllers. |
| **Database** (`database`) | The code with its real data layer, on a database of the pins' own. | The rules that depend on stored rows. | "A product's SKU is unique". |
| The walk (no pin level) | The running application, in a browser. | Attribute facts: sign-in, HTTP verbs, anti-forgery tokens. | `[Authorize]` on the account pages. |

A rule needs a database when the code cannot be given a fake. Look at the Basic Store's `ProductsController`. It creates its own data context in a field: `private readonly StoreContext db = new StoreContext();`. To refuse a second product with the same SKU, it asks that context: `db.Products.Where(p => p.Sku == sku)`. A test cannot replace the context with a fake, because the controller does not receive it from outside. Only a database with a product in it can show the refusal `Another product already uses this SKU.` So the in-memory level cannot prove this rule, and the database level can.

### How CodeLead decides the level

CodeLead decides the level of each fact before any model call, from the code that the fact rests on:

- A fact whose lines are attribute lines only belongs to **the walk**.
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
- `8 attribute facts are the walk's`: facts that no pin proves. The journeys prove them later.

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

## What CodeLead refuses

CodeLead refuses these before they can do harm. Each refusal says the way out.

- **A pin of a .NET Framework web project, on a machine without Visual Studio's MSBuild**. It stops before any model call, with `The pins of webapp need Visual Studio's MSBuild, and it is not on this machine`. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).
- **A database pin that reads the application's configuration**. The patch check refuses a pins file that reads `Web.config` for a connection, or that names the application's database.
- **A gate entry of database pins that points at the application's database**. The gate does not run it.
- **A request that the preflight objects to**. When the preflight says that the request cannot be done, the pin stops with those words.
- **A wrong level**. `--level must be memory or database.`

## The repair rounds

When the gate refuses an increment, CodeLead asks once: `The gate refused the pins: <the failure>. Repair it? [Y/n]`. Then it runs up to two [repair](../concepts/repair.md) rounds by itself: repair, apply, validate. It stops when the gate passes, when a round would return a file to a state that an earlier round tried, or at the limit. Then it lists what each round tried.

A failure of the machine is not repaired: a missing connections file, or a server that does not answer. CodeLead says what is missing instead, because a repair would only hide it.

## The Pin stage of the program

In the [program](the-program.md), the Pin stage runs both levels. It runs the in-memory level to its end, then the database level when facts wait for it. When no fact needs a database, it skips the database level and says why. The stage ends `waiting`, never `done`, while provable facts remain at a level. `codelead modernize next` then stops there, so that the transforms do not run on too few pins.

To continue with the pins as they are, answer `codelead modernize answer pins=enough`. CodeLead takes this answer only while the stage waits or failed. The stage then ends `done`, with the coverage it had: `The pins of Ledger.Web: accepted by the person at 3 of 9 facts pinned in memory, 0 of 10 at the database level.`

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
| `The gate refused the pins: … Repair it? [Y/n]` | The tests do not build or do not pass yet. | Press Enter. CodeLead runs up to two repair rounds. |
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
| `modernization.repair.rounds` | The repair rounds after the gate refuses. | 2 |

- **The commands**: [`/modernize --pin <PROJECT>`](../chat-commands/modernize.md#--pin) with `--level memory|database`, `--units <TYPE>,...`, `--facts <WORDS>` and `--again`. In the program: `codelead modernize run pins`.
- **The test projects**: `<project>.Characterization/` (in memory) and `<project>.Characterization.Db/` (database), beside the project. For a project that targets only .NET Framework, they target `net48`.
- **The gate entries**: labelled with their project, for example `pins build: webapp.Characterization (MSBuild, net48)`, `pins in-memory: …` and `pins database: …`.
- **The ledger**: `docs/codelead/dossier/pin-levels.json` (`codelead.pin-levels/1`). It holds each fact, its unit, its lines, its level and why. When two increments name a fact and no test for it lands, the fact is left for a person.
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
