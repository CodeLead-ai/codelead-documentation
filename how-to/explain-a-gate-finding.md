---
title: Explain a finding of the extraction gate
description: Read a finding of the extraction gate, record why it is intended under its key in extraction-dispositions.json, and read the System Model again until the gate passes.
kind: how-to
area: how-to
date: 2026-11-20
---

# Explain a finding of the extraction gate

This guide explains a finding of the **extraction gate**: the check of the [System Model](../modernization/system-model.md), CodeLead's map of an old application read from its code. A finding is a hole in the map that nobody explained, such as a page that nothing links to. You explain it with a reason, under its key. The rebuild is planned only when the gate passes.

**In short**
- `docs/codelead/rebuild/extraction-gate.md` lists each finding with its key.
- You write a reason under the key in `docs/codelead/rebuild/extraction-dispositions.json`.
- You read the model again with [`codelead model`](../cli/codelead-model.md), or [`codelead modernize next`](../cli/codelead-modernize-next.md) in the program.
- A reason explains a finding. It never changes what the model says.

## Before you begin

- The System Model was read: the program's Model stage ran, or you ran `codelead model --source . --project webapp` in the old application's folder. It calls no model.
- You know the application, or someone who knows it helps you. A reason is a statement about the application, and a reviewer reads it.

> [!NOTE]
> **New to this?** An *orphan* is an item that nothing live reaches, and that CodeLead cannot call dead either. For example, a page of the ASP.NET Identity template that no link of the store opens. CodeLead does not guess. It asks a person why the item is there.

## Read the findings

1. Read the model. On a copy of the Basic Store sample:

   ```console
   codelead model --source . --project webapp
   ```

   ```output
   System Model of webapp (basic-store/WebApplication1, inventory: project-file): 289 nodes, 189 edges; 63 entry points, 154 caller records; 10 not resolved, 0 leading nowhere.
   Recipes: aspnet-mvc, jquery-browser. Passes: server, surfaces, client, channel ran.
   Gate: not passed: 36 findings to resolve (36 unexplained-orphan).
   …
   Written all the same: the model is the evidence the findings are read against (extraction-gate.md names each, with the key to record a reason under).
   ```

   The exit code is `1`: the gate did not pass. The model is written all the same.

2. Open `docs/codelead/rebuild/extraction-gate.md`. Each rule has a section, with its open and explained findings. Each finding ends with its key:

   ```output
   - entry-point account.confirm-email is neither live nor dead: what could not be resolved at WebApplication1/Controllers/AccountController.cs:450, … could still reach it (`WebApplication1/Controllers/AccountController.cs:178`). Key: `unexplained-orphan:GET /Account/ConfirmEmail`
   ```

3. Decide for each finding whether it is intended. Here the page is part of the ASP.NET Identity template, and nothing in the store links to it. That is intended.

When a finding is not intended, the model found a real hole, such as a route that the code cannot reach. Fix the code, or tell the team, before you write a reason.

## Record the reason

1. Create the file `docs/codelead/rebuild/extraction-dispositions.json`, or open it.
2. Write the reason under the key, word for word as the report prints it:

   ```json
   {
     "schema": "codelead.extraction-dispositions/1",
     "dispositions": {
       "unexplained-orphan:GET /Account/ConfirmEmail": "ASP.NET Identity template page; no link of the store reaches it"
     }
   }
   ```

3. Read the model again:

   ```console
   codelead model --source . --project webapp
   ```

   ```output
   …
   Gate: not passed: 35 findings to resolve (35 unexplained-orphan).
   …
   ```

   The finding moved to **Explained**, with your reason, in `extraction-gate.md`. Repeat for each open finding, until the line says `Gate: passed.`

One page can give several findings: its action, its view and the links in the view. Each one has its own key.

## Continue the program

In the modernization program, the Model stage waits on the findings that are yours, with the title `The extraction gate needs your reasons`. When you have written the reasons, run:

```console
codelead modernize next
```

The stage reads the model again, and goes on when the gate passes. The program writes the reasons of the ASP.NET Identity template by itself, when every open finding is such a page. See [The System Model](../modernization/system-model.md#explaining-a-finding).

## Check that it worked

- `codelead model` prints `Gate: passed.`, and its exit code is `0`.
- `extraction-gate.md` lists your reasons under **Explained**, and no finding under the blocking rules.
- No warning says that a recorded reason explains nothing. A key with a typing error explains nothing, and the gate says `1 warning`:

  ```output
  Gate: not passed: 35 findings to resolve (35 unexplained-orphan); 1 warning.
  ```

  Rule 10 of the report, "A recorded reason that explains nothing", then lists it, with the key that the reason is recorded under. Here the file holds two such reasons, a key with a typing error and an empty reason:

  ```output
  ## 10. A recorded reason that explains nothing (warning): 2 open, 0 explained

  - the reason recorded under "unexplained-orphan:GET /Account/ConfirmEmal" explains no finding: the finding is gone, or the key is not one the gate report prints (`docs/codelead/rebuild/extraction-dispositions.json`)
  - the reason recorded under "unexplained-orphan:GET /Account/ForgotPassword" holds no reason: a recorded reason is a text (`docs/codelead/rebuild/extraction-dispositions.json`)
  ```

  Find that key in `extraction-dispositions.json`, and copy the key from the report again. A key that names several findings says `explains nothing: <n> findings carry this subject`, and names the keys to write instead.

> [!TIP]
> **For experienced users:** to try reasons without writing the model, add `--no-write`. The command reads the reasons, prints the gate's line, and writes nothing.

## Details

- **The key.** `<rule>:<subject>`, as the report prints it. When two findings of one rule share a subject at two places, the report prints `<rule>:<subject>@<file>:<line>` for each.
- **A reason on a place.** An orphan that only unresolved places keep open can be explained on those places: `unexplained-orphan:<file>:<line>`. The reason must be on every place that keeps it. A reason on one place of five explains nothing, and the gate warns.
- **An empty reason** explains nothing.
- **Rule 6**, "the extraction read the application", takes no reason. Its finding is a fact of the reading, not a judgement.
- **Who reads the reasons.** The file is committed with the old application. The rebuild's plan and the audit pack read it. Write each reason for a reviewer. See [Extraction dispositions and security waivers](../reference/dispositions-and-waivers.md).
- **Exit codes** of `codelead model`: `0` the gate passed, `1` it did not, `2` nothing could be read or written.

## See also

- [The System Model](../modernization/system-model.md)
- [`codelead model`](../cli/codelead-model.md)
- [The eight stages](../modernization/the-eight-stages.md#5-model)
- [The modernization program](../modernization/the-program.md)
- [Extraction dispositions and security waivers](../reference/dispositions-and-waivers.md)
- [The rebuild](../modernization/rebuild.md)
