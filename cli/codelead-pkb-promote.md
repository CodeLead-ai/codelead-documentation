---
title: codelead pkb promote
description: Make a stored answer an entry of the project's knowledge, by a person's act. The entry is confirmed, with the answer as its source.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb promote

**Applies to:** CodeLead 0.1 and later

`codelead pkb promote` makes a stored answer an entry of the project's encyclopedia in the store. A person does it, and names themself with `--by`. The entry is **confirmed**: a person said it holds. Later answers may cite it, as they cite any confirmed entry.

> [!IMPORTANT]
> An answer is never a source of another answer, unless a person promotes it. Promote an answer only when you checked what it says.

## Name

`codelead pkb promote` - Makes a stored answer a confirmed entry, with the answer as its source and the person's name in its history.

## Synopsis

```console
codelead pkb promote <ANSWER_ID> --by <NAME> [--dir <FOLDER>] [--json]
codelead pkb promote -h|--help
```

## Description

`promote` writes one entry of the type `question`:

| Part of the entry | What it holds |
|---|---|
| Summary | The question's words. |
| Body | Each sentence of the answer, with what it rests on, and a line that says who made the entry from which answer. |
| Trust | Confirmed, by the person named with `--by`. It is never evidence-backed. |
| History | "created" and "confirmed", both by that person, with the answer's id as the reason. |
| Links | The answer, the sources it cited, and the files they open. |
| Evidence | The evidence records the answer cited. |

The question records the entry too. [`codelead pkb asks`](codelead-pkb-asks.md) then shows `Made an entry: <entry id>` under it.

One question has one entry. When you promote a later answer of the same question, the entry gets a new version with that answer. The answers that cited the older version are then marked.

`promote` refuses an answer that says "not in the knowledge base", because there is nothing to make an entry of. It also refuses an answer whose evidence changed since it was given. Re-ask such a question first with [`codelead pkb reask`](codelead-pkb-reask.md), then promote the new answer.

The answer itself never changes. The entry's text is searchable by words at once. It gets its embedding the next time [`codelead pkb embed`](codelead-pkb-embed.md) runs.

## Options

- **`<ANSWER_ID>`**

  The stored answer's id, such as `a-389bf7221afc57adf5d3`. [`codelead pkb ask`](codelead-pkb-ask.md) prints it, and [`codelead pkb asks`](codelead-pkb-asks.md) lists it.

- **`--by <NAME>`**

  Who makes the entry. It is required: a promotion is a person's act, and the entry's history names them.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--json`**

  Prints the result as one JSON document: the entry as stored, the question, whether the entry is new, and how many answers were marked.

## Examples

These examples use a small sample project, `orchard`.

- Make the answer about the cold room's limits an entry:

  ```console
  codelead pkb promote a-389bf7221afc57adf5d3 --by person-b
  ```

  <!-- captured from the command's own code on the sample, read into a store in memory; ids vary -->
  ```output
  Made an entry of answer a-389bf7221afc57adf5d3 (question q-414a21564963d8e5ab2b): e-7e9bea11ed491a3c6902.
  "How do I add another limit to the cold room store?" · question · confirmed by person-b, version 2.
  It is a source of later answers now, as an entry a person confirmed; it is never evidence-backed by this. Its chunk waits for its embedding: codelead pkb embed makes it.
  ```

  The version is 2: the entry was made at version 1 and confirmed at version 2.

- Try to promote an answer whose evidence changed. The exit code is `2`:

  ```console
  codelead pkb promote a-eb4fdfb7f32227424595 --by person-b
  ```

  <!-- captured from the command's own code on the sample, read into a store in memory; ids and dates vary -->
  ```output
  The evidence the answer a-eb4fdfb7f32227424595 rested on changed on 2026-11-24 (an entry it cited was downgraded by person-b (the finance team rounds to even from December): "Decided on card 12: Round invoice totals half up"). Re-ask it with codelead pkb reask q-90ca2ab9cdefd0f682bf, then promote the new answer.
  Nothing was written.
  ```

- Forget the name:

  ```console
  codelead pkb promote a-389bf7221afc57adf5d3
  ```

  ```output
  Name who makes it an entry: --by <name>. A promotion is a person's act, and the entry's history names them.
  Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The entry was made, or it gained a new version. |
| `1` | It started and failed. The words say why. |
| `2` | Refused before anything was written: no answer id, no `--by`, no such answer, an answer that says "not in the knowledge base", an answer whose evidence changed, or the store does not answer. |

## Files

| File | What `promote` does with it |
|---|---|
| `<home>/.codelead/pkb/pkb.json` | Reads the store's address. |
| `<home>/.codelead/pkb/.env` | Reads the store's password. |

The entry is written into the store only. The project's own `.codelead/knowledge.json` is never changed.

## See also

- [`codelead pkb ask`](codelead-pkb-ask.md): ask a question, and store its answer.
- [`codelead pkb asks`](codelead-pkb-asks.md): the stored questions and the entries made from them.
- [`codelead pkb reask`](codelead-pkb-reask.md): ask a question again before you promote it.
- [The knowledge base](../concepts/knowledge-base.md): entries and their trust levels.
