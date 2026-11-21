---
title: /ask
description: Ask how the code works; with --record, CodeLead writes the answer to a file in the repository and keeps it in the knowledge base.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /ask

**Applies to:** CodeLead 0.1 and later

## Name

`/ask` - Asks the chat a question; `/ask --record` writes a documented answer to the repository.

## Synopsis

```console
/ask <QUESTION>
/ask --record <QUESTION>
```

## Description

`/ask <QUESTION>` is the same as [`/chat`](chat.md) `<QUESTION>`: the chat answers, and nothing is written.

`/ask --record <QUESTION>` makes an answer that stays. It is for questions about how the code works, how something is implemented, or what a change would touch and what could break. CodeLead finds the files for the question, the same way as [`/scan`](scan.md). It adds the survey's rules, history and tests when a modernization dossier exists, and the evidence that it has seen. A model then writes the answer:

- a summary, and how the code works, each statement anchored to a file and its lines;
- how a change would proceed, step by step, with the files of each step;
- what could break, with the file and the line.

CodeLead discards every statement that names no file, and says how many it discarded. It writes the answer to `docs/codelead/answers/` and keeps it as an entry of the knowledge base. See [The knowledge base](../concepts/knowledge-base.md).

> [!NOTE]
> **For team leads:** a recorded answer is a document that the team can read and review later. It names the model that wrote it and the files that it rests on.

## Arguments

- **`<QUESTION>`**

  Your question, in plain words. It is required.

## Options

- **`--record`**

  Writes the answer to `docs/codelead/answers/` and records it in the knowledge base. It uses the `architect` model role.

## What you see

| Command | Outcome | Headline |
|---|---|---|
| `/ask <question>` | As [`/chat`](chat.md). | No headline: the answer is the output. |
| `/ask --record <question>` | `done` | `Answer written: docs/codelead/answers/<file>.md` |
| `/ask --record <question>` | `failed` | `The question could not be answered: <the error>` |

## Examples

- Record how a function works:

  ```console
  /ask --record how does openCount work?
  ```

  <!-- described, not captured -->
  ```output
  Ask: how does openCount work?
  Note: No dossier bundle under docs/codelead/dossier: the answer rests on the code alone (run /modernize --plan to add the survey's rules, practices, history and test coverage).
  Retrieved 3 files of 3 scanned: src/tasks.js, test/tasks.test.js, package.json.
  …
  Answer written: docs/codelead/answers/<file>.md; knowledge entry <id>.
  ```

- The same question, when the model server does not run:

  ```output
  Ask: how does openCount work?
  Note: No dossier bundle under docs/codelead/dossier: the answer rests on the code alone (run /modernize --plan to add the survey's rules, practices, history and test coverage).
  Retrieved 3 files of 3 scanned: src/tasks.js, test/tasks.test.js, package.json.
  The question could not be answered.
  Could not connect to lmstudio at http://localhost:1234/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ```

- Ask on the Basic Store sample, after `/modernize --plan` wrote the dossier:

  ```console
  /ask --record how is an order's total calculated?
  ```

- `/ask --record` without a question is refused:

  ```console
  /ask --record
  ```

  ```output
  /ask --record needs a question, e.g. /ask --record how is an order's total calculated?

  Usage:
    /ask <question>  (the same as /chat <question>)
    /ask --record <question about how the code works, how something is implemented, or what a change would touch>  (writes the answer to docs/codelead/answers/)
  ```

## Files

- Reads the repository's files, and `docs/codelead/dossier/` when it exists.
- With `--record`, writes the answer to `docs/codelead/answers/`, adds an entry to `.codelead/knowledge.json`, and lists the answer in `docs/codelead/CATALOG.md`.

## See also

- [`/chat`](chat.md)
- [`/scan`](scan.md)
- [`/modernize`](modernize.md)
- [The knowledge base](../concepts/knowledge-base.md)
