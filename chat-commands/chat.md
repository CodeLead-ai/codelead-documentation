---
title: /chat
description: Ask about this repository, about what CodeLead did to it, or about CodeLead itself, in plain words; the chat never changes code.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /chat

**Applies to:** CodeLead 0.1 and later

## Name

`/chat` - Answers a question about this repository, about what CodeLead did to it, or about how to use CodeLead.

## Synopsis

```console
<QUESTION>
/chat <QUESTION>
/chat --new
/chat --new <QUESTION>
```

## Description

The chat answers questions in plain words. It reads the repository's code and the help of these commands. It also reads CodeLead's records about the code: the evidence records, the run report, the parked parts of a rebuild and the board. Its answers name the files and lines that they rest on. See [The chat](../extension/chat.md).

Text without a `/` goes to the chat, so `/chat` is only the explicit form. [`/ask`](ask.md) is the same command, and `/ask --record` also writes the answer to a file.

The chat never changes code: it has no tool and no patch. When your words ask for a change, it answers with the command to run, for example `/implement add a due date to tasks`, and asks `Run it now?`. That command asks again before it changes a file.

The chat keeps the conversation of this session: each question carries the last 12 exchanges. `/chat --new` forgets them. The conversation ends with the session.

> [!NOTE]
> **New to this?** Ask the way you would ask a colleague who knows the code. For example: "how is the order total calculated?", or "why did the last pin stop?".

> [!TIP]
> **For experienced users:** the chat uses the `deep` role, or the intervention model of a rebuild's run profile. It shows the cost of each answer. A remote model needs your consent, as every model call does. See [Privacy and remote models](../concepts/privacy-and-remote-models.md).

## Arguments

- **`<QUESTION>`**

  Your question, in plain words. Every word after `/chat` belongs to the question, even a word that starts with `--`, except a first `--new`.

## Options

- **`--new`**

  Starts the conversation over. With a question after it, the chat answers that question in the new conversation.

## What CodeLead asks

| Question | CodeLead recommends | Enter gives |
|---|---|---|
| `Run it now? [Y/n]`, after the chat offers a command | Yes: "the command asks again before it changes a file". | Yes. |

## What you see

A question to the chat is informational: the answer is the output, so the result has no headline. The answer ends with the model and its cost, for example `lmstudio the-deep-model · 1,500 in, 120 out tokens, $0.0123`.

| Situation | Result |
|---|---|
| The chat answered | No headline: the answer is above. |
| The model could not answer | `failed`: `The chat could not answer: <the error>`. The way on is the error's own hint, such as `If this is LM Studio, make sure it is open and the Local Server is running.` |
| The model cannot be used here | `The chat's model cannot be used; nothing was sent.`, with the reason, such as `Remote model blocked by privacy settings.` |
| `/chat --new` | `The conversation starts over (2 exchanges forgotten).` |

## Examples

- Ask about the code, with text that has no slash:

  ```console
  how is the order total computed?
  ```

  <!-- described, not captured -->
  ```output
  The total is a sum of the lines (src/order.ts:1-3).
  lmstudio the-deep-model · 1,500 in, 120 out tokens, $0.0123
  ```

- Ask for a change. The chat offers the command, and runs it only on your yes:

  ```console
  add a due date to tasks
  ```

  <!-- described, not captured -->
  ```output
  That touches src/tasks.ts:1, the Task type.
  …
  /implement add a due date to tasks
  Run it now? [Y/n]
  ```

- Start the conversation over:

  ```console
  /chat --new
  ```

  ```output
  The conversation starts over.
  ```

- The model server does not run:

  ```console
  how does openCount work?
  ```

  ```output
  The chat could not answer.
  Could not connect to lmstudio at http://localhost:1234/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ```

- `/chat` without a question is refused:

  ```console
  /chat
  ```

  ```output
  /chat needs a question, e.g. /chat how does the order total get computed?

  Usage:
    <question>  (text without a leading /)
    /chat <question about this repository, what CodeLead did to it, or how to use CodeLead>
    /chat --new
    /chat --new <question>
  ```

## Files

- Reads the repository's files and CodeLead's records about it. The audit pack's masking hides secrets before anything is sent.
- Writes each exchange to the session log as `chat.request` and `chat.response`. It writes no file of the project.

## See also

- [`/ask`](ask.md)
- [`/implement`](implement.md)
- [`/help`](help.md)
- [The chat](../extension/chat.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
- [`codelead chat`](../cli/codelead-chat.md)
