---
title: The patch check
description: What CodeLead checks in a patch before it asks you to apply it, the difference between a blocking finding and a note, and what each status means.
kind: concept
area: concepts
date: 2026-11-23
---

# The patch check

The **patch check** is CodeLead's own reading of a patch against your files, before anything changes. It uses no model, so the same patch on the same files always gets the same verdict. It answers three questions. Does every change fit the files as they are now? Is every path safe to change? Does the patch stay inside the plan?

**In short**
- The patch check runs after a model writes a [patch](requests-plans-and-patches.md) and before CodeLead asks you to apply it.
- A **blocking finding** stops the patch: CodeLead recommends no, or asks the model for a new patch. A **note** never blocks: you read it and decide.
- The status is one of `can_apply`, `cannot_apply`, `unsafe` or `invalid_patch`.
- The patch check does not build or test the code. The [gate](validation-and-the-gate.md) does that after the apply.

## Why it exists

A model's output is untrusted text. It can be written against an older version of a file. It can name a path outside the repository, or a key file such as `.env`. It can be cut off in the middle. Applying such a patch can damage your working tree.

The patch check catches these problems before the apply, without a model. A refused patch costs nothing: your files are as they were. The model gets the check's words and tries again.

> [!NOTE]
> **New to this?** A *hunk* is one block of changed lines in a diff. A few unchanged lines around it give the context. The check uses those lines to find where the change goes.

## How it works

### Where it runs

[`/implement`](../chat-commands/implement.md) runs the patch check by itself after each patch. [`/patch --check`](../chat-commands/patch.md) runs it on the patch that waits in the session. Every build step of a new app and every repair round runs it too.

### What it checks

| Group | What it checks | When it fails |
|---|---|---|
| Form | The patch is a valid unified diff, whole, and it changes something. | Blocking: `invalid_patch`, or "The patch changes nothing". |
| Fit | Each hunk matches the file as it is now. CodeLead searches near the line, then in the whole file. | Blocking: `cannot_apply`. A hunk whose edge differs slightly is placed, with a note. |
| Safety | Every path is inside the repository. No absolute path, no `..`, no generated, dependency, build, cache or session file, no sensitive file. | Blocking: `unsafe`. |
| Sanity | `package.json` and `tsconfig.json` stay valid. Local imports point to files that exist. TypeScript and C# keep their basic syntax rules. No top-level name is declared twice. | Blocking: `cannot_apply`. |
| Tests | The patch does not remove tests that the request did not ask to remove. | Blocking, or a note when tests are only renamed. |
| Scope | The patch includes the files that the plan requires, and the files that must change together. | Blocking when a required file is missing, with two exceptions in the details below. A file outside the plan, or a non-goal crossed, is a note. |
| Habits | New lines follow the repository's [conventions](conventions.md). The layout, stub markers such as `TODO`, controls that do nothing, files with uncommitted changes. | Notes only. |

### A patch that passes, with a note

Here is the real check of a small patch on the Basic Store sample. The patch adds a rule to `OrdersController.cs`. One new line has an `if` without braces, and the repository's conventions say that control statements always take braces:

```output
Patch check
Status: can apply cleanly
Files:
  ✓ WebApplication1/Controllers/OrdersController.cs
Summary:
  Files changed: 1
  Hunks checked: 2
  Additions: 4
  Removals: 0
Warnings:
  - [conventions] WebApplication1/Controllers/OrdersController.cs: a control statement without braces: `if (!seenProducts.Add(line.ProductId.Value)) ModelState.AddM` — the code base's rule: Control statements always take braces, on their own line (e.g. WebApplication1/App_Start/IdentityConfig.cs)

Safety:
  ✓ All paths are inside the repository.
  ✓ No generated/dependency/build/cache/session artifact files are targeted.
  ✓ No sensitive files are targeted.
  ✓ Patch applies cleanly in check mode.
```

Before the apply, CodeLead shows the same finding as a note, and it recommends yes:

```output
Apply checked patch

Files: 1
Hunks: 2
Additions: 4
Removals: 0

Notes (they do not block applying):
- [conventions] WebApplication1/Controllers/OrdersController.cs: a control statement without braces: `if (!seenProducts.Add(line.ProductId.Value)) ModelState.AddM` — the code base's rule: Control statements always take braces, on their own line (e.g. WebApplication1/App_Start/IdentityConfig.cs)

This will modify files in your working tree.
The patch check passed with 1 note. CodeLead recommends applying.
```

The question that follows is `Apply this patch? [Y/n]`. Enter gives yes. You can also say no, ask for a new patch with [`/patch --retry`](../chat-commands/patch.md), or fix the line yourself after the apply.

