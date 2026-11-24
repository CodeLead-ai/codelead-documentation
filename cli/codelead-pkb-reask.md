---
title: codelead pkb reask
description: Ask a stored question again. The new answer rests on what the project knows now, and the earlier answers stay as they were.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb reask

**Applies to:** CodeLead 0.1 and later

`codelead pkb reask` asks a stored question again. It searches afresh: what the store holds now, and the code as it is now. The new answer is stored beside the earlier ones, linked to the one it follows. The earlier answers are never changed, so the question keeps its whole history.

## Name

`codelead pkb reask` - Asks a stored question again, and stores the new answer beside the earlier ones.

## Synopsis

```console
codelead pkb reask <QUESTION_ID> [--dir <FOLDER>] [--by <NAME>] [--dry-run] [--confirm-remote] [--json]
codelead pkb reask -h|--help
```

## Description

Re-ask a question when its answer is marked, because the evidence it rested on changed. You can also re-ask after a sync brought new knowledge. [`codelead pkb asks`](codelead-pkb-asks.md) shows which answers are marked.

A re-ask works as [`codelead pkb ask`](codelead-pkb-ask.md) does, with these differences:

- It takes the question's id, not its words. The words are the stored ones.
- It never opens the stored answer: it always asks again.
- The new answer follows the question's latest answer. Its record names that answer (`previous_id`).
- It ends with a comparison: how many sources both answers cited, and how many only one of them cited.

An earlier answer is never a source of the new one. The search leaves earlier answers out. The prompt never holds them. So a wrong answer cannot prove itself the next time.

The same rules hold as for `ask`. Only the sentences that cite their sources are kept. When nothing clears the floor, no model is called. The question is the stored one, which was scanned when it was first asked.

## Options

- **`<QUESTION_ID>`**

  The stored question's id, such as `q-90ca2ab9cdefd0f682bf`. An answer's id, such as `a-eb4fdfb7f32227424595`, also works: then its question is asked again. [`codelead pkb asks`](codelead-pkb-asks.md) lists the ids.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--by <NAME>`**

  Who asks again. The stored question names them. The default is `codelead pkb reask`.

- **`--dry-run`**

  Searches and prints the prompt that `reask` would send. It calls no model and stores nothing. The store must run, because the question is there.

- **`--confirm-remote`**

  Confirms that the question and its sources may go to a remote chat server or a remote embeddings server. It has an effect only when `privacy.allowRemoteModels` is `true`.

- **`--json`**

  Prints the result as one JSON document: the question, the new answer, and `follows`, the id of the answer it follows.

## Examples

These examples use a small sample project, `orchard`. A person downgraded the decision of card 12 after the question was first asked.

- Ask the marked question again:

  ```console
  codelead pkb reask q-90ca2ab9cdefd0f682bf --by person-b
  ```

  <!-- captured from the command's own code on the sample, read into a store in memory; ids, times and seconds vary -->
  ```output
  Ask · orchard: "Why do invoice totals round half up?"
  Not in the knowledge base.
  Model: none was called, because nothing cleared the floor.
  Note: Searched the project orchard with codelead-shared by words (invoice totals round half), not by meaning, and not in the code: 1 hit, 0 clearing the floor.
  Note: The rule: Not in the knowledge base when no hit clears the floor. …
  Follows answer a-eb4fdfb7f32227424595 of 2026-11-24. Sources cited by both answers: 0; only by the earlier one: 2; only by this one: 0.
  Stored: question q-90ca2ab9cdefd0f682bf, answer a-5a2dc9a4ec8925622808, asked 2026-11-24T12:19:35.673Z by person-b, in … seconds.
  Re-ask: codelead pkb reask q-90ca2ab9cdefd0f682bf.
  ```

  The decision is downgraded now, so it is a conflict and never support. Nothing else clears the floor, so the new answer says "not in the knowledge base". The first answer is still there, with its mark.

- Ask again when the store is not set up. The exit code is `2`:

  ```console
  codelead pkb reask q-90ca2ab9cdefd0f682bf
  ```

  ```output
  The PKB store is not set up: the PKB's folder holds no password for it (.env). codelead pkb store up sets it up.
  Nothing was asked or stored.
  ```

- Forget the id:

  ```console
  codelead pkb reask
  ```

  ```output
  Name the stored question to ask again: codelead pkb reask <question id>. codelead pkb asks lists them.
  Nothing was asked or stored.
  ```

## Exit codes

The codes of [`codelead pkb ask`](codelead-pkb-ask.md#exit-codes). A question id that the store does not hold is refused with `2`.

## Files

The files of [`codelead pkb ask`](codelead-pkb-ask.md#files).

## See also

- [`codelead pkb ask`](codelead-pkb-ask.md): how an answer is made.
- [`codelead pkb asks`](codelead-pkb-asks.md): the stored questions and their marks.
- [`codelead pkb promote`](codelead-pkb-promote.md): make an answer an entry.
- [The knowledge base](../concepts/knowledge-base.md)
