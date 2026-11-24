---
title: codelead pkb search
description: Search what a project knows for a question, by words, by meaning and in the code. Each hit says why it was found. No chat model is called.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb search

**Applies to:** CodeLead 0.1 and later

`codelead pkb search` finds what the store of the [PKB service](codelead-pkb.md) holds about a question. It searches by words and by meaning. For some questions it also reads the code. Each hit says why it was found and how far you can trust it. The command never calls a chat model, so it writes no answer.

> [!IMPORTANT]
> This command shows what an answer would rest on. The answer itself comes later, with `codelead pkb ask`. Sync the project first: see [`codelead pkb sync`](codelead-pkb-sync.md).

## Name

`codelead pkb search` - Searches a project's knowledge for a question and shows each hit with its reasons.

## Synopsis

```console
codelead pkb search "<QUESTION>" [--project <ID>] [--dir <FOLDER>] [--dry-run] [--confirm-remote] [--json]
codelead pkb search -h|--help
```

## Description

You ask a question in plain words. `search` looks for it in three ways, and then makes one list of the best 30 hits.

| Way | What it finds | When |
|---|---|---|
| Words | Pieces of text that hold the words of the question. A name in the code counts whole and in words: `ColdRoomRepository` also matches `cold room repository`. | Always. |
| Meaning | Pieces of text that mean the same thing in other words. It compares [embeddings](codelead-pkb-embed.md). | When the project's pieces have their embeddings, and the embeddings server answers. |
| The code | The files of the project's repositories that the question reaches, read the way [`/ask`](../chat-commands/ask.md) reads them. With each file come the records and entries about it. | When the question asks where, which file or how do I, or names something in the code. |

> [!NOTE]
> **New to this?** A hit is one piece of what the project knows. It can be an entry of the encyclopedia, an evidence record or a section of the dossier. It can also be a decided card, or a part of a code file. Each hit names its source, so you can open it.

### The structural route

The code is read when the question takes the structural route. A rule chooses it, with no model call. The question takes it when:

- it asks **where**: "Where is the ripeness score computed?"
- it asks **which file**: "Which files change together with the data source?"
- it asks **how do I**: "How do I add another limit to the cold room store?"
- it **names something in the code**: "What changed in `ColdRoomRepository`?" A name counts when it is written like code. Examples are `camelCase`, `PascalCase`, `snake_case`, `Company.Data`, `src/app.ts` and `save()`.

The project's own name and its repositories' names do not count. The code is read from each repository's checkout on this machine. Code never goes into the store.

### One list

Each way gives its own ranked list. `search` joins them by reciprocal rank fusion: a hit's score adds 1 / (60 + its rank) for each list that found it. A hit found by two ways ranks above a hit found by one.

Trust never adds to a score. It decides only between two hits with the same score, and then evidence-backed comes first. So a trusted source that does not answer the question never climbs the list.

### What each hit says

```output
 1. ● evidence ev-20261101090000-4wi488 · implement, landed · 2026-11-01
    why: words #1 (add, limit, cold, room, store); structural #1: an evidence record whose change touched src/storage/ColdRoomRepository.ts, which the structural route found; fused 0.0328; evidence-backed; 22 days old. Clears the floor: it holds 5 of the question's 6 words (add, limit, cold, room, store).
    Evidence record ev-20261101090000-4wi488: implement · landed · 2026-11-01T09:00:00.000Z Request: Add a humidity limit to the cold room store …
```

- The first line gives the rank, the trust mark and the source.
- The `why:` line gives the reasons. They are the words matched and the rank by words, then the similarity and the rank by meaning. Then come the structural reason, the fused score, the trust and the age.
- The last line is the start of the text.

The trust marks are those of the [knowledge base](../concepts/knowledge-base.md):

| Mark | Meaning |
|---|---|
| `●` | Evidence-backed. |
| `◐` | Confirmed by a person. |
| `○` | Reported: nobody proved it yet. |
| `✕` | Downgraded: the evidence contradicts it. |
| `‹/›` | Code, at a named commit. It counts as evidence at that commit. |

### Conflicts and earlier answers

- A **downgraded** source is listed apart, under `Conflicts`. It shows what was once said, and never counts as support.
- An **earlier answer** is never a hit. Answers of `/ask`, and questions asked before, are left out and counted. A person can make an answer an entry. Then it counts, as a confirmed entry.

### Not in the knowledge base

The last lines say whether the question is in the knowledge base. The rule is the same for every question:

- A hit **clears the floor** when it holds at least half of the question's words, and at least two of them.
- It also clears the floor when its meaning is at least 0.6 close to the question's meaning.
- It also clears the floor when it holds a name in the code that the question names.
- When no hit clears the floor, the question is **not in the knowledge base**. An answer would then say so, with no model call.

Words like "why", "the" and "is" do not count as the question's words. A conflict never clears the floor.

