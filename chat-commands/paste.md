---
title: /paste
description: Read the clipboard into the session, look at it, clear it, or have a model analyze it, for example a test failure that you copied.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /paste

**Applies to:** CodeLead 0.1 and later

## Name

`/paste` - Reads the clipboard into the session, shows it, clears it, or analyzes it.

## Synopsis

```console
/paste
/paste --show
/paste --clear
/paste --analyze
```

## Description

`/paste` reads the text on the clipboard of the machine where CodeLead runs and keeps it in the session. CodeLead detects what kind of text it is:

| Type | What it looks like |
|---|---|
| `patch` | A unified diff. |
| `implementation-summary` | A coding agent's summary of the work it did. |
| `error-log` | An error message or a stack trace. |
| `test-output` | The output of a test run. |
| `plain-text` | Any other text. |
| `unknown` | Text that CodeLead could not read as any of these. |

You use `/paste` when the text that matters is outside the session. For example: a failure from your continuous integration, a log from a server, or another agent's summary. `/paste --analyze` asks the configured model what the text means for this repository. A failed validation suggests it: `Fix the issue manually, or paste the output with /paste --analyze.`

`/paste` never changes a file. The text stays in the session until you clear it or the session ends.

> [!NOTE]
> **New to this?** Copy the text first, in any program. Then type `/paste` in the session. To give a request from the clipboard instead, use `--from-clipboard` with [`/implement`](implement.md), [`/patch`](patch.md) or [`/preflight`](preflight.md).

## Options

- **`--show`**

  Shows the pasted text: its type, its size, its source, and the first 500 characters.

- **`--clear`**

  Removes the pasted text and its analysis from the session.

- **`--analyze`**

  Sends the pasted text to the configured model with the context of the session: the last request, the last prompt and the project. The model's analysis is printed under `Pasted Content Analysis`. Very long text is cut before it is sent, and the analysis says so.

The forms without dashes, such as `/paste show`, are refused with the form to use.

## What you see

| Outcome | Headline |
|---|---|
| `done` | `✓ Pasted 1,234 characters from clipboard.`, then `Detected: test-output` |
| `done` | `Clipboard is empty.` (copy text first, then run `/paste`) |
| `done` | `Last pasted content`, with the preview (`--show`) |
| `done` | `✓ Cleared pasted content.` or `No pasted content to clear.` (`--clear`) |
| `done` | `No pasted content is available.` (`--show` or `--analyze` before `/paste`) |
| `failed` | `Failed to read clipboard.` or `Failed to analyze pasted content: <the error>` |

## Examples

- Read a test failure that you copied, then ask what it means:

  ```console
  /paste
  /paste --analyze
  ```

  <!-- described, not captured -->
  ```output
  ✓ Pasted 2,318 characters from clipboard.
  Detected: test-output
  Paste analysis complete.
  Pasted Content Analysis
  …
  ```

- Show the pasted text when nothing was pasted:

  ```console
  /paste --show
  ```

  ```output
  No pasted content is available.
  Run /paste first.
  ```

- Clear the pasted text, when there is none:

  ```console
  /paste --clear
  ```

  ```output
  No pasted content to clear.
  ```

- The form without dashes:

  ```console
  /paste show
  ```

  ```output
  Unsupported command form: /paste show

  Use:
    /paste --show
  ```

## Files

`/paste` reads the system clipboard and keeps the text in the session. `--analyze` sends the text to the configured model. It writes no file of the project.

## See also

- [`/copy`](copy.md)
- [`/validate`](validate.md)
- [`/repair`](repair.md)
- [Sessions](../concepts/sessions.md)
