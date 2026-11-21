---
title: Conventions
description: Learn the two layers of a repository's conventions, the rules you write and the habits CodeLead reads from the code, and how new code is held to them.
kind: concept
area: concepts
date: 2026-11-20
---

# Conventions

Conventions are the rules that a repository's code follows: how files are named, where they live, how the code is formatted, how tests are written. CodeLead knows them in two layers. The first layer is the rules that you write, in plain words. The second layer is the habits that CodeLead reads from the code itself, each with its evidence. Both go into every change that CodeLead asks a model for.

**In short**
- You write your rules in `.codelead/CONVENTIONS.md`. CodeLead gives them to the model word for word, and they win over the second layer.
- [`codelead conventions`](../cli/codelead-conventions.md) reads the code's habits into `.codelead/conventions.json`, with the files that show each rule.
- The [patch check](the-patch-check.md) compares the new lines of a patch with these habits. It writes a note for each difference. A note never blocks a patch.
- "Where the file you edit differs from a rule, follow the file you edit."

## Why it exists

A model writes code in the style it learned elsewhere. In your repository that code looks foreign: a different naming, another folder, a new pattern. A reviewer then spends time on style, not on behaviour. Conventions tell the model what "the way we do it here" is, before it writes. The check after the patch shows where it still differs.

## How it works

### Layer one: the rules you write

CodeLead reads the first of these files that holds text, and gives it to the model as it is:

1. `.codelead/CONVENTIONS.md`
2. `CONVENTIONS.md`, `STYLEGUIDE.md`, `STYLE_GUIDE.md`, `CODING_GUIDELINES.md`, `CODING-GUIDELINES.md`
3. `docs/STYLEGUIDE.md`, `docs/CONVENTIONS.md`
4. `CONTRIBUTING.md`, `.github/CONTRIBUTING.md`

The first 4,000 characters are used. In the [extension](../extension/settings.md), **Settings → Conventions → Create** writes a short starter file.

### Layer two: the habits of the code

`codelead conventions` samples up to 300 source files (C#, TypeScript and JavaScript) and counts how the code does each thing. A habit that 70% or more of the cases follow becomes a rule. A habit between 30% and 70% is reported as **mixed**, and is never stated as a rule. Each rule keeps the files it was read from. A real run on the Basic Store sample:

```console
codelead conventions
```

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
…
- Control statements always take braces, on their own line (e.g. WebApplication1/App_Start/IdentityConfig.cs)
…
Instruction: new code follows these conventions and the exemplars. Where the file you edit differs from a rule, follow the file you edit.

16 rules from 31 sampled files. Written to <your project>/.codelead/conventions.json.
```

This text is the **conventions card**. The patch prompt of [`/implement`](../chat-commands/implement.md) carries it. For the files that the plan expects to change, the card also shows exemplars: files of the same kind, as a pattern.

### The check on new lines

The [patch check](the-patch-check.md) reads the lines that a patch adds, and the files that it creates, against the second layer. Each difference is a note that names the rule and its evidence. A real note, for a patch that adds a one-line `if` to the sample's `OrdersController.cs`:

```output
[conventions] WebApplication1/Controllers/OrdersController.cs: a control statement without braces: `if (!seenProducts.Add(line.ProductId.Value)) ModelState.AddM` — the code base's rule: Control statements always take braces, on their own line (e.g. WebApplication1/App_Start/IdentityConfig.cs)
```

The note appears under "Notes (they do not block applying)". CodeLead still recommends applying the patch. You decide.

> [!NOTE]
> **New to this?** A note is information for you, not a refusal. The change works or fails at [the gate](validation-and-the-gate.md). The note tells you where the new code looks different from the code around it.

## What it is not

- **Not a linter**. The check knows a short list of mechanical rules. For C#: the names of private fields, interfaces, async methods and tests, braces and namespaces. For TypeScript and JavaScript: the import style, the exports and the names of new files. For every language: the place of new files and of new tests. Your own linter stays the judge of style. Declare it in the gate if you want style to block.
- **Not a check of your written rules**. The rules you write reach the model as words. The check tests only the habits read from the code.
- **Not a rule for mixed habits**. A mixed habit is never checked. Lines that the patch did not add are never judged. A rule that the edited file already breaks is not held against the new lines.

## Details

- **Files**. `.codelead/conventions.json` (version 1) holds the rules, each with `area`, `statement`, `rate`, `sampled`, `evidence` and `mixed`, and the written rules' file in `overrides`.
- **When it is read**. Each patch prompt reads the code again, and stores the profile when it changed. The patch check uses the stored profile, or reads the code when there is none. `CODELEAD_CONVENTIONS_CARD=0` leaves the card out of the patch prompt.
- **Options**. `--for <path>` shows the exemplars chosen for a file that the next change would add. `--json` prints the profile. `--no-write` writes nothing.
- **For other agents**. `codelead conventions --agents` writes the conventions, the validation gate and the project's documents into `AGENTS.md` and `CLAUDE.md`, in CodeLead's own block. Your text in those files is kept.
- **A failing linter**. When the failing gate command is a linter or a formatter, the repair prompt carries the conventions card too.
- **The record**. The [evidence record](evidence-records.md) keeps the check's conformance notes in `conventions.conformance_warnings`.
- **Limits**. At most six notes for each file. The written rules are cut after 4,000 characters.

> [!TIP]
> **For experienced users:** write in `.codelead/CONVENTIONS.md` the rules that the code cannot show, such as "use an injected clock, not the system time". Keep the rules that the code already shows to the second layer.

## Related content

- [`codelead conventions`](../cli/codelead-conventions.md)
- [`/implement`](../chat-commands/implement.md) and [`/patch`](../chat-commands/patch.md)
- [The patch check](the-patch-check.md)
- [Write your conventions](../how-to/write-conventions.md)
- [Settings](../extension/settings.md) in the extension
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
- [Glossary](glossary.md)