### Projects

`search` searches one project. It also searches `codelead-shared`, the project that holds CodeLead's own knowledge, when the store holds it.

### Privacy

To search by meaning, `search` sends the question to the embeddings server. It sends nothing else.

- An embeddings server on this machine or on your network is fine. That is the default.
- A remote server gets the question only when `privacy.allowRemoteModels` is `true` in `pkb.json` and you add `--confirm-remote`. The send is then a line of the disclosure log.
- When `pkb.json` says `paused`, no model is called. `search` answers by words alone and says so.

## Options

- **`<QUESTION>`**

  The question, in words. Put it in quotes.

- **`--project <ID>`**

  The project of `pkb.json` to search. You can leave it out when `pkb.json` names one project only.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--dry-run`**

  Reads the project into a store in memory, as `codelead pkb sync --dry-run` reads it, and searches that. It needs no store and calls no model, so it searches by words and in the code only. Nothing is written.

- **`--confirm-remote`**

  Confirms that the question may go to a remote embeddings server. It has an effect only when `privacy.allowRemoteModels` is `true`.

- **`--json`**

  Prints the result as one JSON document: the route, each hit with its text and reasons, the conflicts, and `not_in_kb`. The default is text.

## Examples

These examples search a small sample project, `orchard`. It holds a cold room's crates and an orchard's harvest plan.

- Ask how to make a change. The question asks "how do I", so the code is read too:

  ```console
  codelead pkb search "How do I add another limit to the cold room store?" --dry-run
  ```

  ```output
  Dry run: the project read into a store in memory, as codelead pkb sync --dry-run reads it. Nothing was written.
  Search of the project orchard (with codelead-shared): "How do I add another limit to the cold room store?"
  Route: words (add another limit cold room store), the structural route, as it asks how do I (3 files of 7 scanned; 2 entries and records joined).
  Note: By words alone: a dry run calls no model.
  Note: The structural route left out 4 files of the context package that hold none of the question's words (anchors and related files).
  5 hits, best first.

   1. ● evidence ev-20261101090000-4wi488 · implement, landed · 2026-11-01
      why: words #1 (add, limit, cold, room, store); structural #1: an evidence record whose change touched src/storage/ColdRoomRepository.ts, which the structural route found; fused 0.0328; evidence-backed; 22 days old. Clears the floor: it holds 5 of the question's 6 words (add, limit, cold, room, store).
      Evidence record ev-20261101090000-4wi488: implement · landed · 2026-11-01T09:00:00.000Z Request: Add a humidity limit to the cold room store Files: src/storage/ColdRoomRepository.ts, src/storage/limits.ts (+12 −1) Valid…

   2. ○ entry invariant · A crate is never kept above 8 degrees
      why: words #2 (cold, room); structural #2: an entry about src/storage/limits.ts, which the structural route found; fused 0.0323; reported; today. Below the floor: it holds 2 of the question's 6 words.
      A crate is never kept above 8 degrees The cold room holds the crates at 2 to 4 degrees; a crate above 8 degrees is moved out.

   3. ‹/› code orchard:src/storage/ColdRoomRepository.ts lines 1–12 @ f03bf61
      why: structural #1: lexical match; symbol/token match for cold, room, limit, store; related source file for selected context; fused 0.0164; code, evidence at its commit; today. Clears the floor: it holds 4 of the question's 6 words (limit, cold, room, store).
      import { MAX_CRATE_DEGREES, MAX_HUMIDITY } from "./limits"; export interface IColdRoomStore { saveCrate(crate: { id: string; degrees: number; humidity: number }): Promise<void>; } /** Keeps the crates of the cold room i…

   4. ‹/› code orchard:src/storage/limits.ts lines 1–3 @ f03bf61
      why: structural #2: lexical match; symbol/token match for limit, cold, room; related source file for selected context; fused 0.0161; code, evidence at its commit; today. Clears the floor: it holds 3 of the question's 6 words (limit, cold, room).
      /** The cold room's limits: a crate above these is moved out. */ export const MAX_CRATE_DEGREES = 8; export const MAX_HUMIDITY = 0.95;

   5. ‹/› code orchard:README.md lines 1–3 @ f03bf61
      why: structural #3: lexical match; symbol/token match for cold, room; fused 0.0159; code, evidence at its commit; today. Below the floor: it holds 2 of the question's 6 words.
      # Orchard Orchard keeps the harvest plan and the crates of the cold room.

  In the knowledge base: 3 of the hits clear the floor.
  The rule: Not in the knowledge base when no hit clears the floor. A hit clears it when it holds at least 50% of the question's words, and at least two of them when the question has two or more; or when its meaning is at least 0.6 close to the question's (cosine similarity); or when it holds an identifier that the question names. Conflicts and earlier answers never count.
  No chat model was called.
  ```

  The evidence record comes first: its words match, and its change touched the files that the code route found.

- Ask why. This question does not take the structural route:

  ```console
  codelead pkb search "Why do invoice totals round half up?" --dry-run
  ```

  ```output
  Dry run: the project read into a store in memory, as codelead pkb sync --dry-run reads it. Nothing was written.
  Search of the project orchard (with codelead-shared): "Why do invoice totals round half up?"
  Route: words (invoice totals round half). Not the structural route: the question asks no where, which file or how do I, and names no identifier.
  Note: By words alone: a dry run calls no model.
  2 hits, best first.

   1. ◐ entry decision · Decided on card 12: Round invoice totals half up
      why: words #1 (invoice, totals, round, half); fused 0.0164; confirmed; 21 days old. Clears the floor: it holds 4 of the question's 4 words (invoice, totals, round, half).
      Decided on card 12: Round invoice totals half up Decision: Half up, by person-a, on 2026-11-02. Asked: Should an invoice total round half up, or to even? Options: Half up; To even The finance team checks totals by hand,…

   2. ○ dossier section orchard:docs/codelead/dossier/business-rules.md · Business rules > Invoices line 9
      why: words #2 (invoice); fused 0.0161; reported; today. Clears the floor: it holds 2 of the question's 4 words (invoice, totals).
      Business rules > Invoices An invoice total is kept in cents.

  In the knowledge base: 2 of the hits clear the floor.
  …
  No chat model was called.
  ```

- Ask something that the project does not know:

  ```console
  codelead pkb search "How are employees paid for overtime?" --dry-run
  ```

  ```output
  Dry run: the project read into a store in memory, as codelead pkb sync --dry-run reads it. Nothing was written.
  Search of the project orchard (with codelead-shared): "How are employees paid for overtime?"
  Route: words (employees paid overtime). Not the structural route: the question asks no where, which file or how do I, and names no identifier.
  Note: By words alone: a dry run calls no model.
  No hit.

  Not in the knowledge base: no hit clears the floor. An answer would say so, with no model call.
  …
  No chat model was called.
  ```

- Search the store, with meaning. The store must run, and the project's pieces need their embeddings:

  ```console
  codelead pkb search "What is the highest chill allowed for produce?"
  ```

  <!-- described, not captured -->
  ```output
  Search of the project orchard (with codelead-shared): "What is the highest chill allowed for produce?"
  Route: words (highest chill allowed produce), meaning (text-embedding-nomic-embed-text-v1.5). Not the structural route: …
   1. ○ entry invariant · A crate is never kept above 8 degrees
      why: meaning #1, …; fused …; reported; 13 days old. Clears the floor: its meaning is … close to the question's (the floor is 0.6).
  …
  ```

  No word of this question is in the entry. Meaning finds it.

- Search when the store is not set up. Nothing is searched, and the exit code is `2`:

  ```console
  codelead pkb search "Why do invoice totals round half up?"
  ```

  ```output
  The PKB store is not set up: the PKB's folder holds no password for it (.env). codelead pkb store up sets it up.
  Nothing was searched.
  ```

- Forget the question:

  ```console
  codelead pkb search
  ```

  ```output
  Give the question in words: codelead pkb search "Why does the build skip the tests?".
  Nothing was searched.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The search ran. This is also the code when the question is not in the knowledge base. |
