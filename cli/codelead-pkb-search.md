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

When the question names a type, a function or a method, the route also reads where it is declared. For a type, it also reads what implements it or extends it. These files come first on the route's list. When the question asks what implements, such as "Where is `IShiftClock` implemented?", the implementing classes come before the declaration.

The reader knows these forms:

| Language | Declares | Implements or extends |
|---|---|---|
| C# | `interface IClock`, `class SystemClock`, `record`, `struct`, a method with its body | `class SystemClock(TimeProvider time) : IClock`, with or without the constructor's parameters |
| TypeScript, JavaScript, Java | `interface`, `class`, `type X =`, `function x(`, `const x =` | `class A extends B implements C` |
| Python | `class X`, `def x` | `class A(B):` |

A test project's classes, such as `Api.Tests/FakeClock.cs`, come after the code's own.

### One list

Each way gives its own ranked list. `search` joins them by reciprocal rank fusion: a hit's score adds 1 / (60 + its rank) for each list that found it. A hit found by two ways ranks above a hit found by one.

Trust never adds to a score. It decides only between two hits with the same score, and then evidence-backed comes first. So a trusted source that does not answer the question never climbs the list.

**Each source once.** An entry, an evidence record or a file of the store can have many pieces. In the list it stands once, by its best piece. When one of its pieces clears the floor, that piece stands for it. The hit then says how many more of its pieces were found. A piece whose very text a better hit holds is folded into that hit. The hit then names the other sources that hold the same text. Both happen before the best 30 are kept.

**Code near the top.** When the code is read and the route finds files, code and the rest alternate from the top. The best code file comes first, then the best other hit, then the second code file, and so on. A code file is found by one way only, so it would sink below the records about it.

### What each hit says

```output
 2. ● evidence ev-20261101090000-eji9hw · implement, landed · 2026-11-01
    why: words #1 (add, limit, cold, room, store); structural #1: an evidence record whose change touched src/storage/ColdRoomRepository.ts, which the structural route found; fused 0.0328; evidence-backed; 23 days old. Clears the floor: it holds 5 of the question's 5 words (add, limit, cold, room, store), 100% of their weight.
    Evidence record ev-20261101090000-eji9hw: implement · landed · 2026-11-01T09:00:00.000Z Request: Add a humidity limit to the cold room store …
```

- The first line gives the rank, the trust mark and the source.
- The `why:` line gives the reasons. They are the words matched and the rank by words, then the similarity and the rank by meaning. Then come the structural reason, the fused score, the trust and the age. Then comes how many more pieces of the same source were found, and who else holds the same text.
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

The last lines say whether the question is in the knowledge base. The rule is the same for every question.

First, each word of the question gets a **weight** by how rare it is in the project. A word that no piece of the project holds weighs 1. A word that every piece holds weighs almost 0. The weight is ln((N + 2) / (n + 1)) / ln(N + 2), for N pieces of which n hold the word. A line under the route shows each word with its weight:

```output
Words of the question, by how rare they are in the project's 12 chunks: refund 1.00 (held by none); policy 1.00 (held by none); returned 0.74 (1 chunk); goods 0.74 (1 chunk). The project never holds refund, policy: 58% of the question's weight.
```

Then:

- A hit **clears the floor** when it holds at least 50% of the question's weight, and at least two of its words.
- It also clears the floor when its meaning is at least 0.65 close to the question's meaning.
- It also clears the floor when it holds a name in the code that the question names.
- When no hit clears the floor, the question is **not in the knowledge base**. An answer would then say so, with no model call.

A word that the whole project never holds is strong evidence: the question is likely about something else. A word that every piece holds says nothing. So a hit cannot clear the floor on the question's common words alone.

Words like "why", "the", "is", "another" and "say" do not count as the question's words. A conflict never clears the floor.

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

These examples search a small sample project, `orchard`. It holds a cold room's crates, an orchard's harvest plan and a rota of pickers.

