---
title: Requests, plans and patches
description: What a request is, how the preflight turns it into a plan, and what a patch is before anything changes in your files.
kind: concept
area: concepts
date: 2026-11-20
---

# Requests, plans and patches

A **request** is what you want changed, in plain words. The **preflight** turns the request into a **plan**: the files to change, what must stay as it is, and how to check the change. A model then writes a **patch**: the changes to the files, as a diff. Nothing in your files changes until you approve the patch.

**In short**
- A good request names what must change, where, and how you will know that it works.
- The preflight finds the files that the request is about, with no model, and then a model analyzes them. It can ask you questions or list its assumptions.
- The plan is the scope of the change. The [patch check](the-patch-check.md) compares the patch with it.
- A patch is a proposal. [`/edit`](../chat-commands/edit.md) applies it only after your yes.

## Why it exists

> [!NOTE]
> **New to this?** A *diff* lists the lines to add and the lines to remove in each file. `git diff` prints one. A patch is a diff that nobody has applied yet.

A model that starts to write code at once often guesses. It guesses the file, the behaviour you want, or the way to test it. A wrong guess costs a patch, a failed build and a repair.

The preflight asks before it guesses. It is like a careful colleague who reads the code first and asks two questions before starting. The plan that comes out of it also limits the change. A patch that edits a file outside the plan, or crosses a non-goal, gets a note. A patch that leaves out a file that the plan requires can be refused.

## How it works

### The request

Write the request as you would write it for a colleague. For example:

```text
In WebApplication1/Controllers/OrdersController.cs, in ValidateOrderAsync, refuse an order
that has the same product on two lines. Show the error "Each product can appear on one line
only." on the second line's product field.
```

[`/lint`](../chat-commands/lint.md) and [`codelead lint`](../cli/codelead-lint.md) read a request before a run, with no model. They say which files it names and whether those files exist. On the Basic Store sample, the request above gives:

```output
Request lint (the request, the repository, the System Model and the journey spec; no model call)

Files the request names:
- WebApplication1/Controllers/OrdersController.cs: exists

Test files it names, and those beside the files it names:
- none

Libraries it assumes:
- none named
…
```

### The preflight

The preflight has two parts:

1. **CodeLead chooses the files.** It indexes the repository into the [project library](project-library.md) and scores each file against the words of the request. This part uses no model. [`codelead scan`](../cli/codelead-scan.md) shows the same choice.
2. **A model analyzes.** It reads the request and the chosen files, and writes the plan. The [strategy](strategies.md) decides how deep the analysis goes and how many questions it may ask.

Here is the start of the file choice for a request about stock and orders on the Basic Store sample:

```output
Repository Context
- Project type: existing
- Languages: C#, JavaScript
- Tools: .NET
- Validation: dotnet build
- Files scanned: 89
- Relevant files shown: 12
…
Likely Relevant Files

1. WebApplication1/Controllers/OrdersController.cs
   Role: source (C#)
   Score: 1000
   Original score: 14
   Reason: lexical match; symbol/token match for order, controller, product, quantity
…
```

When something is not clear, the preflight asks a clarifying question. Each question says why it matters and shows a default. Press Enter to take the default, which is CodeLead's recommended answer:

<!-- described, not captured -->
```output
Clarification 1/2

Should the error also appear when the order is edited, or only when it is created?

Why it matters:
ValidateOrderAsync runs for both forms.

Default:
Both: the rule applies to every order.

Answer, press Enter to use default:
```

When the preflight is sure enough, it lists its assumptions in place of questions. You accept them all with Enter, reject them all with `no`, or correct one by its number. See [Prompts and recommended answers](prompts-and-recommended-answers.md).

### The plan

The preflight's result is the plan. Depending on the strategy, it holds:

- the files to change, and the files to read only;
- the implementation, step by step;
- the **non-goals**: what the change must not touch;
- the risks;
- how to validate the change.

[`/preflight`](../chat-commands/preflight.md) shows the plan as a prompt that you can read, or copy for another coding agent with [`/copy`](../chat-commands/copy.md).

### The patch

A model writes the patch from the plan. It is a unified diff, like the output of `git diff`. [`/patch --show`](../chat-commands/patch.md) shows it:

```output
Patch Summary
Request: add a clearCompleted function that removes the done tasks, with a test
Parse status: valid unified diff
Files affected: 2
Added lines: 10
Removed lines: 1
…
```

Then the patch check reads the patch against your files, and you decide whether to apply it.

## What it is not

- **A request is not the whole prompt**. CodeLead adds the plan, the files, the [conventions](conventions.md) and its own rules around it.
- **The plan is not a promise.** It is the scope that the patch check holds the patch to. The [gate](validation-and-the-gate.md) decides whether the change works.
- **A patch is not a change.** It is a proposal. Only [`/edit`](../chat-commands/edit.md), or the apply step of [`/implement`](../chat-commands/implement.md), changes files, and only after your yes.

## Details

- **Options count only before the request.** In `/implement make the deploy script stop using git push --force`, the words `--force` belong to the request.
- **A bug whose cause is not found**. The request asks to fix a bug, and the preflight does not find its cause. Then `/implement` ends after the preflight with no patch. The evidence record's outcome is `cause_not_found`. Name the cause in the request, or add `--force`.
- **A composed request.** The pins and the transforms of the modernization program write their own requests. Their preflight asks nothing: it takes each default and records it. The chat shows these once as "CodeLead assumed". See [Pins](../modernization/pins.md).
- **Patch retries.** When the check refuses a patch, `/implement` asks for a new one, at most 2 times. The retry gets the check's problems and asks for smaller hunks.
- **Large files.** For a large file, the model can return an anchored block (`CODELEAD_INSERT` or `CODELEAD_REPLACE_MEMBER`) in place of a diff. CodeLead turns it into a local hunk of a diff.
- **A reply cut at its cap.** When the model's reply stops at its token cap, CodeLead never applies the part that arrived. It says so and can try once more. See [Results and stops](results-and-stops.md).
- **A new application** has its own plan: a project specification and an execution plan of steps ([`/create-app`](../chat-commands/create-app.md)), or a plan of increments ([`/build-incremental`](../chat-commands/build-incremental.md)).

## Related content

- [`/implement`](../chat-commands/implement.md), [`/preflight`](../chat-commands/preflight.md), [`/patch`](../chat-commands/patch.md), [`/edit`](../chat-commands/edit.md)
- [`/lint`](../chat-commands/lint.md), [`/scan`](../chat-commands/scan.md), [`/strategy`](../chat-commands/strategy.md), [`/copy`](../chat-commands/copy.md)
- [`codelead lint`](../cli/codelead-lint.md), [`codelead scan`](../cli/codelead-scan.md), [`codelead preflight`](../cli/codelead-preflight.md)
- [How CodeLead works](how-codelead-works.md)
- [The patch check](the-patch-check.md)
- [Strategies](strategies.md)
- [Prompts and recommended answers](prompts-and-recommended-answers.md)
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