| `1` | The search started and failed. The words say why. |
| `2` | Refused before anything was searched: no question, no `pkb.json`, no such project, the store does not answer or is not set up, or an option is wrong. |

## Files

| File | What `search` does with it |
|---|---|
| `<home>/.codelead/pkb/pkb.json` | Reads the project, its repositories' checkouts, and the `embeddings` and `privacy` settings. |
| `<home>/.codelead/pkb/.env` | Reads the store's password. |
| Each repository's checkout | Reads the code for the structural route. It writes nothing there. |

## Details

- The words sent to the store leave out words like "why" and "the". A name in the code goes whole and in words.
- The store fuses words and meaning. MongoDB does it with its `$rankFusion` stage. The JSON says which one fused, in `fusion.by`.
- The code route keeps only the files that hold a word of the question or a name it gives. It leaves out the other files of its context package, and says how many.
- A search with no embedding to compare answers by words alone. Its first note says why: a dry run, `paused`, no model named, the server down, or no piece embedded yet.
- The floor's numbers are a first setting. The evaluation set measures them: see `docs/pkb/eval/questions.json` in the repository.

## See also

- [`codelead pkb`](codelead-pkb.md): the commands of the PKB service.
- [`codelead pkb sync`](codelead-pkb-sync.md): send a project's knowledge into the store.
- [`codelead pkb embed`](codelead-pkb-embed.md): make the embeddings that the search by meaning compares.
- [`/ask`](../chat-commands/ask.md): the question in the chat, which reads the code the same way.
- [The knowledge base](../concepts/knowledge-base.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
