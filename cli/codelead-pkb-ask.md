---
title: codelead pkb ask
description: Ask what a project knows. Each sentence of the answer names its sources and its trust. The question and its answer are stored.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb ask

**Applies to:** CodeLead 0.1 and later

`codelead pkb ask` answers a question from what the store of the [PKB service](codelead-pkb.md) holds about a project. A chat model writes the answer from the sources that the search found. The service keeps only the sentences that cite those sources. Each kept sentence says how far you can trust it. The question and its answer are stored, so you can open them again and see later when the evidence changed.

> [!IMPORTANT]
> Sync the project first: see [`codelead pkb sync`](codelead-pkb-sync.md). The chat model runs on a model server that `pkb.json` names. When nothing that the search found supports an answer, no model is called at all.

## Name

`codelead pkb ask` - Answers a question about a project, each sentence with its sources, and stores the question and its answer.

## Synopsis

```console
codelead pkb ask "<QUESTION>" [--project <ID>] [--dir <FOLDER>] [--by <NAME>] [--dry-run] [--confirm-remote] [--json]
codelead pkb ask -h|--help
```

## Description

You ask in plain words. `ask` makes the answer in these steps:

1. It scans the question for secrets, paths of this machine and private names. See [The question is scanned](#the-question-is-scanned).
2. When the same question was asked before in the project, it shows the stored answer and stops. Same question means the same words, in any case and with any punctuation. [`codelead pkb reask`](codelead-pkb-reask.md) asks it again.
3. It searches the project as [`codelead pkb search`](codelead-pkb-search.md) does: by words, by meaning and, for some questions, in the code.
4. When no hit clears the floor, the answer is **not in the knowledge base**. No model is called. The answer says what was searched and the rule.
5. Otherwise it builds the prompt from the best 12 to 20 hits that fit. See [The prompt](#the-prompt).
6. It sends the prompt to the chat model, and reads the reply.
7. It keeps the sentences that cite the sources they were given, and drops the rest. See [What the service drops](#what-the-service-drops).
8. It stores the question and the answer, and prints both.

> [!NOTE]
> **New to this?** A source is one thing that the project knows. It can be an entry of the encyclopedia, an evidence record or a section of the dossier. It can also be a decided card or a code file. The model reads only the sources it is given. It never decides what is true.

### The two parts of an answer

An answer has two parts.

- **Backed by evidence.** Each sentence here rests on evidence-backed sources only. Code at a named commit counts as evidence.
- **Not proven.** Each sentence here rests on at least one source that nobody proved. It says what it rests on: a person's decision, a confirmed entry, the dossier or a reported entry.

A sentence is as trusted as the weakest source it cites. The marks are those of the [knowledge base](../concepts/knowledge-base.md):

| Mark | Meaning |
|---|---|
| `●` | Evidence-backed. |
| `◐` | Confirmed by a person. |
| `○` | Reported: nobody proved it yet. |
| `‹/›` | Code, at a named commit. In the list of sources only. |

Under the sentences comes the list of the sources cited, numbered in the order they are first cited. Each one shows its label from the prompt, such as `(S2)`.

The last lines name the model and count the sources cited of those given. They also count what was dropped, and why. Each `Note:` says how the answer was made. One says what was searched. Another says how many hits the model was given.

### What the service drops

The service checks every sentence of the reply. It never leaves this to the model.

| The sentence | What happens |
|---|---|
| cites none of its sources | Dropped, and counted as "with no source". |
| names a source it was not given, such as `S42` | Dropped, and counted as "naming a source it was not given". This is so even when it also names a source it was given. |
| is empty | Dropped, and counted as "empty". |
| cites an evidence-backed source and a weaker one | Kept, at its weakest source, under "Not proven". It is counted as mixed. |
| cites sources it was given | Kept, at its weakest source. |

When nothing is kept, the answer is "not in the knowledge base", whatever the model said. A reply that is not the JSON asked for is still read. Its sentences are read as text, each with the `[S2]` marks it holds. A reply that is empty keeps nothing.

The instruction asks the model never to mix a reported and an evidence-backed source in one sentence. A model can still do it. The service then keeps the sentence and puts it under "Not proven". So a weaker source never borrows the trust of a stronger one.

The model may also say what the sources do not say. That note is printed apart. It is never a sentence of the answer, and never evidence.

### The prompt

The prompt has one instruction, the question, the numbered sources and the reply's format. Each source starts with a line that names it and its trust. For code, the line also says why the search found the file. Then comes its text.

```output
Answer questions about Orchard using only the numbered sources. End each sentence with the sources it rests on ([S2] or [S2, S5]); never mix a reported and an evidence-backed source in one sentence. If the sources do not answer, set notInKnowledgeBase and say what is missing.
Question: How do I add another limit to the cold room store?
[S1] code orchard:src/storage/ColdRoomRepository.ts lines 1–12 @ ed8b3d7 · why: lexical match; symbol/token match for cold, room, limit, store; related source file for selected context · evidence-backed:
import { MAX_CRATE_DEGREES, MAX_HUMIDITY } from "./limits";
…
[S2] evidence ev-20261101090000-1r65o3 · implement, landed · 2026-11-01 · evidence-backed:
Evidence record ev-20261101090000-1r65o3: implement · landed · 2026-11-01T09:00:00.000Z
Request: Add a humidity limit to the cold room store
…
Return only JSON: {"sentences": [{"text": "…", "sources": ["S1"]}], "notInKnowledgeBase": false, "missing": ""}
```

- The sources come in the order of the search. A source that does not clear the floor comes in only while fewer than 12 are taken.
- A downgraded source is never in the prompt. Neither is an earlier answer.
- A source's text stops at 2,400 characters. The whole prompt stops at 60,000 characters, about 15,000 tokens.
- `--dry-run` prints the whole prompt, with its size, and calls no model.

### The chat model

`pkb.json` names the chat model in its `chat` section:

| Key | Default | What it is |
|---|---|---|
| `chat.baseUrl` | `http://localhost:8080/v1` | The address of an OpenAI-compatible model server. |
| `chat.model` | none | The model that writes the answers. `ask` refuses until you name one. |
| `chat.remote` | `false` | Marks the server as remote. A server whose address is not on this machine or your network counts as remote anyway. |

The chat model follows the same rule as the [embeddings server](codelead-pkb-embed.md#the-embeddings-server):

- When `pkb.json` says `"paused": true`, no model is called. `ask` refuses and says how to go on.
- A remote server gets the question and its sources only when `privacy.allowRemoteModels` is `true` and you add `--confirm-remote`.
- Each send to a remote server is a line of the disclosure log, in the store and in `.codelead/disclosures.jsonl` of each repository. The line holds ids and counts, never the text.

### The question is scanned

A question is your own words, and you might paste a secret into it. So `ask` scans the question with the project's own rules before it uses it. These are the rules of [`codelead pkb sync`](codelead-pkb-sync.md#what-never-leaves).

- A password, a key or a token becomes `<redacted>`.
- A path of this machine becomes a placeholder, such as `<home>`.
- A private name of the project becomes `<name>`.

The scanned question is what is searched, sent, stored and printed. A note says how many things the scan replaced. The code that the model is given is scanned the same way, and so is what the model writes.

### Code in an answer

Code is read from each repository's checkout, as the search reads it. Code never goes into the store. When the model is given a code file, the store keeps a small record of it. The record holds the repository, the path, the commit and a hash of the file. The answer keeps the lines it cited and a hash of those lines. A later sync at a newer commit hashes those lines again. When they changed, the answer is marked. See [`codelead pkb asks`](codelead-pkb-asks.md#marks).

### Stored asks

Each answer is a dated snapshot that is never edited.

- [`codelead pkb asks`](codelead-pkb-asks.md) lists the stored questions with their answers and their marks.
- [`codelead pkb reask`](codelead-pkb-reask.md) asks a question again, and stores a new answer beside the old ones.
- [`codelead pkb promote`](codelead-pkb-promote.md) makes an answer an entry of the encyclopedia.

An answer is never a source of a later answer, unless a person makes it an entry.

## Options

- **`<QUESTION>`**

  The question, in words. Put it in quotes.

- **`--project <ID>`**

  The project of `pkb.json` to ask. You can leave it out when `pkb.json` names one project only.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--by <NAME>`**

  Who asks. The stored question names them. The default is `codelead pkb ask`.

- **`--dry-run`**

  Searches and prints the prompt that `ask` would send, with its size. It calls no model, not even the embeddings model, so it searches by words and in the code only. It stores nothing. It reads the store when the store answers and holds the project. Otherwise it reads the project into a store in memory, as `codelead pkb sync --dry-run` does.

- **`--confirm-remote`**

  Confirms that the question and its sources may go to a remote chat server or a remote embeddings server. It has an effect only when `privacy.allowRemoteModels` is `true`.

- **`--json`**

  Prints the result as one JSON document: the stored question, the answer with its sentences, citations and counts, and the seconds it took. A dry run prints the prompt, its sources and its size.

## Examples

These examples ask a small sample project, `orchard`. It holds a cold room's crates and an orchard's harvest plan.

- See the prompt for a "how do I" question, with no model call. The store is not set up here, so the project is read into memory:

  ```console
  codelead pkb ask "How do I add another limit to the cold room store?" --dry-run
  ```

  ```output
  Dry run: the PKB store is not set up; the project read into a store in memory, as codelead pkb sync --dry-run reads it. No model was called and nothing was stored.
  Searched the project orchard with codelead-shared by words (add limit cold room store), not by meaning, and in the code (3 files of 6 scanned): 6 hits, 3 clearing the floor.
  Note: By words alone: a dry run calls no model.
  Note: The structural route left out 3 files of the context package that hold none of the question's words (anchors and related files).
  The prompt: 2,540 characters, about 635 tokens; 6 sources of the 6 hits.
  The chat endpoint http://localhost:8080/v1 (qwen3.6-35b-a3b) would be asked.

  --- the prompt ---
  Answer questions about Orchard using only the numbered sources. …
  Question: How do I add another limit to the cold room store?
  [S1] code orchard:src/storage/ColdRoomRepository.ts lines 1–12 @ ed8b3d7 · why: lexical match; symbol/token match for cold, room, limit, store; related source file for selected context · evidence-backed:
  …
  [S6] dossier section orchard:docs/codelead/dossier/business-rules.md · Business rules found in the code — orchard > Cold room line 11 · reported:
  Business rules found in the code — orchard > Cold room
  A crate above 8 degrees or 95% humidity is moved out (`src/storage/ColdRoomRepository.ts:10`).
  Return only JSON: {"sentences": [{"text": "…", "sources": ["S1"]}], "notInKnowledgeBase": false, "missing": ""}
  --- end of the prompt ---
  ```

- Ask the same question with the store and the chat model:

  ```console
  codelead pkb ask "How do I add another limit to the cold room store?" --by person-a
  ```

  <!-- captured from the command's own code on the sample in memory, with a recorded reply; ids and times vary -->
  ```output
  Ask · orchard: "How do I add another limit to the cold room store?"
  Backed by evidence
    ● The limits live in src/storage/limits.ts, as MAX_CRATE_DEGREES and MAX_HUMIDITY. [1]
    ● ColdRoomRepository.saveCrate refuses a crate that is over a limit, so a new limit is checked there too. [2]
    ● The last limit, for humidity, landed this way, and npm test passed. [3]
  Not proven
    ○ A crate is never kept above 8 degrees, which the code enforces. [4, 1] (rests on a reported invariant)
  Sources cited
    [1] ‹/› code orchard:src/storage/limits.ts lines 1–3 @ ed8b3d7 (S3)
    [2] ‹/› code orchard:src/storage/ColdRoomRepository.ts lines 1–12 @ ed8b3d7 (S1)
    [3] ● evidence ev-20261101090000-1r65o3 · implement, landed · 2026-11-01 (S2)
    [4] ○ entry invariant · A crate is never kept above 8 degrees (S4)
  Model: qwen3.6-35b-a3b · 4 of 6 sources cited · dropped 0 · 1 sentence mixed an evidence-backed source with a weaker one: kept as not proven, at the weakest.
  Note: Searched the project orchard with codelead-shared by words (add limit cold room store), not by meaning, and in the code (3 files of 6 scanned): 6 hits, 3 clearing the floor.
  Note: The model was given 6 of the 6 hits (2,540 characters).
  …
  Stored: question q-414a21564963d8e5ab2b, answer a-389bf7221afc57adf5d3, asked 2026-11-24T12:19:35.669Z by person-a, in … seconds.
  Re-ask: codelead pkb reask q-414a21564963d8e5ab2b. Make it an entry: codelead pkb promote a-389bf7221afc57adf5d3 --by <name>.
  ```

  The fourth sentence cites a reported invariant and code. So it stands under "Not proven", and it is counted as mixed.

- Ask why. Two sentences of this reply are dropped:

  ```console
  codelead pkb ask "Why do invoice totals round half up?" --by person-a
  ```

  <!-- captured from the command's own code on the sample in memory, with a recorded reply; ids and times vary -->
  ```output
  Ask · orchard: "Why do invoice totals round half up?"
  Not proven
    ◐ Invoice totals round half up because person-a decided so on card 12. [1] (rests on a person's decision)
    ◐ The finance team checks the totals by hand. [1] (rests on a person's decision)
    ○ An invoice total is kept in cents. [2] (rests on the dossier, reported)
  Sources cited
    [1] ◐ entry decision · Decided on card 12: Round invoice totals half up (S1)
    [2] ○ dossier section orchard:docs/codelead/dossier/business-rules.md · Business rules found in the code — orchard > Invoices line 7 (S2)
  Model: qwen3.6-35b-a3b · 2 of 2 sources cited · dropped 2: 1 with no source and 1 naming a source it was not given.
  Note: Searched the project orchard with codelead-shared by words (invoice totals round half), not by meaning, and not in the code: 2 hits, 1 clearing the floor.
  Note: The model was given 2 of the 2 hits (995 characters).
  Note: By words alone: pkb.json names no embeddings model (embeddings.model).
  Stored: question q-90ca2ab9cdefd0f682bf, answer a-eb4fdfb7f32227424595, asked 2026-11-24T12:19:35.628Z by person-a, in … seconds.
  Re-ask: codelead pkb reask q-90ca2ab9cdefd0f682bf. Make it an entry: codelead pkb promote a-eb4fdfb7f32227424595 --by <name>.
  ```

  The reply had five sentences. One cited nothing ("Rounding half up is the usual choice in finance"). One cited `S7`, a source it was not given. Both were dropped.

- Ask the same question again, in other case. The stored answer is shown, and nothing is asked:

  ```console
  codelead pkb ask "why do invoice totals round half up"
  ```

  <!-- captured from the command's own code on the sample, read into a store in memory -->
  ```output
  Asked before: q-90ca2ab9cdefd0f682bf, 1 time. The stored answer is shown; nothing was asked.
  Ask · orchard: "Why do invoice totals round half up?" · answer 1 of 1, 2026-11-24
  Not proven
  …
  Re-ask: codelead pkb reask q-90ca2ab9cdefd0f682bf. Every answer it had: codelead pkb asks q-90ca2ab9cdefd0f682bf.
  ```

- Ask something that the project does not know. No model is called:

  ```console
  codelead pkb ask "How are employees paid for overtime?" --by person-a
  ```

  <!-- captured from the command's own code on the sample, read into a store in memory -->
  ```output
  Ask · orchard: "How are employees paid for overtime?"
  Not in the knowledge base.
  Model: none was called, because nothing cleared the floor.
  Note: Searched the project orchard with codelead-shared by words (employees paid overtime), not by meaning, and not in the code: 0 hits, 0 clearing the floor.
  Note: The rule: Not in the knowledge base when no hit clears the floor. …
  Stored: question q-4ee96f36c1867b47d17e, answer a-4e8fdfdf1fd60445ef50, asked 2026-11-24T12:19:35.670Z by person-a, in … seconds.
  Re-ask: codelead pkb reask q-4ee96f36c1867b47d17e.
  ```

- Ask when the store is not set up. Nothing is asked, and the exit code is `2`:

  ```console
  codelead pkb ask "Why do invoice totals round half up?"
  ```

  ```output
  The PKB store is not set up: the PKB's folder holds no password for it (.env). codelead pkb store up sets it up.
  Nothing was asked or stored.
  ```

- Forget the question:

  ```console
  codelead pkb ask
  ```

  ```output
  Give the question in words: codelead pkb ask "Why does the build skip the tests?".
  Nothing was asked or stored.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | An answer was given and stored, a stored answer was shown, or a dry run ran. This is also the code for "not in the knowledge base". |
| `1` | It started and failed: the chat server did not answer, or the store failed. Nothing was stored. The words say what to do. |
| `2` | Refused before anything was asked: no question, no `pkb.json`, no such project, the store does not answer or is not set up, `paused`, no chat model named, or the remote rule. |

## Files

| File | What `ask` does with it |
|---|---|
| `<home>/.codelead/pkb/pkb.json` | Reads the project, its repositories, the `chat`, `embeddings` and `privacy` settings, and `paused`. |
| `<home>/.codelead/pkb/.env` | Reads the store's password. |
| Each repository's checkout | Reads the code for the structural route. It writes nothing there, except a disclosure line after a send to a remote server. |
| `.codelead/disclosures.jsonl` | Gains one line for each send to a remote chat server. |

## Details

- The chat model gets one message, a temperature of 0.1, room for a reply of 2,000 tokens and a pinned reasoning effort of `medium`. A thinking model's reasoning is cut after 300 seconds or about 8,000 tokens. A server that puts the answer in its reasoning channel is read there.
- A thinking model's `<think>` text is never read as the answer.
- When the model says the sources do not fully answer, the sentences it cited are kept, and a note says so.
- The answer stores what the prompt held: each source's label, its line and its trust. The citations keep each entry's and each source's version, so a later change can be found.
- The answer names the model that the server says answered.

## See also

- [`codelead pkb`](codelead-pkb.md): the commands of the PKB service.
- [`codelead pkb search`](codelead-pkb-search.md): the search that an answer rests on, and the rule for "not in the knowledge base".
- [`codelead pkb asks`](codelead-pkb-asks.md): the stored questions, their answers and their marks.
- [`codelead pkb reask`](codelead-pkb-reask.md): ask a stored question again.
- [`codelead pkb promote`](codelead-pkb-promote.md): make an answer an entry.
- [`/ask`](../chat-commands/ask.md): the question in the chat, about the code.
- [The knowledge base](../concepts/knowledge-base.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