- Ask how to make a change. The question asks "how do I", so the code is read too, and code comes first:

  ```console
  codelead pkb search "How do I add another limit to the cold room store?" --dry-run
  ```

  ```output
  Dry run: the project read into a store in memory, as codelead pkb sync --dry-run reads it. Nothing was written.
  Search of the project orchard (with codelead-shared): "How do I add another limit to the cold room store?"
  Route: words (add limit cold room store), the structural route, as it asks how do I (3 files of 11 scanned; 2 entries and records joined).
  Words of the question, by how rare they are in the project's 12 chunks: add 0.74 (1 chunk); limit 0.74 (1 chunk); cold 0.58 (2 chunks); room 0.58 (2 chunks); store 0.74 (1 chunk).
  Order: the question takes the structural route, so code and the rest alternate from the top, code first.
  Note: By words alone: a dry run calls no model.
  Note: The structural route left out 8 files of the context package that hold none of the question's words (anchors and related files).
  5 hits, best first, each owner once.

   1. ‹/› code orchard:src/storage/ColdRoomRepository.ts lines 1–12 @ 982038c
      why: structural #1: lexical match; symbol/token match for cold, room, limit, store; related source file for selected context; fused 0.0164; code, evidence at its commit; today. Clears the floor: it holds 4 of the question's 5 words (limit, cold, room, store), 78% of their weight.
      import { MAX_CRATE_DEGREES, MAX_HUMIDITY } from "./limits"; export interface IColdRoomStore { saveCrate(crate: { id: string; degrees: number; humidity: number }): Promise<void>; } /** Keeps the crates of the cold room i…

   2. ● evidence ev-20261101090000-eji9hw · implement, landed · 2026-11-01
      why: words #1 (add, limit, cold, room, store); structural #1: an evidence record whose change touched src/storage/ColdRoomRepository.ts, which the structural route found; fused 0.0328; evidence-backed; 23 days old. Clears the floor: it holds 5 of the question's 5 words (add, limit, cold, room, store), 100% of their weight.
      Evidence record ev-20261101090000-eji9hw: implement · landed · 2026-11-01T09:00:00.000Z Request: Add a humidity limit to the cold room store Files: src/storage/ColdRoomRepository.ts, src/storage/limits.ts (+12 −1) Valid…

   3. ‹/› code orchard:src/storage/limits.ts lines 1–3 @ 982038c
      why: structural #2: lexical match; symbol/token match for limit, cold, room; related source file for selected context; fused 0.0161; code, evidence at its commit; today. Clears the floor: it holds 3 of the question's 5 words (limit, cold, room), 56% of their weight.
      /** The cold room's limits: a crate above these is moved out. */ export const MAX_CRATE_DEGREES = 8; export const MAX_HUMIDITY = 0.95;

   4. ○ entry invariant · A crate is never kept above 8 degrees
      why: words #2 (cold, room); structural #2: an entry about src/storage/limits.ts, which the structural route found; fused 0.0323; reported; today. Below the floor: it holds 2 of the question's 5 words (35% of their weight).
      A crate is never kept above 8 degrees The cold room holds the crates at 2 to 4 degrees; a crate above 8 degrees is moved out.

   5. ‹/› code orchard:README.md lines 1–3 @ 982038c
      why: structural #3: lexical match; symbol/token match for cold, room; fused 0.0159; code, evidence at its commit; today. Below the floor: it holds 2 of the question's 5 words (35% of their weight).
      # Orchard Orchard keeps the harvest plan and the crates of the cold room.

  In the knowledge base: 3 of the hits clear the floor.
  The rule: Not in the knowledge base when no hit clears the floor. Each word of the question weighs by how rare it is in the project: a word no chunk holds weighs 1, a word every chunk holds almost 0. A hit clears the floor when it holds at least 50% of the question's weight, and at least two of its words when it has two or more; or when its meaning is at least 0.65 close to the question's (cosine similarity); or when it holds an identifier that the question names. Conflicts and earlier answers never count.
  No chat model was called.
  ```

  The file to change comes first. The evidence record of the last change to it comes second.

