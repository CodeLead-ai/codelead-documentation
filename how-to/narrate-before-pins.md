---
title: Narrate the code before you pin it
description: Write the narration of an old application's web project before its pins, by hand or when the program's Pin stage asks, so that the pins prove facts and can plan the database level.
kind: how-to
area: how-to
date: 2026-11-20
---

# Narrate the code before you pin it

This guide writes the **narration** of a web project before its pins. The narration states what each unit of the code does, one plain statement per fact. The [pins](../modernization/pins.md) (tests that record what the code does today) take their facts from it. Without a narration, the Pin stage pins once in memory, counts no facts, and cannot plan the database level.

**In short**
- You narrate by hand with [`/modernize --narrate`](../chat-commands/modernize.md#--narrate) `--project webapp`, or you answer yes to the Pin stage's question `Narrate webapp now? [Y/n]`.
- The narration makes one model call for each unit of the code, and writes `functional-narration.json` and `functional-narration.md`.
- After it, the Pin stage counts its coverage in facts, and runs the database level when facts need it.

## Before you begin

- The dossier exists: the program's Read stage ran, or [`/modernize --plan`](../chat-commands/modernize.md#--plan) ran. See [The dossier](../modernization/dossier.md).
- The web project was built before the survey. The survey reads the built assemblies to list the units. For a .NET Framework project, build on Windows. See [Build .NET Framework projects on Windows](build-on-windows.md).
- A model is set up. The narration runs on the `architect` [role](../concepts/models-and-roles.md). See [Set up local models](set-up-local-models.md).

> [!NOTE]
> **New to this?** A *unit* is a part of the code that the survey found: a controller action, a handler or a data context. A *fact* is one statement about what a unit does, with the numbers of the lines that it describes.

## Narrate by hand

Use this way when you want the narration before you start the program, or to narrate again after a change.

1. Open the chat in the old application's folder, or run [`codelead interactive`](../cli/codelead-interactive.md) there.
2. Narrate the web project:

   ```console
   /modernize --narrate --project webapp
   ```

3. Read the first line. It says how many units the survey found, and that each one costs a model call:

   <!-- described, not captured -->
   ```output
   Narration: 51 units from the structural inventory (project webapp); one model call each.
   [1/51] <unit>: 2 statements
   …
   Narration written: docs/codelead/dossier/functional-narration.md — <n> statements over 51 units (<n> rules, <n> data, <n> outputs), <n> discarded.
   ```

   On the Basic Store sample, a recorded run narrated 51 units in about 11 minutes on a local model.

When the survey listed no unit, nothing is narrated, and no model is called. Here with `--limit 2`:

```console
/modernize --narrate --limit 2
```

```output
Narration: 0 units from the structural inventory, first 2; one model call each.
Nothing to narrate: the structural inventory lists no action, handler or EF context with a declaring source file.
```

Build the project, run the Read stage again ([`codelead modernize run`](../cli/codelead-modernize-run.md) `read`), then narrate.

> [!TIP]
> **For experienced users:** `--limit <N>` narrates the first N units only. Use it to see the cost and the quality of the statements on two or three units first.

## Let the Pin stage narrate

The program's Pin stage checks for a narration before it pins. When the web project has none, it says what it will do, why, and how long it takes:

<!-- described, not captured -->
```output
Narration first: the pins take their facts from it, and webapp has none. CodeLead narrates it (/modernize --narrate --project webapp): a model call for each of its 51 units; how long it takes is not known yet (no narration here has recorded its time per unit).
Narrate webapp now? [Y/n]
```

1. Press **Enter**. Yes is the recommended answer, "the pins take their facts from it".
2. Wait while CodeLead narrates the project. Then the stage pins on the narration's facts.

A narration records how long its units took. A later narration in the same folder then gives an estimate, such as `about 11 min (51 units × 13 s a unit, as the last narration here took)`.

In unattended mode, the stage narrates without asking.

## Know what happens without a narration

When you answer no, the stage pins once in memory, as before. Its summary says what is missing and how to make it:

<!-- described, not captured -->
```output
… Coverage in facts is not available and the database level cannot be planned: webapp has no narration (you declined it; /modernize --narrate --project webapp makes it).
```

The pins still land, and the gate runs them. But nothing counts what they prove, and the store's rules at the database level stay unpinned.

When the stage cannot ask, for example when `codelead modernize` runs with no terminal attached, the question is declined. The words in parentheses then read `no answer could come here` in place of `you declined it`.

## Check that it worked

- `docs/codelead/dossier/functional-narration.md` exists, and lists statements for the units of `webapp`.
- After the Pin stage, [`codelead modernize status`](../cli/codelead-modernize-status.md) shows a `Coverage:` line under the stage, in facts.
- The Pin stage's outputs list `functional-narration.json` and `functional-narration.md` when the stage made them.

## Details

- **The files.** `functional-narration.json` and `functional-narration.md`, in `docs/codelead/dossier/`. Every statement starts as `Inferred`. A pin that proves it changes it to `Pinned`.
- **What is dropped.** A statement whose lines fall outside its unit is dropped, and counted as `discarded`.
- **A narration that states nothing.** The narration can write no fact of the project. The stage then says so, and pins as it would without a narration.
- **No unit to narrate.** The stage says `Narration: not made: webapp has no narration, and the survey's structure lists no unit of it to narrate`. Build the project, run the Read stage again, then the Pin stage.
- **Cost.** One model call for each unit. With a paid provider, a narration costs money. See [Set up remote models](set-up-remote-models.md#read-the-cost-of-the-calls).

## See also

- [Pins](../modernization/pins.md)
- [The dossier](../modernization/dossier.md#the-narration)
- [`/modernize`](../chat-commands/modernize.md#--narrate)
- [The modernization program](../modernization/the-program.md)
- [The eight stages](../modernization/the-eight-stages.md)
- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)
