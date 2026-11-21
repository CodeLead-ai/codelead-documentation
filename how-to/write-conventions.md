---
title: Write your conventions
description: Write the rules of your team in .codelead/CONVENTIONS.md, see them come first in the conventions card that every change carries, and share them with other coding agents.
kind: how-to
area: how-to
date: 2026-11-20
---

# Write your conventions

This guide writes your team's rules into `.codelead/CONVENTIONS.md`. CodeLead gives that file to the model word for word with every change, before the habits that it reads from the code. Write there what the code cannot show, such as "money is a decimal, never a double".

**In short**
- [`codelead conventions`](../cli/codelead-conventions.md) shows the habits that CodeLead already reads from your code.
- You write the rules that the code cannot show in `.codelead/CONVENTIONS.md`, one rule per line.
- Your rules come first in the conventions card, and they outrank the habits.
- `codelead conventions --agents` writes the same rules into `AGENTS.md` and `CLAUDE.md` for other coding agents.

## Before you begin

- You ran [`codelead init`](../cli/codelead-init.md) in the repository.
- You know the two layers of [conventions](../concepts/conventions.md): the rules that you write, and the habits that CodeLead reads from the code.

> [!NOTE]
> **New to this?** The *conventions card* is the text that CodeLead adds to the prompt of every patch. It holds your rules, the habits of the code, and examples of files like the one that the change adds.

## See what the code already shows

Run the command in the repository. It calls no model. On the Basic Store sample:

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
16 rules from 31 sampled files. Written to <your project>/.codelead/conventions.json.
```

Do not write these rules again. The code shows them, and [the patch check](../concepts/the-patch-check.md) already compares new lines with them.

## Write the rules of your team

1. Create the file `.codelead/CONVENTIONS.md`.
2. Write one rule per line, in plain words. Write the rules that a reader cannot see in the code:

   ```markdown
   # Conventions

   - Money is a decimal, never a double.
   - Error messages that a person reads are full sentences that end with a period.
   ```

3. Commit the file, so that the whole team and every machine use the same rules.

In the [extension](../extension/settings.md), open the Settings page at **Conventions**. **Create** writes a short starter file and opens it. Its example rules are in a comment: move a line out of the comment to make it a rule. **Edit** opens the file when it exists.

> [!TIP]
> **For experienced users:** a repository that already has `CONVENTIONS.md`, `STYLEGUIDE.md` or `CONTRIBUTING.md` needs no new file. CodeLead reads the first file of its list that holds text. `.codelead/CONVENTIONS.md` comes first in that list. See [Conventions](../concepts/conventions.md#layer-one-the-rules-you-write).

## Read the card again

Run `codelead conventions` again. Your rules now come first, before the habits:

```output
PROJECT CONVENTIONS

Extracted from 31 sampled source files (C#, JavaScript); .codelead/conventions.json holds the evidence per rule.

Team rules from .codelead/CONVENTIONS.md (these outrank everything extracted below):
  # Conventions
  
  - Money is a decimal, never a double.
  - Error messages that a person reads are full sentences that end with a period.

Formatting:
- C# files are indented with 4 spaces (e.g. WebApplication1/App_Start/BundleConfig.cs)
…
16 rules from 31 sampled files; team rules from .codelead/CONVENTIONS.md. <your project>/.codelead/conventions.json unchanged.
```

The last line names your file. That is the sign that CodeLead found it.

## Share the rules with other coding agents

Other agents in the same repository can follow the same rules. This command writes CodeLead's block into `AGENTS.md` and `CLAUDE.md`, and keeps the rest of each file:

```console
codelead conventions --agents
```

```output
AGENTS.md: CodeLead block written
CLAUDE.md: CodeLead block written
…
```

The block holds the validation gate, the project's documents, your rules and the habits. It sits between the markers `<!-- codelead:conventions:start -->` and `<!-- codelead:conventions:end -->`. Run the command again after you change a rule: CodeLead replaces its block, and nothing else.

## Check that it worked

- `codelead conventions` shows `Team rules from .codelead/CONVENTIONS.md`, and its last line says `team rules from .codelead/CONVENTIONS.md`.
- `.codelead/conventions.json` holds your file's name and text under `overrides`.
- To see the examples that a new file would get, add `--for <path>`, for example `codelead conventions --for WebApplication1/Services/Clock.cs`. The card then lists `Exemplars` under the rules.

## Details

- **What the check tests.** The patch check tests the added lines only against the habits that are not mixed. Your written rules reach the model as words, and no check tests them. Put a rule that must block in your linter, and the linter in the [gate](declare-validation-commands.md).
- **The limit.** CodeLead uses the first 4,000 characters of your file. Keep the file short.
- **When it is read.** Each patch prompt reads the code and your file again. `codelead conventions` stores the profile, and says `unchanged` when nothing moved.
- **Turning the card off.** `CODELEAD_CONVENTIONS_CARD=0` leaves the card out of the patch prompt. Use it only to compare runs.
- **The gate in AGENTS.md.** The block names the commands that the gate really runs. These are the commands of `.codelead/validation.json` when you declare them. Otherwise, they are the commands that the gate resolves for the project, for example `MSBuild sampleapp.slnx -restore -nologo -v:minimal` for the Basic Store sample. A `.codelead/validation.json` that CodeLead cannot read gives one `Warning:` line, and the block names the detected commands. See [Declare validation commands](declare-validation-commands.md).

## See also

- [Conventions](../concepts/conventions.md)
- [`codelead conventions`](../cli/codelead-conventions.md)
- [The patch check](../concepts/the-patch-check.md)
- [Settings](../extension/settings.md) in the extension
- [Declare validation commands](declare-validation-commands.md)
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
