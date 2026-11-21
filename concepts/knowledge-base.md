---
title: The knowledge base
description: Learn what CodeLead knows about a repository, how much it trusts each fact, where the facts come from, and the words capsule, context package and capability profile.
kind: concept
area: concepts
date: 2026-11-20
---

# The knowledge base

The knowledge base is what CodeLead knows about one repository, kept as files in that repository. Its main part is the **encyclopedia**: a list of facts, each with its source and a trust level. CodeLead adds a fact only when something it ran or read shows the fact. This page also explains three more words of CodeLead's vocabulary: the capsule, the context package and the capability profile.

**In short**
- The encyclopedia is `.codelead/knowledge.json`: facts such as "the stack is C#" or "`dotnet build` passes", each with its evidence.
- Each fact has a trust level: `evidence_backed`, `confirmed`, `reported` or `downgraded`. Nothing is believed until it is shown.
- A context package is what one model call receives. A capsule is one governed unit of work. A capability profile fits the work to a model's strength.
- You see the facts in the extension's Knowledge base view and in the file itself.

## Why it exists

A model forgets everything between calls. A person who joins a team reads documents that are often out of date. CodeLead keeps what it learned in a file that stays with the code, and it keeps the proof beside each fact. The next request, the next person and the chat can then start from facts that were checked, not from guesses.

## How it works

### The encyclopedia

Each entry of the encyclopedia has a type, a short summary, a body, a trust level, its sources and the scope in which it holds. Today these commands write entries:

| What runs | The entries it writes | Trust level |
|---|---|---|
| [`codelead init`](../cli/codelead-init.md), [`codelead map`](../cli/codelead-map.md), [`/map`](../chat-commands/map.md), the end of a new app's build | The stack, the validation commands and the modules that the [repository map](repository-map.md) read | `evidence_backed` |
| An [`/implement`](../chat-commands/implement.md) that lands | "Validation command passes: …", one for each command that passed | `evidence_backed` |
| An `/implement` that stops, for example on the patch check or on the gate | A failure pattern, with the stop reason | `evidence_backed` |
| [`/ask --record`](../chat-commands/ask.md) | The question and the answer, as a trace | `reported` |
| A modernization's testimony: documents (`.md` or `.txt`) in which people describe the old application | One entry for each statement | `reported` |

The trust levels mean this:

| Trust level | Meaning |
|---|---|
| `evidence_backed` | Something that CodeLead ran or read shows the fact: a command that passed, a file that it read. |
| `confirmed` | A level between proof and report. No command writes it today. |
| `reported` | A person or a model said it, and nothing has proven it yet. |
| `downgraded` | Later evidence went against the fact. |

The same fact seen again is not added twice. CodeLead updates the entry's last-seen time and count. A fact can move up a trust level when new evidence backs it. CodeLead never moves a fact down without saying so: a move down is an explicit `downgraded` entry in its history.

A real entry, written by `codelead init` on the Basic Store sample:

```json
{
  "id": "k-20261120045758-5k8omg",
  "type": "stack_fact",
  "summary": "Stack: C#, JavaScript",
  "body": "C#, JavaScript; tools: .NET; an existing project.\n- C#: Found 26 .cs files, Found WebApplication1/webapp.csproj\n- JavaScript: Found 10 .js files\nDetected an existing project using C#, JavaScript; tools include .NET.\nSource: the repository map (docs/codelead/repository-map.json), read from the code with no model call, written 2026-11-20T04:57:58.426Z at git bd30c0ac3c5f (uncommitted changes); evidence record ev-20261120045758-skyvwz.",
  "trust_level": "evidence_backed",
  "source_links": [
    "docs/codelead/repository-map.json"
  ],
  "validity_scope": "the repository as its map read it: it holds while the map is fresh (the same git head, the file count within 2%); codelead map reads it again",
  "reuse_tags": [
    "repository-map"
  ],
  "history": [
    {
      "at": "2026-11-20T04:57:58.462Z",
      "event": "created"
    }
  ],
  "created_at": "2026-11-20T04:57:58.462Z",
  "last_seen": "2026-11-20T04:57:58.462Z",
  "seen_count": 1
}
```

The entry names the [evidence record](evidence-records.md) of the read that wrote it. That record lists the entries it stands behind in its `knowledge_ids`.

> [!NOTE]
> **For team leads:** the [Knowledge base view](../extension/knowledge-base-view.md) of the extension shows the entries by group, with their trust level. The evidence records are beside them.

### Context packages

A context package is everything that one model call receives. For a change, it holds the request and the plan of the preflight. It also holds the files that CodeLead selected from the code, the repository map's summary and the [conventions](conventions.md) card. For a chat question, it holds the sources that CodeLead read for the question, between two markers, inside a size limit.

CodeLead builds the package. The model does not choose what it reads. You see every package in the [session log](sessions.md): each call's prompt is a file under `model/` in the session's folder.

### Capsules

A capsule is CodeLead's word for one governed unit of work. It is a request with its scope, its plan, its patch, its gate and its evidence record. There is no capsule file and no capsule command today. You see a capsule as one `/implement` run, one build step of a new app, or one increment, with its chain of evidence records.

### Capability profiles

A capability profile fits the size of the work to the strength of a model. A strong model gets larger slices of work. A small model gets smaller slices, a preflight brief before each one, and an edit format that it handles well. Today three profiles exist: `capable`, `text-edit` and `constrained`.

Today only the increment run driver, `scripts/incrementalBuildRun.ts`, applies a profile. It reads `CODELEAD_PROFILE`: a profile's name, `auto` (picked from the model's name) or `none`. The default is `none`. The interactive session and the extension do not apply a profile.

## What it is not

- **Not a copy of your documents**. The encyclopedia holds what CodeLead ran or read, with its source. It does not import a wiki.
- **Not sent anywhere**. The file stays in the repository. A model sees an entry only when CodeLead puts it in a context package. Today `/ask --record` does this: it joins the entries that name the files of the question. The patch prompts do not read the encyclopedia yet.
- **Not always right**. A fact holds within its `validity_scope`. A stale repository map makes its facts stale until the next read.

## Details

- **The file**. `.codelead/knowledge.json`, schema `codelead.knowledge/1`. `CODELEAD_KNOWLEDGE_DIR` moves it, with the evidence records.
- **Entry types**. `validation_command`, `stack_fact`, `failure_pattern`, `invariant`, `open_question`, `trace`, `contract_fact`, `cochange_fact`, `risk_fact`, `expert_testimony`.
- **Trust levels**, highest first: `evidence_backed`, `confirmed`, `reported`, `downgraded`. A fact recorded again keeps the higher level. A `downgraded` fact is not promoted again.
- **History events**. `created`, `seen`, `promoted`, `downgraded`, `edited`, each with its time.
- **Never in the way**. A store that cannot be read or written never fails the command that writes to it.
- **No model call**. Every entry is written by code, from recorded state. No model writes an entry, except the answer text of `/ask --record`, which is marked `reported`.

## Related content

- [`codelead map`](../cli/codelead-map.md), [`codelead init`](../cli/codelead-init.md) and [`/map`](../chat-commands/map.md)
- [`/ask`](../chat-commands/ask.md) and [`/chat`](../chat-commands/chat.md)
- [`/implement`](../chat-commands/implement.md)
- [The repository map](repository-map.md)
- [Evidence records](evidence-records.md)
- [Sessions](sessions.md)
- [The Knowledge base view](../extension/knowledge-base-view.md)
- [Files and folders](../reference/files-and-folders.md)
- [Glossary](glossary.md)
