---
title: The Knowledge base view
description: The evidence records of a project and its encyclopedia in one view - what each row means, what a click opens, and the Repository map node with its refresh.
kind: concept
area: extension
date: 2026-11-20
---

# The Knowledge base view

The **Knowledge base** view shows what CodeLead has proven and learned about a project, in two groups. **Evidence records** lists the record of each run: what was asked, what changed and what the checks said. **Encyclopedia** lists the facts that CodeLead knows about the code, starting with the **Repository map**. See [The knowledge base](../concepts/knowledge-base.md).

**In short**
- Each evidence record has a mark: green for a landing, yellow for a change that was applied but not validated, red for a stop.
- A record that came from a board card opens the board on that card. Any other record opens its own page.
- **Repository map** shows how long ago CodeLead read the repository. Its pencil reads it again with `codelead map --refresh`.
- The encyclopedia's entries are grouped by type, and coloured by how much CodeLead trusts them.

![The Knowledge Base view. The Evidence records group has 8 records. Each record has a green, yellow or red mark, an outcome, the first words of the request and a date. The Encyclopedia group has 4 entries. Its first row is Repository map, read 1 d ago, with its pencil. The entry types failure_pattern, invariant and validation_command follow.](../media/extension/knowledge-base-view.png)

## Why it exists

A team lead must be able to check what CodeLead did without reading the session logs. Evidence records are the short, proven record of each run. The encyclopedia is what CodeLead knows about the code, and how sure it is. Both are files in the repository: this view is where a person reads them. The view is in the free plan.

## Evidence records

With one project, the two groups are at the top of the view. With several projects, each project is a row, with its two groups under it. The group's description counts the records, for example `8 records`.

### A record's row

Each row is one run, or one chain of runs: a change and the repairs that followed it.

| Part | What it shows | Example |
|---|---|---|
| The mark | Green check: landed, or a read that went well. Yellow warning: applied, not validated. Red circle: stopped or failed. | ✔ |
| The label | The outcome and the first words of the request. A repair says its round. Other commands name themselves first. | `landed · Show a thank-you page after the payment` |
| The description | The date, the repairs and the final outcome of the chain, and the stage. | `2026-09-20 · 3 repairs → landed · pin` |
| The tooltip | The record's id, the command, the outcome and its stop reason, the request, and the checkpoint commit. | |

A row with repairs has an arrow: open it to see each repair's record.

### What a click opens

- **A record that came from a card**, such as a run of `/implement --card`, opens the [board](board-view.md) on that card. The request, the thread and the record's own comment are there. The document icon on the row opens the record's page.
- **Any other record** opens its page. The page shows the request, the outcome, the change surface (the files), the patch check and the validation. It also shows the conventions, the chain, the spec walk and the checkpoint commit.
- **The file icon** on any row opens the record's JSON file in the editor.

> [!NOTE]
> **For team leads:** every run writes an evidence record, landed or stopped. A reviewer can follow the request, the files, the check and the gate from the record alone. See [Evidence records](../concepts/evidence-records.md).

## Encyclopedia

The group's description counts the entries and says when the map was read, for example `4 entries · map read 1 d ago`.

### The Repository map node

**Repository map** is the first row of the encyclopedia when the project holds `docs/codelead/repository-map.md`. CodeLead writes the map when you initialize the folder, when a build finishes in an initialized folder, and when you run [`codelead map`](../cli/codelead-map.md) or [`/map`](../chat-commands/map.md). See [The repository map](../concepts/repository-map.md).

- **The description** says how long ago CodeLead read the repository: `read just now`, `read 12 min ago`, `read 3 h ago` or `read 1 d ago`. It is the age of the read, not a promise that the code did not change after it.
- **The tooltip** gives the file and the time of the read.
- **A click** opens the map's page, `docs/codelead/repository-map.md`: the modules, their role and files, the entry points, and how to build and test.
- **The pencil**, **Read the repository again**, runs `codelead map --refresh` in the project. Its output goes to the output panel **CodeLead**, and the view refreshes when it ends.

The map is what CodeLead reads first when you [`/implement`](../chat-commands/implement.md) a change, and what the chat reads first to answer a question about the code. Read it again after a large change, such as a new module or a moved folder.

> [!TIP]
> **For experienced users:** the same refresh is **CodeLead: Read the repository again** in the Command Palette. It runs only in a trusted folder. When the command fails, the message says "the repository map could not be refreshed" with the exit code.

### The entries

Under the map, the entries are grouped by type, such as `failure_pattern`, `invariant`, `validation_command` or `stack_fact`. Each group shows its count.

Each entry row shows its summary. Its description is its trust level, and how often CodeLead saw it, for example `evidence_backed · seen 3×`. Its colour follows the trust level:

| Trust level | Colour | Meaning |
|---|---|---|
| `evidence_backed` | Green | Something that CodeLead ran or read shows the fact. |
| `confirmed` | Cyan | A level between proof and report. No command writes it today. |
| `reported` | Grey | A person or a model said it, and nothing has proven it yet. |
| `downgraded` | Red | Later evidence went against the fact. |

A click opens the knowledge ledger page, `Knowledge — <project>`, at that entry. The page shows each entry with its trust, its history and its sources. See [The knowledge base](../concepts/knowledge-base.md#the-encyclopedia).

## When a group is empty

An empty group shows one row: "No evidence records" or "No knowledge entries". In a folder with no CodeLead files, the view says "No CodeLead files in this folder yet. Initialize CodeLead from the Projects view above."

## Details

- **The files.** The evidence records are `.codelead/evidence/ev-*.json`. The entries are in `.codelead/knowledge.json`. The map is `docs/codelead/repository-map.md` and `.json`. See [Files and folders](../reference/files-and-folders.md).
- **The chain.** CodeLead links a repair's record to the record that it follows, with the field `follows`. The view groups them, and the description gives the final outcome.
- **The read's record.** Each read of the repository writes an evidence record with the outcome `success`. It also seeds the encyclopedia with facts about the stack, the validation commands and the modules.
- **Refresh.** The view refreshes when CodeLead writes under `.codelead/` or `docs/codelead/`.

## Related content

- [The knowledge base](../concepts/knowledge-base.md)
- [Evidence records](../concepts/evidence-records.md)
- [The repository map](../concepts/repository-map.md)
- [`codelead map`](../cli/codelead-map.md) and [`/map`](../chat-commands/map.md)
- [The Board view](board-view.md)
- [Evidence record format](../reference/evidence-record-format.md)