- Ask where an interface is implemented. The class that implements it comes first, and the declaration after it:

  ```console
  codelead pkb search "Where is IShiftClock implemented?" --dry-run
  ```

  ```output
  Dry run: the project read into a store in memory, as codelead pkb sync --dry-run reads it. Nothing was written.
  Search of the project orchard (with codelead-shared): "Where is IShiftClock implemented?"
  Route: words (ishiftclock implemented shift clock), the structural route, as it asks where and it names the identifier IShiftClock (5 files of 11 scanned; 2 entries and records joined).
  Words of the question, by how rare they are in the project's 12 chunks: ishiftclock 0.74 (1 chunk); implemented 0.47 (3 chunks).
  Order: the question takes the structural route, so code and the rest alternate from the top, code first.
  Note: By words alone: a dry run calls no model.
  Note: The structural route left out 6 files of the context package that hold none of the question's words (anchors and related files).
  7 hits, best first, each owner once.

   1. ‹/› code orchard:src/rota/SystemShiftClock.ts lines 3–8 @ 982038c
      why: structural #1: SystemShiftClock implements IShiftClock (line 4: export class SystemShiftClock implements IShiftClock {); fused 0.0164; code, evidence at its commit; today. Clears the floor: it holds 2 of the question's 2 words (ishiftclock, implemented), 100% of their weight.
      3: /** The machine's own date. */ 4: export class SystemShiftClock implements IShiftClock { 5: today(): string { 6: return new Date().toISOString().slice(0, 10); 7: } 8: }

   2. ● evidence ev-20261105100000-213hfa · implement, landed · 2026-11-05
      why: words #1 (ishiftclock, shift, clock); structural #2: an evidence record whose change touched src/rota/IShiftClock.ts, which the structural route found; fused 0.0325; evidence-backed; 19 days old. Clears the floor: it holds 2 of the question's 2 words (ishiftclock, implemented), 100% of their weight.
      Evidence record ev-20261105100000-213hfa: implement · landed · 2026-11-05T10:00:00.000Z Request: Give the picker rota and the rota board the shift clock (IShiftClock) Files: src/rota/PickerRota.ts, src/rota/RotaBoard.ts…

   3. ‹/› code orchard:src/rota/IShiftClock.ts lines 1–4 @ 982038c
      why: structural #2: declares IShiftClock (line 2: export interface IShiftClock {); fused 0.0161; code, evidence at its commit; today. Clears the floor: it holds IShiftClock, which the question names.
      1: /** The day a shift of pickers starts on. */ 2: export interface IShiftClock { 3: today(): string; 4: }
  …
  ```

  The two files that use the interface, `PickerRota.ts` and `RotaBoard.ts`, come next, at 5 and 6.

- Ask why. This question does not take the structural route:

  ```console
  codelead pkb search "Why do invoice totals round half up?" --dry-run
  ```

  ```output
  Dry run: the project read into a store in memory, as codelead pkb sync --dry-run reads it. Nothing was written.
  Search of the project orchard (with codelead-shared): "Why do invoice totals round half up?"
  Route: words (invoice totals round half). Not the structural route: the question asks no where, which file or how do I, and names no identifier.
  Words of the question, by how rare they are in the project's 12 chunks: invoice 0.58 (2 chunks); totals 0.58 (2 chunks); round 0.74 (1 chunk); half 0.74 (1 chunk).
  Note: By words alone: a dry run calls no model.
  2 hits, best first, each owner once.

   1. ◐ entry decision · Decided on card 12: Round invoice totals half up
      why: words #1 (invoice, totals, round, half); fused 0.0164; confirmed; 22 days old. Clears the floor: it holds 4 of the question's 4 words (invoice, totals, round, half), 100% of their weight.
      Decided on card 12: Round invoice totals half up Decision: Half up, by person-a, on 2026-11-02. Asked: Should an invoice total round half up, or to even? Options: Half up; To even The finance team checks totals by hand,…

   2. ○ dossier section orchard:docs/codelead/dossier/business-rules.md · Business rules found in the code — orchard > Invoices line 11
      why: words #2 (invoice); fused 0.0161; reported; today. Below the floor: it holds 2 of the question's 4 words (44% of their weight).
      Business rules found in the code — orchard > Invoices An invoice total is kept in cents.

  In the knowledge base: 1 of the hits clear the floor.
  …
  No chat model was called.
  ```

