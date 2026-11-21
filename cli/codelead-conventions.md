---
title: codelead conventions
description: Extract the repository's conventions with their evidence, print the card that /implement carries, and write instructions for other coding agents.
kind: reference
area: cli
date: 2026-11-20
---

# codelead conventions

**Applies to:** CodeLead 0.1 and later

`codelead conventions` reads the repository and extracts how its code is written: the formatting, the naming, where files go, the tests and the idioms. Each rule carries its evidence. It calls no model.

## Name

`codelead conventions` - Extracts the repository's conventions with their evidence, and prints the card that /implement uses.

## Synopsis

```console
codelead conventions [--for <PATH>]... [--json] [--agents] [--no-write]
codelead conventions -h|--help
```

## Description

New code should look like the code around it. CodeLead reads a sample of the source files and states the [conventions](../concepts/conventions.md) that it finds. For example: "C# files are indented with 4 spaces", or "Controllers live in `WebApplication1/Controllers/`". Each rule has an example file. When not every file follows a rule, the rule shows a share, such as `[97% of 309]`.

The result is the **convention profile**. CodeLead writes it to `.codelead/conventions.json`. It also prints the **conventions card**: a short text that every [`/implement`](../chat-commands/implement.md) carries in its [context package](../concepts/how-codelead-works.md). The model then follows the rules.

> [!NOTE]
> **New to this?** A *convention* is a habit of a codebase that no compiler checks: names, layout, style. When a rule is mixed, the card says "follow the file you edit".

### Team rules

The team's own rules come first. When the repository has `.codelead/CONVENTIONS.md`, `CONVENTIONS.md`, `CONTRIBUTING.md` or `STYLEGUIDE.md`, CodeLead reads it, and the summary line names it. See [Write your conventions](../how-to/write-conventions.md).

### Exemplars

`--for <path>` names a file that the next change would add. CodeLead then chooses existing files that look like it (the **exemplars**) and prints them under the card. For example, for a new controller it prints an existing controller.

### Instructions for other coding agents

`--agents` writes CodeLead's block into `AGENTS.md` and `CLAUDE.md` at the repository's root. The block holds the conventions, the validation gate (the commands that must pass) and the project documents. Other coding agents read these files. Only the text between CodeLead's markers is replaced; the rest of each file stays yours.

> [!TIP]
> **For experienced users:** the block's validation gate comes from `.codelead/validation.json` when that file declares commands. Otherwise the block names the commands that the gate resolves for the project, as [`/validate`](../chat-commands/validate.md) runs them. See [Declare validation commands](../how-to/declare-validation-commands.md).

## Options

- **`--for <PATH>`**

  A file that the next change would add. Shows the exemplars chosen for it. Give it again for more files.

- **`--json`**

  Prints the profile as JSON, with the evidence of each rule. The card is not printed.

- **`--agents`**

  Writes the conventions, the validation gate and the project documents into `AGENTS.md` and `CLAUDE.md`, for other coding agents. Only CodeLead's block is written; your text is kept.

- **`--no-write`**

  Does not write `.codelead/conventions.json`.

## Examples

- Extract the conventions of the Basic Store sample:

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

  Placement:
  - Controllers live in `WebApplication1/Controllers/` (6 of 6) (e.g. WebApplication1/Controllers/AccountController.cs)
  - ViewModels live in `WebApplication1/Models/` (4 of 4) (e.g. WebApplication1/Models/AccountViewModels.cs)
  …
  Instruction: new code follows these conventions and the exemplars. Where the file you edit differs from a rule, follow the file you edit.

  16 rules from 31 sampled files. Written to <your project>/.codelead/conventions.json.
  ```

- See the exemplars for a new controller, and write nothing:

  ```console
  codelead conventions --for WebApplication1/Controllers/CategoriesController.cs --no-write
  ```

  ```output
  …
  Exemplars (existing files like the ones this change adds — imitate their shape):

  File: WebApplication1/Controllers/AccountController.cs (a Controller like the CategoriesController.cs this change adds)
  using System;
  using System.Globalization;
  …
  namespace WebApplication1.Controllers
  {
      [Authorize]
      public class AccountController : Controller
  …
  16 rules from 31 sampled files. Not written (--no-write).
  ```

  In the real output, the exemplar's code sits in its own code fence.

- Write the instructions for other coding agents:

  ```console
  codelead conventions --agents
  ```

  ```output
  AGENTS.md: CodeLead block written
  CLAUDE.md: CodeLead block written
  PROJECT CONVENTIONS
  …
  ```

  The block in `AGENTS.md` starts like this:

  ```output
  <!-- codelead:conventions:start -->
  ## Working in this repository (generated by CodeLead, 2026-11-20)

  Regenerate with `codelead conventions --agents`; everything between the CodeLead markers is replaced, the rest of this file is yours. The evidence for every rule below is in `.codelead/conventions.json`.

  ### Validation gate

  A change is done when these pass; run them before you hand anything over:

  - `MSBuild sampleapp.slnx -restore -nologo -v:minimal`
  …
  ```

  The list names the commands that [the gate](../concepts/validation-and-the-gate.md) really runs. The Basic Store sample is a .NET Framework solution, so the gate builds it with Visual Studio's MSBuild, not with `dotnet build`. A test project adds `dotnet test <project> --no-build -nologo`.

- A `.codelead/validation.json` that CodeLead cannot read gives one warning, and the agents' files name the detected commands:

  <!-- described, not captured -->
  ```output
  Warning: .codelead/validation.json was not read (it is not valid JSON: …), so the agents' files name the detected commands; correct the file, or remove it, and run codelead conventions --agents again.
  AGENTS.md: CodeLead block written
  …
  ```

  Run it again with no change in the code, and both files stay as they are:

  ```output
  AGENTS.md: unchanged
  CLAUDE.md: unchanged
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The conventions were extracted. A repository with no source files is also `0`: the card says that nothing was extracted. |

## Files

| File | What `conventions` does with it |
|---|---|
| The repository's source files | Reads a sample. |
| `.codelead/CONVENTIONS.md`, `CONVENTIONS.md`, `CONTRIBUTING.md`, `STYLEGUIDE.md` | Reads the team's rules, when one is there. |
| `.codelead/conventions.json` | Writes the profile with the evidence of each rule. Not with `--no-write`. |
| `AGENTS.md`, `CLAUDE.md` | With `--agents`, writes CodeLead's block between its markers. |
| `.codelead/validation.json` | With `--agents`, reads the declared validation commands. |

## See also

- [Conventions](../concepts/conventions.md)
- [Write your conventions](../how-to/write-conventions.md)
- [Settings in Visual Studio Code](../extension/settings.md): the Conventions section.
- [`/implement`](../chat-commands/implement.md): carries the conventions card.
- [Declare validation commands](../how-to/declare-validation-commands.md)
