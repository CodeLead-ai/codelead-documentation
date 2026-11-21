---
title: Pin levels file
description: The format of pin-levels.json, codelead.pin-levels/1 - the ledger that says, for each project, which facts each pin level leaves, the facts that wait for the walk or a person, the misses and the increments.
kind: reference
area: reference
date: 2026-11-20
---

# Pin levels file

**Applies to:** CodeLead 0.1 and later

A [pin](../modernization/pins.md) is a test that records what the old code does today. Pins come in two **levels**: `memory`, tests that run without a database, and `database`, tests that run against a copy of the database. Before any model call, CodeLead gives each fact of the narration the level that can prove it. `pin-levels.json` is the ledger of that work. For each project, it lists the facts that wait for the database level, and those that only the walk proves. It lists the facts left for a person, the misses, and every increment that ran.

> [!NOTE]
> **New to this?** A *fact* is one statement of the narration about one unit of code. For example: "The order must hold at least one line". The narration is in `functional-narration.json`. See [Narrate the code before you pin it](../how-to/narrate-before-pins.md).

## The file

| Fact | Value |
|---|---|
| Path | `docs/codelead/dossier/pin-levels.json`, beside `functional-narration.json`. With `/modernize --plan --out <folder>`, in that folder. |
| Schema | `codelead.pin-levels/1`. |
| Written by | [`/modernize --pin`](../chat-commands/modernize.md#--pin) and the program's Pin stage: before each increment runs, and again after it. |
| Read by | The next pin request, the Pin stage's coverage, and [`codelead modernize status`](../cli/codelead-modernize-status.md). |
| In git | Yes, with the dossier. |

A missing or unreadable file reads as an empty ledger. CodeLead never fails on it.

## How a fact gets its level

| Level | A fact gets it when |
|---|---|
| `walk` | Its lines are attribute lines only: an authorization, a verb, an anti-forgery token. A unit test runs past them. The [walk](../modernization/walk-and-parity.md) proves them. |
| `database` | Its path reaches a data context or a repository that the unit builds itself, with no constructor to inject it. Or it is a rule about stored rows: unique, exists, a count or a sum over stored rows. |
| `memory` | Everything else: a member whose dependencies can be injected, or that has none. |

Each fact also has a **category**. The category orders what a level pins first:

1. `rule`: a condition or a calculation that decides what happens.
2. `guard`: a check of an input, answered with an error.
3. `output`: what the unit returns or shows.
4. `attribute`: the walk's, never pinned.

A fact that two increments of one level named, with no test landing for it, leaves that level. From `memory` it goes to `database`. From `database` it goes to a person.

## An example

The ledger of the Basic Store sample's web project, after one in-memory increment:

<!-- described, not captured -->
```json
{
  "schema": "codelead.pin-levels/1",
  "projects": {
    "webapp": {
      "updatedAt": "2026-11-20T09:41:02.000Z",
      "deferred": [
        {
          "id": "webapp/OrdersController.Create#4",
          "type": "OrdersController",
          "member": "Create",
          "file": "WebApplication1/Controllers/OrdersController.cs",
          "lines": [61, 62],
          "text": "An order line names a product that exists in the store",
          "category": "rule",
          "reason": "reaches db, the StoreContext OrdersController builds itself (line 12)",
          "waitsFor": "database",
          "deferredBy": "classification"
        }
      ],
      "walk": [
        {
          "id": "webapp/OrdersController.Create#1",
          "type": "OrdersController",
          "member": "Create",
          "file": "WebApplication1/Controllers/OrdersController.cs",
          "lines": [48],
          "text": "Only a signed-in user can create an order",
          "category": "attribute",
          "reason": "rests on [Authorize] only: a test that calls the method runs past it; the walk proves it"
        }
      ],
      "missed": {
        "webapp/OrdersController.Create#2": { "memory": 1 }
      },
      "person": [],
      "increments": [
        {
          "level": "memory",
          "at": "2026-11-20T09:41:02.000Z",
          "evidence": "ev-20261120094102-r5j8qd",
          "units": ["OrdersController"],
          "facts": ["webapp/OrdersController.Create#2", "webapp/OrdersController.Create#3"],
          "landed": true,
          "tests": 3,
          "pinned": ["webapp/OrdersController.Create#3"]
        }
      ]
    }
  }
}
```

## Keys

| Key | Type | What it holds |
|---|---|---|
| `schema` | string | `codelead.pin-levels/1`. |
| `projects` | object | One ledger for each project, by the project's name. |

### A project's ledger

| Key | Type | What it holds |
|---|---|---|
| `updatedAt` | string | When the ledger last changed. |
| `deferred` | list of facts | The facts that no in-memory pin can prove, waiting for the database level. |
| `walk` | list of facts | The attribute facts. The walk proves them, and no pin level spends its budget on them. |
| `missed` | object | By fact id: how many increments of each level named the fact and landed no test for it, as `{ "memory": 1, "database": 0 }`. |
| `person` | list of facts | The facts that two database increments named with no test: left for a person. |
| `increments` | list | Every pin increment of the project, in order. |

`deferred` and `walk` are written again from the narration at each run: a fact that is pinned leaves them. `missed` and `increments` grow.

### A fact

| Key | Type | What it holds |
|---|---|---|
| `id` | string | `<narration unit id>#<n>`: the n-th statement of the unit, from 1. It stays stable while the narration does. |
| `type` | string | The class or module. |
| `member` | string | The method, when there is one. |
| `file` | string | The source file, relative to the repository. |
| `lines` | list of numbers | The lines that the fact rests on. |
| `text` | string | The fact, in the narration's words. |
| `category` | string | `rule`, `guard`, `output` or `attribute`. |
| `reason` | string | Why this level, in a few words. |
| `waitsFor` | `database` | On a deferred fact only. |
| `deferredBy` | string | On a deferred fact only: `classification` (no in-memory increment ran yet), `memory` (an in-memory increment left it), or `misses` (two in-memory increments named it with no test). |

### An increment

| Key | Type | What it holds |
|---|---|---|
| `level` | `memory` or `database` | The level of the increment. |
| `at` | string | When it ran. |
| `evidence` | string | Its [evidence record](evidence-record-format.md). |
| `units` | list of strings | The units that it named. |
| `facts` | list of strings | The facts that it named. |
| `landed` | boolean | Whether it landed. |
| `tests` | number | The tests that it added, counted from the sources. |
| `pinned` | list of strings | Of the facts that it named, those that the narration marks pinned once it landed. |

## What reads the ledger

- **The next pin request** names the facts of its level in category order. It leaves out the deferred and walk facts, and a fact that left its level.
- **The Pin stage** counts its coverage from it: units with tests, facts pinned of provable, rules pinned of provable. See [Modernization state file](modernization-state-file.md#the-coverage-of-the-pins).
- **A person** reads `person` for the facts that no pin could prove, and decides what to do with each.

## See also

- [Pins](../modernization/pins.md)
- [`/modernize --pin`](../chat-commands/modernize.md#--pin)
- [Narrate the code before you pin it](../how-to/narrate-before-pins.md)
- [The dossier](../modernization/dossier.md)
- [Modernization state file](modernization-state-file.md)
- [Configuration file](configuration.md#modernization): `modernization.pins`
