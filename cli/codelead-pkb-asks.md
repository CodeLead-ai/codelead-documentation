---
title: codelead pkb asks
description: List a project's stored questions, each with its dated answers and the marks that say the evidence changed.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb asks

**Applies to:** CodeLead 0.1 and later

`codelead pkb asks` lists the questions that [`codelead pkb ask`](codelead-pkb-ask.md) stored for a project. Each question shows its answers, newest first, with their dates. When the evidence that an answer rested on changed, the answer carries a mark. With a question's id, `asks` shows that question with every answer in full.

## Name

`codelead pkb asks` - Lists a project's stored questions with their answers and their marks; with an id, one question in full.

## Synopsis

```console
codelead pkb asks [<QUESTION_ID>] [--project <ID>] [--dir <FOLDER>] [--json]
codelead pkb asks -h|--help
```

## Description

A stored question keeps every answer it ever had. An answer is a dated snapshot: its sentences, its sources at their versions, the model and the counts. It is never edited. So the list shows how the answer to a question moved with the project.

Each question takes a few lines:

```output
q-90ca2ab9cdefd0f682bf · orchard · "Why do invoice totals round half up?" · asked 2 times
  answer 2 · 2026-11-24 · current · not in the knowledge base · a-5a2dc9a4ec8925622808
  answer 1 · 2026-11-24 · a-eb4fdfb7f32227424595
      ⚠ the evidence changed on 2026-11-24: an entry it cited was downgraded by person-b (the finance team rounds to even from December): "Decided on card 12: Round invoice totals half up"
```

- The first line gives the question's id, the project, the words and how many times it was asked.
- Each answer line gives its number, its date and its id. The current answer is the newest. An answer with no mark shows its trust mix, such as `2 confirmed, 1 reported`, or "not in the knowledge base".
- A mark line starts with `⚠`. It gives the date of the change and what changed.
- A question that a person made an entry ends with `Made an entry: <entry id>`.

The questions asked last come first.

### Marks

An answer is marked when something it cited changed after it was given. The answer itself stays as it was. These changes mark an answer:

| What changed | Which answers are marked |
|---|---|
| An entry was downgraded or confirmed by a person, or downgraded in a workspace and synced | The answers that cited the entry at an older version. |
| An entry changed at a sync: its text, its trust, a stale mark, its scope or its links | The answers that cited the entry at an older version. |
| A file of the project changed, so its source has a new version | The answers that cited the version before. |
| An evidence record changed | The answers that cited it. |
| Code lines that an answer cited changed at a newer commit | The answers that cited those lines. A sync finds this when a checkout's commit moved. |

