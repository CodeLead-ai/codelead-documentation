---
title: Evidence records
description: The file that CodeLead writes at the end of every run, field by field, how records form a chain, and why a team lead reads them.
kind: concept
area: concepts
date: 2026-11-20
---

# Evidence records

An **evidence record** is a small JSON file that CodeLead writes at the end of every run, whether the change landed or stopped. It says what was asked, which files changed, what the patch check said, and which commands of the gate ran with their exit codes. CodeLead builds it from what it recorded during the run, never from a model's words.

**In short**
- CodeLead writes one record per run to `.codelead/evidence/ev-<time>-<id>.json`, landed or stopped.
- A record answers four questions: what was asked, what changed, what the patch check said, and what the gate said.
- Records form a chain: a repair round's record names the record that it follows.
- A record holds request texts and file names, not your code. It is safe to commit and to review.

## Why it exists

> [!NOTE]
> **For team leads:** an evidence record lets you review the process without reading the session. You see the request, the files, the check's verdict and the exit codes of your own build and tests. A pull request made with [`/pr`](../chat-commands/pr.md) carries the record as its body.

"The model says it works" is a claim. "The build exited 0 and the tests exited 0 after this patch" is evidence. A reviewer, an auditor or a teammate who joins later needs the second kind.

CodeLead writes the record from its own state. It keeps the request that you typed, the files of the applied patch, the check's result and the commands that ran. A model writes none of it. A third person can follow the path from the request to the verdict with the record alone.

## How it works

### A record, field by field

Here is a record that CodeLead's own code assembles in its tests (`src/knowledge/__tests__/evidenceRecord.test.ts`). The workspace and the person are replaced by placeholders:

```json
{
  "id": "ev-20260902120000-7zv3ba",
  "schema": "codelead.evidence/1",
  "created_at": "2026-09-02T12:00:00.000Z",
  "workspace": "<your project>",
  "who": {
    "name": "<your name>",
    "hash": "<12 hex characters>"
  },
  "command": "implement",
  "request": "Add a Count method to the data access library",
  "outcome": "landed",
  "preflight": {
    "strategy": "architect",
    "confidence": 0.78
  },
  "surface": {
    "files": ["src/Repo.cs", "src/RepositoryBase.cs"],
    "additions": 151,
    "removals": 3
  },
  "patch_check": {
    "status": "can_apply",
    "problems": [],
    "warning_count": 2
  },
  "apply": {
    "status": "applied",
    "files_changed": 2
  },
  "validation": {
    "status": "passed",
    "commands": [
      { "command": "dotnet build", "status": "passed", "exitCode": 0 }
    ]
  },
  "knowledge_ids": ["k-1"]
}
```

| Field | What it says |
|---|---|
| `id` | The record's name: `ev-`, the time in UTC, and a short random part. The file is `<id>.json`. |
| `schema` | The format: `codelead.evidence/1`. |
| `created_at` | When the run ended, in UTC. |
| `workspace` | The repository that the run worked in. |
| `who` | The person who ran it: the git `user.name`, and a hash of the git `user.email`. The email itself is never stored. |
| `command` | The kind of run: `implement` here. A repair round writes `repair`. |
| `request` | The request, word for word. |
| `outcome` | How it ended: `landed` here. A run that did not land is `stopped`, with a `stop_reason`. |
| `preflight` | The [strategy](strategies.md) that the preflight used, and its confidence. It can also hold how many questions were asked and how many took the default. |
| `surface` | The files that the patch changed, with the lines added and removed. |
| `patch_check` | The [patch check](the-patch-check.md)'s status, its blocking findings and how many notes it printed. |
| `apply` | Whether the patch was applied, and to how many files. |
| `validation` | The [gate](validation-and-the-gate.md)'s verdict, and each command with its status and exit code. |
| `knowledge_ids` | The entries of the [knowledge base](knowledge-base.md) that this run created or confirmed. |

### A chain of records

When the gate refuses a change and a [repair round](repair.md) fixes it, there are two records. The first stops with `"stop_reason": "validation_failed"`. The second, written when the round's validation ends, points back to it:

