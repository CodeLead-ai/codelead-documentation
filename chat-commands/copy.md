---
title: /copy
description: Copy the last coding-agent prompt or the last patch of the session to the clipboard.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /copy

**Applies to:** CodeLead 0.1 and later

## Name

`/copy` - Copies the last coding-agent prompt, or the last patch, to the clipboard.

## Synopsis

```console
/copy
/copy --prompt
/copy --patch
```

## Description

`/copy` puts text from the session on the clipboard of the machine where CodeLead runs. You then paste it where you need it.

- The prompt is the final coding-agent prompt that the last [`/preflight`](preflight.md) wrote. Paste it into another coding agent when that agent will do the change.
- The patch is the unified diff of the last [`/patch`](patch.md). Paste it into a review, or apply it with `git apply`.

`/copy` never changes a file of the project. It replaces what the clipboard held before.

## Options

- **`--prompt`**

  Copies the last final coding-agent prompt. This is the default: `/copy` alone does the same.

- **`--patch`**

  Copies the last patch proposal, as a unified diff.

The forms without dashes, such as `/copy prompt`, are refused with the form to use. Words after an option are refused: `Unexpected /copy request: <words>`.

## What you see

| Outcome | Headline |
|---|---|
| `done` | `✓ Copied last final coding-agent prompt to clipboard.` |
| `done` | `✓ Copied last generated patch to clipboard.` |
| `done` | `No final coding-agent prompt is available yet.` (run `/preflight <request>` first) |
| `done` | `No patch is available yet.` (run `/patch <request>` first) |
| `done` | `No valid patch is available to copy.` (the last proposal held no valid diff) |
| `failed` | `Failed to copy prompt to clipboard`, when the clipboard cannot be written |

## Examples

- Copy the prompt after a preflight, to give it to another coding agent:

  ```console
  /copy
  ```

  <!-- described, not captured -->
  ```output
  ✓ Copied last final coding-agent prompt to clipboard.
  ```

- Copy the patch, when there is none yet:

  ```console
  /copy --patch
  ```

  ```output
  No patch is available yet.
  Run /patch <request> first.
  ```

- Copy the prompt, when no preflight ran:

  ```console
  /copy --prompt
  ```

  ```output
  No final coding-agent prompt is available yet.
  Run /preflight <request> first.
  ```

- The form without dashes:

  ```console
  /copy prompt
  ```

  ```output
  Unsupported command form: /copy prompt

  Use:
    /copy --prompt
  ```

## Files

`/copy` writes the system clipboard only. It writes no file.

## See also

- [`/paste`](paste.md)
- [`/preflight`](preflight.md)
- [`/patch`](patch.md)
- [Requests, plans and patches](../concepts/requests-plans-and-patches.md)
