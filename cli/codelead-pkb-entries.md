---
title: codelead pkb entries
description: List a project's entries with their trust, or open one entry with its history and the stored answers that cite it.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb entries

**Applies to:** CodeLead 0.1 and later

`codelead pkb entries` lists the entries of a project in the store of the [PKB service](codelead-pkb.md). This list is the project's ledger. Each line gives an entry's trust, its type, its summary and its version. With an entry's id, `entries` opens that entry in full.

## Name

`codelead pkb entries` - Lists a project's entries, filtered by type, trust, stale or words; with an id, one entry in full.

## Synopsis

```console
codelead pkb entries [<ENTRY_ID>] [--project <ID>] [--type <TYPE>] [--trust <LEVEL>] [--stale] [--query <WORDS>] [--dir <FOLDER>] [--json]
codelead pkb entries -h|--help
```

## Description

An entry is one thing that the project knows, in its encyclopedia. It has a type, a summary, a trust level and a history. Its version rises each time what it says changes. Answers cite entries at their versions.

Each line of the list starts with the mark of the entry's trust:

| Mark | Trust |
|---|---|
| `●` | Evidence-backed. |
| `◐` | Confirmed by a person. |
| `○` | Reported: nobody proved it yet. |
| `✕` | Downgraded. |

A line also says when an entry is stale, or in conflict with another one.

One entry in full shows its scope, its text, its sources, its evidence records and its history. It ends with the stored answers that cite it. An answer that cited an older version says so.

`entries` goes through the service when one is named. See [How the commands find the service](codelead-pkb-serve.md#how-the-commands-find-the-service).

## Options

- **`<ENTRY_ID>`**

  An entry's id, such as `e-7cb74370cbf98f2023cc`. Shows that entry in full.

- **`--project <ID>`**

  The project to list. You can leave it out when `pkb.json` names one project only, or when `pkb.project` names it.

- **`--type <TYPE>`**

  Only entries of this type, such as `decision`, `invariant` or `validation_command`.

- **`--trust <LEVEL>`**

  Only entries at this trust: `evidence-backed`, `confirmed`, `reported` or `downgraded`.

- **`--stale`**

  Only entries marked stale: the files they are about changed after they were seen.

- **`--query <WORDS>`**

  Only entries whose summary or text holds these words.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The command then reads the store on this machine, not the service. The default is `<home>/.codelead/pkb`.

- **`--json`**

  Prints the result as one JSON document: the entries as stored, or one entry with `cited_by`.

## Examples

These examples use the sample project `tally`, through a service on port `7425`.

- List the ledger:

  ```console
  codelead pkb entries
  ```

  ```output
  Entries of tally: 14 (9 evidence-backed, 1 confirmed, 3 reported, 1 downgraded).
    ● e-10e06e77aa16924d3999 · contract_fact · Entry point POST /Invoice/Create: Invoice.Create (action) · version 1
    ● e-4ceae5bba4d1bb7da2cc · contract_fact · Walk: journey create-invoice: every step passed · version 1
    ◐ e-6db166295bf3516879c6 · decision · Decided on card 1: Keep the old invoice numbers in the rebuild · version 1
    ✕ e-7cb74370cbf98f2023cc · invariant · An invoice total is never negative · version 1
    …
    ○ e-ffa9d7b67c42d3734e22 · stack_fact · The web project targets .NET Framework 4.8 · version 1
  One entry in full: codelead pkb entries <entry id>. A person's confirm or downgrade: codelead pkb confirm <entry id> --by <name> --reason "<words>".
  ```

- List the reported entries only:

  ```console
  codelead pkb entries --trust reported
  ```

  ```output
  Entries of tally reported: 3 of 14 (3 reported).
    ○ e-9e32eaf63441ddc6a72b · failure_pattern · /implement patch_check_failed: request_coverage of the invoice page · version 1
    ○ e-f32a75effc7d0e7011ab · contract_fact · Entry point GET /Invoice/List: Invoice.List (action) · version 1
    ○ e-ffa9d7b67c42d3734e22 · stack_fact · The web project targets .NET Framework 4.8 · version 1
  One entry in full: codelead pkb entries <entry id>. A person's confirm or downgrade: codelead pkb confirm <entry id> --by <name> --reason "<words>".
  ```

- Open one entry. It was downgraded in the workspace before the sync:

  ```console
  codelead pkb entries e-7cb74370cbf98f2023cc
  ```

  ```output
  "An invoice total is never negative" · invariant · ✕ downgraded · version 1
  Id: e-7cb74370cbf98f2023cc; project tally; seen 1 time, last on 2026-11-24.
  Scope: workspace.

  InvoiceStore refuses a negative total.

  History:
    2026-11-24T14:55:24.488Z · created · by tally (workspace)
    2026-11-24T14:55:24.488Z · downgraded · by tally (workspace) · the import writes credit notes with a negative total
  No stored answer cites it.
  A person's confirm or downgrade: codelead pkb confirm e-7cb74370cbf98f2023cc --by <name> --reason "<words>", or codelead pkb downgrade.
  ```

- Ask for a filter that nothing matches:

  ```console
  codelead pkb entries --trust reported --type invariant
  ```

  ```output
  Entries of tally reported, of type invariant: 0 of 14.
  Nothing matches.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The list or the entry was shown. Also when nothing matches. |
| `1` | It started and failed. The words say why. |
| `2` | Refused: a type or a trust that does not exist, no `pkb.json`, no such project or entry, the store does not answer, or the service named does not answer. |

## See also

- [`codelead pkb confirm`](codelead-pkb-confirm.md) and [`codelead pkb downgrade`](codelead-pkb-downgrade.md): a person's act on an entry.
- [`codelead pkb asks`](codelead-pkb-asks.md): the stored answers and their marks.
- [The knowledge base](../concepts/knowledge-base.md): entries and trust, in each project's own folder.
- [`codelead pkb`](codelead-pkb.md)