```json
{
  "id": "ev-20260902120600-kx8huo",
  "command": "repair",
  "request": "Add a Count method to the data access library",
  "outcome": "landed",
  "follows": "ev-20260902120000-049sp5",
  "repair_round": 1,
  "repair_rounds": [
    { "round": 1, "outcome": "patch_applied", "validation": "passed" }
  ],
  …
}
```

Read a chain from its last record back with `follows`. The last record says how the task ended, and every round is listed in `repair_rounds`.

### Where you see records

- The session prints the record's path at the end of [`/implement`](../chat-commands/implement.md): `Evidence record: <your project>/.codelead/evidence/ev-….json`.
- [`/pr`](../chat-commands/pr.md) and [`codelead pr`](../cli/codelead-pr.md) use the last landed record as the body of the pull request. Then the record keeps the pull request's address.
- A card of the [board](../pinboard/index.md) worked by [`codelead implement`](../cli/codelead-implement.md) gets the record's id in its thread.
- The extension's [Knowledge base view](../extension/knowledge-base-view.md) lists the records and opens each one.

## What it is not

- **Not a session log.** The [session log](sessions.md) keeps every prompt and every answer. The record keeps the facts of the outcome, in a few kilobytes.
- **Not a copy of your code.** A record holds no diff and no source text. It names files and counts lines.
- **Not written by a model.** Every field comes from CodeLead's recorded state. A model's claim that "the tests pass" never reaches the record.
- **Not only for landings.** A run that stopped writes a record too, with the reason. A stop is evidence as well.

## Details

- **Folder.** `.codelead/evidence/` in the workspace. `CODELEAD_KNOWLEDGE_DIR` moves it, together with the knowledge base.
- **Outcomes.** `landed`, `stopped`, `cause_not_found` (the preflight did not find the cause of a bug), `not-landed`, `reopened`, `asked`, `success`, `failure`. Older records can also hold `landed_unvalidated`. CodeLead reads it and no longer writes it.
- **Common stop reasons.** `preflight_failed`, `patch_failed`, `patch_check_failed`, `validation_failed`, `no_repair_patch`, `repair_patch_check_failed`, `repair_cycle`, `reply_cut`, `validation_declined`, `validation_not_run`. See [Results and stops](results-and-stops.md).
- **Other kinds of record** (`command`). `map` when the [repository map](repository-map.md) is written, `rebuild-init`, `modernize` for a stage of the modernization program, and `judge-again`, `unapply`, `reopen`, `ask`, `intervene` for the parts of a rebuild. The map, rebuild-init and modernize records change no code. Readers of increments, such as `/pr`, pass over them.
- **Optional fields**. A field that does not apply is left out. The optional fields are:
  - `retries_used`, `cause_reading`, `request_lint`, `reply_cut`, `model_unreachable`;
  - `composed`, for a pin or a transform, and `preflight.assumptions`, for what a composed request assumed;
  - `specs`, for the walk of a spec after the landing, and `acceptance`;
  - `checkpoint_commit`, `checkpoint_branch`, `conventions.conformance_warnings` and `card`.
- **Seats.** The licence's seat count reads `who.hash` from the records of the last 30 days. See [Licence and plans](licence-and-plans.md).
- **Never fails a run.** Writing a record is advisory: a folder that cannot be written never fails the command that the record documents.
- **The full format** is on the [Evidence record format](../reference/evidence-record-format.md) page.

## Related content

- [`/implement`](../chat-commands/implement.md), [`/repair`](../chat-commands/repair.md), [`/validate`](../chat-commands/validate.md)
- [`/pr`](../chat-commands/pr.md) and [`codelead pr`](../cli/codelead-pr.md)
- [`codelead implement`](../cli/codelead-implement.md) and [`codelead licence`](../cli/codelead-licence.md)
- [Evidence record format](../reference/evidence-record-format.md)
- [Repair](repair.md)
- [The knowledge base](knowledge-base.md)
- [The audit pack](../modernization/audit-pack.md)
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