An answer is marked only once for one change. An answer that cited something else is never marked. [`codelead pkb sync`](codelead-pkb-sync.md#stored-answers) says how many answers it marked.

When the latest answer of a question is marked, re-ask it with [`codelead pkb reask`](codelead-pkb-reask.md).

## Options

- **`<QUESTION_ID>`**

  A stored question's id, such as `q-90ca2ab9cdefd0f682bf`. An answer's id, such as `a-eb4fdfb7f32227424595`, also works: then its question is shown. `asks` then shows that question with every answer in full, newest first.

- **`--project <ID>`**

  The project of `pkb.json` to list. You can leave it out when `pkb.json` names one project only. A question's id names its project already.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--json`**

  Prints the questions as one JSON document, each with all its answers as stored.

## Examples

These examples use a small sample project, `orchard`, after four questions. Then a person downgraded the decision of card 12.

- List the stored questions:

  ```console
  codelead pkb asks
  ```

  <!-- captured from the command's own code on the sample, read into a store in memory; ids and dates vary -->
  ```output
  Stored questions of the project orchard: 4; the latest answer of 1 rests on evidence that changed.

  q-414a21564963d8e5ab2b · orchard · "How do I add another limit to the cold room store?" · asked 1 time
    answer 1 · 2026-11-24 · current · 3 evidence-backed, 1 reported · a-389bf7221afc57adf5d3

  q-4ee96f36c1867b47d17e · orchard · "How are employees paid for overtime?" · asked 1 time
    answer 1 · 2026-11-24 · current · not in the knowledge base · a-4e8fdfdf1fd60445ef50

  q-90ca2ab9cdefd0f682bf · orchard · "Why do invoice totals round half up?" · asked 1 time
    answer 1 · 2026-11-24 · current · a-eb4fdfb7f32227424595
        ⚠ the evidence changed on 2026-11-24: an entry it cited was downgraded by person-b (the finance team rounds to even from December): "Decided on card 12: Round invoice totals half up"

  q-d7c0f5820199eb1a8ab2 · orchard · "Which test command does the project run?" · asked 1 time
    answer 1 · 2026-11-24 · current · not in the knowledge base · a-5d5b09e0f18695da2e89

  Re-ask one whose evidence changed: codelead pkb reask q-90ca2ab9cdefd0f682bf
  ```

  Only the answer that cited the decision is marked. The answer about the cold room cited other sources.

- Open one question with every answer in full, after it was asked again:

  ```console
  codelead pkb asks q-90ca2ab9cdefd0f682bf
  ```

  <!-- captured from the command's own code on the sample, read into a store in memory; ids and dates vary -->
  ```output
  q-90ca2ab9cdefd0f682bf · orchard · "Why do invoice totals round half up?" · asked 2 times
    answer 2 · 2026-11-24 · current · not in the knowledge base · a-5a2dc9a4ec8925622808
    answer 1 · 2026-11-24 · a-eb4fdfb7f32227424595
        ⚠ the evidence changed on 2026-11-24: an entry it cited was downgraded by person-b (the finance team rounds to even from December): "Decided on card 12: Round invoice totals half up"

  Answer 2 · 2026-11-24T12:19:35.673Z · current · a-5a2dc9a4ec8925622808 · follows a-eb4fdfb7f32227424595
  Not in the knowledge base.
  Model: none was called, because nothing cleared the floor.
  …

  Answer 1 · 2026-11-24T12:19:35.628Z · a-eb4fdfb7f32227424595
  Not proven
    ◐ Invoice totals round half up because person-a decided so on card 12. [1] (rests on a person's decision)
    ◐ The finance team checks the totals by hand. [1] (rests on a person's decision)
    ○ An invoice total is kept in cents. [2] (rests on the dossier, reported)
  Sources cited
    [1] ◐ entry decision · Decided on card 12: Round invoice totals half up (S1)
    [2] ○ dossier section orchard:docs/codelead/dossier/business-rules.md · Business rules found in the code — orchard > Invoices line 7 (S2)
  Model: qwen3.6-35b-a3b · 2 of 2 sources cited · dropped 2: 1 with no source and 1 naming a source it was not given.
  …
  ⚠ the evidence changed on 2026-11-24: an entry it cited was downgraded by person-b (the finance team rounds to even from December): "Decided on card 12: Round invoice totals half up"

  Re-ask: codelead pkb reask q-90ca2ab9cdefd0f682bf.
  ```

  The first answer rested on the decision. Once the decision was downgraded, nothing clears the floor for this question any more. So the second answer says "not in the knowledge base".

- List when the store is not set up. The exit code is `2`:

  ```console
  codelead pkb asks
  ```

  ```output
  The PKB store is not set up: the PKB's folder holds no password for it (.env). codelead pkb store up sets it up.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The list or the question was shown. This is also the code when no question is stored yet. |
| `1` | It started and failed. The words say why. |
| `2` | Refused: no `pkb.json`, no such project or question, or the store does not answer or is not set up. |

## Files

| File | What `asks` does with it |
|---|---|
| `<home>/.codelead/pkb/pkb.json` | Reads the project and the store's address. |
| `<home>/.codelead/pkb/.env` | Reads the store's password. |

`asks` only reads. It writes nothing.

## See also

- [`codelead pkb ask`](codelead-pkb-ask.md): ask a question, and store its answer.
- [`codelead pkb reask`](codelead-pkb-reask.md): ask a stored question again.
- [`codelead pkb promote`](codelead-pkb-promote.md): make an answer an entry.
- [`codelead pkb sync`](codelead-pkb-sync.md): the sync that marks answers when what they cited changed.
- [The knowledge base](../concepts/knowledge-base.md)