### A patch that is refused

Here the patch was written against text that is not in the file. The check refuses it, and nothing changes:

```output
Patch check
Status: cannot apply cleanly
Files:
  ✗ WebApplication1/Controllers/OrdersController.cs
…
Problems:
  - WebApplication1/Controllers/OrdersController.cs: Hunk 1 does not match current file contents. Tried line 256 and searched nearby/whole file. Expected context line was not found: "                ? new List<int>()". …

Safety:
  ✓ All paths are inside the repository.
  ✓ No generated/dependency/build/cache/session artifact files are targeted.
  ✓ No sensitive files are targeted.
  ✗ Patch does not apply cleanly in check mode.

Next:
  Regenerate the patch, or adjust the request with more context.
  Review the patch with /patch --show.
```

A patch that targets a key file is `unsafe`:

```output
Patch check
Status: unsafe
Files:
  ✗ .env
…
Problems:
  - .env: Unsafe path ".env": sensitive files are not allowed.

Safety:
  ✗ One or more patch paths failed safety checks.
  ✓ Patch check did not apply or modify files.

Next:
  Do not apply this patch.
  Regenerate the patch or adjust the request to target safe repository files.
```

## What it is not

- **Not a build or a test.** The check reads text. It does not compile the code or run it. A patch that passes the check can still fail the gate.
- **Not a model's opinion.** No model takes part in the verdict. The same patch on the same files gets the same result.
- **Not a style review that blocks.** Convention and layout findings are notes. CodeLead's rule is that a heuristic check warns and never refuses work that can be applied.

> [!NOTE]
> **For team leads:** every evidence record keeps the check's status and its blocking findings. It also keeps the count of notes, and the convention notes word for word. See [Evidence records](evidence-records.md).

## Details

- **Statuses.** `can_apply` (shown as "can apply cleanly"), `cannot_apply`, `unsafe`, `invalid_patch`, and `no_patch` when nothing waits to be checked.
- **Retries.** After a refusal, `/implement` asks for a new patch at most 2 times. The retry carries the problems and asks for smaller hunks. [`/patch --retry`](../chat-commands/patch.md) does the same by hand.
- **Files the request does not name.** A missing planned file is a note when the request names other files and not this one. A name that several files share names none of them. In a web project where every screen is a `page.tsx` in its own folder, "the page web/src/app/items/list/page.tsx" names that page only. Another screen's `page.tsx` that the plan added stays a note when the patch leaves it out. Only a path that tells the files apart names one of them.
- **The file a change goes through.** Some requests name an existing file only as the way the change goes. For example: "through the API client module (src/lib/api.ts)". The patch may change that file, but it does not have to. The note then says: "the request names it as what the change goes through, not as a file to change". A file that does not exist yet stays required. So does a file that the request also asks to change, as in "and add the same data to the mock".
- **The change surface.** CodeLead derives the files that a request is likely to change. A file outside them gets a note that ends with the words "a note, it does not block applying". In a .NET repository, some files must change together, for example a class and the base class it relies on. A patch that changes only one side gets a blocking finding.
- **Re-anchoring.** A hunk whose context differs at its first or last line is placed where the rest matches, with a note that says where.
- **A cut reply.** A patch whose reply was cut at the model's cap is refused whole, so that what arrived is never applied.
- **The pins' database.** A database pin project that reads the application's own configuration or connection is refused. This is one of the few checks that protect data. See [Pins](../modernization/pins.md).
- **A rebuild's accepted parts.** In a rebuild, the check also refuses a patch that breaks what an accepted part built. Such a patch removes an accepted endpoint, a screen's route or a test. Or it changes what such a test sends or asserts. Its words start with `accepted_parts:` and say the way out. A part that landed and is not accepted yet is a warning only. See [The rebuild](../modernization/rebuild.md#what-an-accepted-part-built-stays-built).
- **Templates.** In a folder built from a template (`.codelead/template.json` exists), the design-token rule for new stylesheets stands down.
- **Where the words go.** Into the session log, into the next patch prompt after a refusal, and into the evidence record's `patch_check` field.

## Related content

- [`/patch`](../chat-commands/patch.md) (`--check`, `--retry`, `--show`) and [`/edit`](../chat-commands/edit.md)
- [`/implement`](../chat-commands/implement.md)
- [`codelead conventions`](../cli/codelead-conventions.md)
- [Requests, plans and patches](requests-plans-and-patches.md)
- [Validation and the gate](validation-and-the-gate.md)
- [Conventions](conventions.md)
- [Prompts and recommended answers](prompts-and-recommended-answers.md)
- [Messages and what to do](../reference/messages.md)
