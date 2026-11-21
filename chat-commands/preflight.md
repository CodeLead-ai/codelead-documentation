---
title: /preflight
description: Turn a coding request into a precise, implementation-ready prompt, for CodeLead's own patch step or for another coding agent.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /preflight

**Applies to:** CodeLead 0.1 and later

## Name

`/preflight` - Analyzes a request with a model and writes an implementation-ready prompt for a coding agent.

## Synopsis

```console
/preflight <REQUEST>
/preflight --from-clipboard
```

## Description

`/preflight` is the first step of [`/implement`](implement.md), as a command of its own. CodeLead finds the files that the request is about, the same way as [`/scan`](scan.md). Then a model reads the request and those files. It writes an analysis and a final prompt: the request, made precise enough that a coding agent can do it without guessing.

You use `/preflight` when you want to read the plan before any patch exists. You also use it when another coding agent will do the work: [`/copy`](copy.md) copies the prompt to the clipboard. To let CodeLead write the patch from it, run [`/patch --from-preflight`](patch.md).

`/preflight` never changes a file.

> [!NOTE]
> **New to this?** A preflight asks what a careful colleague asks first. Which files? Which behaviour must stay? How will we test it? When something is not clear, the preflight asks you, and each question shows a default that you can take with Enter.

> [!TIP]
> **For experienced users:** the depth of the analysis follows [`/strategy`](strategy.md). The `auto` strategy picks one from the words of the request: words such as "security" or "migration" choose `deep`. The `fast` strategy asks no questions.

## Arguments

- **`<REQUEST>`**

  The change, in plain words.

## Options

- **`--from-clipboard`**

  Takes the request from the clipboard text.

## What CodeLead asks

The clarifying questions of the preflight, when the strategy asks questions. Each question ends with `Answer, press Enter to use default:`. CodeLead recommends the default that the question shows, and Enter gives it.

## What you see

| Outcome | Headline | The way on |
|---|---|---|
| `done` | `Preflight complete: the implementation prompt is ready` | `/patch --from-preflight`, `/copy --prompt` |
| `failed` | `The preflight did not complete: failed to run preflight: <the error>` | The error's own ways out, such as the provider's hint. Run it again after you fix the cause. |
| `stopped` | `No preflight ran: /preflight needs a request`, after `Usage: /preflight <request>` | `/preflight <request>`, `/preflight --from-clipboard` |

## Examples

- Analyze a request and keep its prompt in the session:

  ```console
  /preflight add a clearCompleted function to src/tasks.js that removes the done tasks
  ```

  <!-- described, not captured -->
  ```output
  Preflight
  Generating an implementation-ready coding-agent prompt.
  …
  Preflight complete.
  ```

  The result's way on is `/patch --from-preflight` or `/copy --prompt`.

- The model server does not run:

  ```output
  Preflight
  Generating an implementation-ready coding-agent prompt.
  Failed to run preflight
  Could not connect to lmstudio at http://localhost:1234/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ```

  The result is `The preflight did not complete: failed to run preflight: Could not connect to lmstudio at http://localhost:1234/v1`. Its way on is the provider's hint: `If this is LM Studio, make sure it is open and the Local Server is running.` Check the server with [`codelead doctor`](../cli/codelead-doctor.md).

- Give the request from the clipboard, for example a bug report that you copied:

  ```console
  /preflight --from-clipboard
  ```

## Files

- Reads the files of the repository that the request is about.
- Writes the model's prompt and answer to the session folder (`.codeleadsessions/<session>/`).
- Keeps the final prompt in the session for `/copy` and `/patch --from-preflight`. It writes no file of the project.

## See also

- [`/implement`](implement.md)
- [`/patch`](patch.md)
- [`/copy`](copy.md)
- [`/strategy`](strategy.md)
- [Requests, plans and patches](../concepts/requests-plans-and-patches.md)
- [Strategies](../concepts/strategies.md)
- [`codelead preflight`](../cli/codelead-preflight.md)