- Ask something that the project does not know, though one piece holds half its words. The two words that say the most are in no piece:

  ```console
  codelead pkb search "What is the refund policy for returned goods?" --dry-run
  ```

  ```output
  Dry run: the project read into a store in memory, as codelead pkb sync --dry-run reads it. Nothing was written.
  Search of the project orchard (with codelead-shared): "What is the refund policy for returned goods?"
  Route: words (refund policy returned goods). Not the structural route: the question asks no where, which file or how do I, and names no identifier.
  Words of the question, by how rare they are in the project's 12 chunks: refund 1.00 (held by none); policy 1.00 (held by none); returned 0.74 (1 chunk); goods 0.74 (1 chunk). The project never holds refund, policy: 58% of the question's weight.
  Note: By words alone: a dry run calls no model.
  1 hit, best first, each owner once.

   1. ○ dossier section orchard:docs/codelead/dossier/business-rules.md · Business rules found in the code — orchard > Market line 15
      why: words #1 (returned, goods); fused 0.0164; reported; today. Below the floor: it holds 2 of the question's 4 words (42% of their weight).
      Business rules found in the code — orchard > Market Pallets returned by the market stall are checked again; bruised goods go to the cider press.

  Not in the knowledge base: no hit clears the floor. An answer would say so, with no model call.
  …
  No chat model was called.
  ```

- Ask something that the project does not know at all:

  ```console
  codelead pkb search "How are employees paid for overtime?" --dry-run
  ```

  ```output
  Dry run: the project read into a store in memory, as codelead pkb sync --dry-run reads it. Nothing was written.
  Search of the project orchard (with codelead-shared): "How are employees paid for overtime?"
  Route: words (employees paid overtime). Not the structural route: the question asks no where, which file or how do I, and names no identifier.
  Words of the question, by how rare they are in the project's 12 chunks: employees 1.00 (held by none); paid 1.00 (held by none); overtime 1.00 (held by none). The project never holds employees, paid, overtime: 100% of the question's weight.
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
      why: meaning #1, …; fused …; reported; 13 days old. Clears the floor: its meaning is … close to the question's (the floor is 0.65).
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
- The floor compares words by a light stem, so "totals" matches "total". A last doubled letter counts once, so "cancelled" matches "cancel".
- The store fuses words and meaning. MongoDB does it with its `$rankFusion` stage. The JSON says which one fused, in `fusion.by`.
- `search` asks the store for its best 100 pieces, so that 30 sources stay after each source stands once.
- The code route keeps only the files that hold a word of the question or a name it gives. It leaves out the other files of its context package, and says how many.
- A search with no embedding to compare answers by words alone. Its first note says why: a dry run, `paused`, no model named, the server down, or no piece embedded yet.
- The question's embedding is made with the query prefix of `pkb.json`. It is compared only with pieces embedded with the same model and prefixes. See [the task prefixes](codelead-pkb-embed.md#the-task-prefixes).
- The floor's numbers come from a first measure on a real project. There, two questions with no answer reached a meaning of 0.56 and 0.62. The numbers change with the model and its prefixes.
- The evaluation set measures the floor: see `docs/pkb/eval/questions.json` in the repository. Its scorer prints the floors that separate the set's answered questions from its "not in the knowledge base" ones.

## See also

- [`codelead pkb`](codelead-pkb.md): the commands of the PKB service.
- [`codelead pkb sync`](codelead-pkb-sync.md): send a project's knowledge into the store.
- [`codelead pkb embed`](codelead-pkb-embed.md): make the embeddings that the search by meaning compares.
- [`/ask`](../chat-commands/ask.md): the question in the chat, which reads the code the same way.
- [The knowledge base](../concepts/knowledge-base.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
