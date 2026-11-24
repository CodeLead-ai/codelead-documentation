---
title: Evidence record format
description: The evidence record, codelead.evidence/1, field by field - every kind of record, every outcome and stop reason, the chain of records, and real records.
kind: reference
area: reference
date: 2026-11-20
---

# Evidence record format

**Applies to:** CodeLead 0.1 and later

An [evidence record](../concepts/evidence-records.md) is the JSON file that CodeLead writes at the end of a run. CodeLead builds it from its own recorded state, never from a model's words. This page is the full format: every field, every kind of record, every outcome, and how records form a chain. The examples come from runs on the Basic Store sample.

## The file

| Fact | Value |
|---|---|
| Path | `.codelead/evidence/<id>.json`. `CODELEAD_KNOWLEDGE_DIR` moves the folder. |
| Name | `<id>` is `ev-`, the UTC time as 14 digits (`20261120091240`), a dash and 6 random characters. |
| Schema | `codelead.evidence/1`. A new optional field never changes the version. |
| Encoding | JSON, two spaces, UTF-8, one newline at the end. |
| Written by | Every run that ends, landed or not. Writing is advisory: a folder that cannot be written never fails the run. |
| In git | Yes. A record holds requests, file names, verdicts and exit codes. It holds no source code and no diff. |

## A record of a change that did not land

A request on the Basic Store sample: the patch applied, the build passed, and one test failed. The record stops with `validation_failed`:

<!-- described, not captured -->
```json
{
  "id": "ev-20261120091240-862yjb",
  "schema": "codelead.evidence/1",
  "created_at": "2026-11-20T09:12:40.000Z",
  "workspace": "<your project>",
  "who": { "name": "Sam", "hash": "4be1c0a9d27e" },
  "command": "implement",
  "request": "Show the stock of each product on the order form",
  "outcome": "stopped",
  "stop_reason": "validation_failed",
  "preflight": { "strategy": "architect", "confidence": 0.82, "clarifications": 2, "defaults_used": 0 },
  "retries_used": 1,
  "surface": {
    "files": [
      "WebApplication1/Controllers/OrdersController.cs",
      "WebApplication1/Views/Orders/Create.cshtml"
    ],
    "additions": 24,
    "removals": 3
  },
  "patch_check": { "status": "can_apply", "problems": [], "warning_count": 1 },
  "apply": { "status": "applied", "files_changed": 2 },
  "validation": {
    "status": "failed",
    "commands": [
      { "command": "msbuild sampleapp.slnx", "status": "passed", "exitCode": 0, "label": "build" },
      { "command": "vstest.console WebApplication1.Tests.dll", "status": "failed", "exitCode": 1, "label": "tests" }
    ]
  },
  "knowledge_ids": []
}
```

The [`/repair`](../chat-commands/repair.md) round that fixed it writes a second record that points back to the first with `follows`:

<!-- described, not captured -->
```json
{
  "id": "ev-20261120091502-4ziz8b",
  "schema": "codelead.evidence/1",
  "created_at": "2026-11-20T09:15:02.000Z",
  "workspace": "<your project>",
  "who": { "name": "Sam", "hash": "4be1c0a9d27e" },
  "command": "repair",
  "request": "Show the stock of each product on the order form",
  "outcome": "landed",
  "follows": "ev-20261120091240-862yjb",
  "repair_round": 1,
  "retries_used": 1,
  "repair_rounds": [ { "round": 1, "outcome": "patch_applied", "validation": "passed" } ],
  "surface": { "files": ["WebApplication1/Controllers/OrdersController.cs"], "additions": 4, "removals": 1 },
  "patch_check": { "status": "can_apply", "problems": [], "warning_count": 0 },
  "apply": { "status": "applied", "files_changed": 1 },
  "validation": {
    "status": "passed",
    "commands": [
      { "command": "msbuild sampleapp.slnx", "status": "passed", "exitCode": 0, "label": "build" },
      { "command": "vstest.console WebApplication1.Tests.dll", "status": "passed", "exitCode": 0, "label": "tests" }
    ]
  },
  "checkpoint_commit": "4f2a9c1",
  "checkpoint_branch": "codelead/show-the-stock-of-each-product-2026-11-20",
  "knowledge_ids": []
}
```

## The fields that every record has

| Field | Type | What it holds |
|---|---|---|
| `id` | string | The record's name, also its file name. |
| `schema` | string | Always `codelead.evidence/1`. |
| `created_at` | string | When the run ended, as an ISO time in UTC. |
| `workspace` | string | The folder of the repository that the run worked in. |
| `who` | object | Who ran it: `name`, the git `user.name` (else the account's name), and `hash`, 12 hex characters of the SHA-256 of the git `user.email` in lower case (else of the account's name). The email itself is never stored. Records of older releases may lack `who`. |
| `command` | string | The kind of record. See [the kinds](#the-kinds-of-record). |
| `request` | string | The request, word for word, cut at 2000 characters. |
| `outcome` | string | How the run ended. See [the outcomes](#outcomes). |
| `surface` | object | `files`: the files that the patch changed. `additions` and `removals`: the lines added and removed. An empty `files` list on a record that changed nothing. |
| `knowledge_ids` | list of strings | The entries of the [knowledge base](../concepts/knowledge-base.md) that the run created or confirmed. |

A field that does not apply is left out. A reader passes over a field that it does not know.

## The kinds of record

| `command` | Written by | Changes code |
|---|---|---|
| `implement` | [`/implement`](../chat-commands/implement.md), [`codelead implement`](../cli/codelead-implement.md), a pin or an in-place transform | Yes |
| `repair` | the [`/validate`](../chat-commands/validate.md) that follows a `/repair` | Yes |
| `judge-again` | `/modernize --transform T9 --judge-again`: a landed part of a rebuild judged again on the same tree. The rebuild's `--next` writes one too, by itself, for a part whose walk failed only on another part's page, accepted since. | No |
| `unapply` | the end of a session with a rebuild part applied and its gate not passed: the patch taken out again | It takes a change out |
| `reopen` | `/modernize --transform T9 --reopen <part> --reason <why>`: a parked part planned again | No |
| `ask` | the model ladder of a rebuild run, when a part's repair rung failed too: a person's turn | No |
| `intervene` | `/modernize --transform T9 --intervene <part>`: an agent loop on a strong model took a part over | When it lands |
| `map` | `codelead init`, [`codelead map`](../cli/codelead-map.md), [`/map`](../chat-commands/map.md), the end of a build | No |
| `rebuild-init` | [`codelead rebuild init`](../cli/codelead-rebuild-init.md) and `/modernize --rebuild-init` | No |
| `modernize` | every stage of the [modernization program](../modernization/the-program.md) that ends, and `codelead modernize skip` | No |

The `map`, `rebuild-init` and `modernize` records are not increments. [`/pr`](../chat-commands/pr.md), the board queue, the rebuild run and the chat's evidence lines pass over them.

## Outcomes

| `outcome` | Meaning |
|---|---|
| `landed` | The change is in the tree, and the [gate](../concepts/validation-and-the-gate.md) passed. |
| `landed_unvalidated` | A landing that no gate judged. Older releases wrote it. Readers still count it as a landing. |
| `stopped` | The run ended before a landing. `stop_reason` says why. |
| `cause_not_found` | The preflight said that it has not found the cause of a bug. No patch was made. `cause_reading` says why. |
| `not-landed` | A rebuild part whose gate had not passed when its session ended. The patch was taken out (`command: "unapply"`), or an intervention did not land. |
| `reopened` | A parked rebuild part, planned again (`command: "reopen"`). |
| `asked` | A rebuild part, or a stage of the program, that waits for a person. |
| `success` | A repository read, a rebuild made and ready to run, or a stage of the program that is done or skipped. |
| `failure` | A rebuild made and not ready to run, or a stage of the program that failed. `stop_reason` says why. |

### Stop reasons

| `stop_reason` | Where | Meaning |
|---|---|---|
| `preflight_failed` | `implement` | The preflight did not complete, for example because the model server did not answer. |
| `patch_failed` | `implement` | No patch came back from the model. |
| `patch_check_failed` | `implement` | The [patch check](../concepts/the-patch-check.md) refused the patch, after the retries. |
| `reply_cut` | `implement` | The model's reply was cut at its cap. `reply_cut` holds the counts. |
| `validation_failed` | `implement`, `repair` | The patch applied, and the gate failed. |
| `validation_declined` | `implement` | The patch applied, and you answered no to the validation. |
| `validation_not_run` | `implement` | The patch applied, and the validation did not run for another reason. |
| `pin_objection`, `transform_objection` | `implement` | The preflight of a pin or a transform said that the request cannot be done. |
| `acceptance_port_held` | `implement` | A rebuild's acceptance walk could not start: another process holds a server's port. |
| `no_repair_patch` | `repair` | The repair round produced no patch. |
| `repair_patch_invalid` | `repair` | The repair answer held no usable patch. |
| `repair_patch_check_failed` | `repair` | The patch check refused the repair patch. |
| `repair_cycle` | `repair` | The repair loop met a cycle twice and stopped for a person. |
| `gate_not_passed` | `unapply` | The part's gate had not passed when its session ended. |
| `intervention_validation-failed`, `intervention_not-accepted` | `intervene` | The intervention left a tree whose validation failed, or that the walk did not accept. |

For the words that you see with each stop, see [Messages and what to do](messages.md).

## The chain

A task can take several runs: the implement, then one or more repair rounds. Each later record names the one it follows:

```text
ev-20261120091240-862yjb   implement   stopped   validation_failed
        ↑ follows
ev-20261120091502-4ziz8b   repair      landed    repair_round 1
```

- **`follows`**: the id of the record that this one follows. A repair record follows the implement record or the previous repair. A `judge-again` record follows the landing that it judged. An `unapply` record follows the attempt's last record. An `intervene` record follows the record that it is about.
- **`repair_round`**: 1 for the first repair after the implement record, 2 for the next, and so on.
- **`repair_rounds`**: every round since the implement record, this one included, with what became of each.

Read a chain from its last record back along `follows`. The last record says how the task ended. `map`, `rebuild-init`, `modernize`, `reopen` and `ask` records follow no record.

## The fields of a change

| Field | Type | What it holds |
|---|---|---|
| `stage` | `pin` or `transform` | The modernization stage that the increment belongs to. |
| `card` | object | The [board](../pinboard/index.md) card that the run worked: `id`, and `board`, the board's folder relative to the workspace, such as `docs/codelead/board`. |
| `preflight` | object | `strategy`: the [strategy](../concepts/strategies.md) that the preflight used. `confidence`: from 0 to 1. `clarifications`: the questions that it settled. `defaults_used`: how many of them took their default. `assumptions`: for a pin or a transform, each `question`, the default `assumed`, and `why`. |
| `cause_reading` | object | When the preflight has not found a bug's cause: `found` (`false`), `statement`, `reasons`, `questions`, `confidence`, and `forced` when `--force` sent the run on. |
| `request_lint` | object | The [request lint](../chat-commands/lint.md) of the request: the files and tests named, the libraries assumed, the lines to add, and `linted_request` when the linted words differ. |
| `retries_used` | whole number | The patch retries after the check refused a patch. `0` when the first patch passed. |
| `patch_check` | object | `status`: `can_apply`, `cannot_apply`, `unsafe` or `invalid_patch`. `problems`: the blocking findings. `warning_count`: the notes. `layout_warnings` and `notes`: optional, at most 20 each. In a rebuild, `notes` also keeps the warnings and notes of the check of accepted parts, word for word. |
| `placement_problems` | list of strings | The anchored blocks of the answer that CodeLead could not place in their files. |
| `reply_cut` | object | A reply that was cut at its cap: each attempt with its reply and thinking tokens, its cap and its effort, and `second_attempt`, what CodeLead did next. |
| `apply` | object | `status` (`applied`) and `files_changed`. |
| `validation` | object | `status` (`passed` or `failed`) and `commands`. Each command has `command` (the command line), `status`, `exitCode`, and for a declared command its `label`. A command found in two scopes and run once lists them in `sources`. |
| `composed` | object | For a pin or a transform: `kind`, `subject` (the project or the scope), `level` (`memory` or `database`), `units`, and `command`, the line that composes it again. |
| `pins_database` | string | The database that a database pin's tests ran against, in words. Never a password. |
| `set_aside` | list | On a landed pin increment: the tests that CodeLead set aside so that the rest could land. Each has `test`, `file`, `wholeFile` (when the whole file was removed), `failure`, `reason`, `fact` (when the test's name anchors one) and `text`, the test as it was. See [Pins](../modernization/pins.md#a-pin-that-cannot-run-in-the-test-process). |
| `model_unreachable` | object | A model call that never got an answer: `call`, `role`, `provider` and `address`. |
| `conventions` | object | `conformance_warnings`, and `formatting`: the formatter that ran on apply, its `status` and its `files`. |
| `checkpoint_commit` | string | The commit that holds the landing, when checkpoint commits are on. |
| `checkpoint_branch` | string | The work branch of that commit, such as `codelead/…`. |

Each repair round of `repair_rounds`:

| Field | What it holds |
|---|---|
| `round` | 1, 2, and so on. |
| `outcome` | `patch_applied`, `no_patch` or `did_not_apply`. |
| `validation` | The gate's status after the round. |
| `retries_used` | The retries after the check refused the round's patch. |
| `repair_model` | When the repair role answered: `role` (`repair`), `provider`, `model`, and `served_model`, the model that the server says answered. |
| `cycle` | A cycle that the round met: `kind` (`content` or `failure`), `returned_to`, `file`, `first_error`, `escalated`, `stopped`. |

## The fields of a walk

| Field | What it holds |
|---|---|
| `acceptance` | A rebuild part's acceptance walk, beside the validation and never in its place: the `verdict`, the `round`, the part's own steps that passed, failed or were blocked, the regressions, the new and fixed findings, `max_rounds`, and `repair_offered`. A change that the walk did not accept has still landed. |
| `specs` | The walk of each spec named with `/implement --spec`: `how` the application ran (`base`, `run.json` or `rebuild`), each spec's verdict and steps, the `summary`, and the declared `seed` and `reset`. See [Specification files](spec-files.md). |

## The fields of a rebuild

| Field | On | What it holds |
|---|---|---|
| `rebuild_part` | an implement of a rebuild part | `id`, the part's id, and `composer`, the CodeLead build that composed it. |
| `ladder` | every record of a session that the model ladder runs | `rung` (`compose`, `repair` or `intervene`), `provider`, `model`, `effort`, and `stops_after` on a reopen. |
| `judged_again` | `judge-again` | `of`, the landing; `reason`, a person's words, or the run's own when it judged again by itself; `walker`; `target` (`plan`); `stepsChanged`; `note`; `against`. |
| `unapplied` | `unapply` | `decided` (`unattended` or `confirmed`), `repair_rounds`, `restored`, `removed`, `patches`, `checkpoint`, `not_restored`, `outside_surface`. |
| `reopened` | `reopen` | `reason`, and `attempts`: the records of the attempts that parked the part. |
| `asked` | `ask` | `of`, `on`, `reason`, `walk_words`, `files`, and `landed`: the board card or the plan line where the ask went. |
| `intervened` | `intervene` | The model and its effort, the turns, the tool calls, the walks, the usage and cost, the limits, the minutes, why it `stopped`, and the `gate` and its words. |
| `rebuild_init` | `rebuild-init` | `source`, `template`, `answers` (`framework`, `identity`, `database`), `installed`, the `journeys` counts by source, the `plan` (`parts`, `planned`, `dropped`, `blockers`) and the `files` written. |

## The fields of a repository read and of the program

The record that `codelead init` writes when it reads the repository into its map. This one is real, with the folder and the name replaced:

```json
{
  "id": "ev-20261120070542-2ss1d4",
  "schema": "codelead.evidence/1",
  "created_at": "2026-11-20T07:05:42.116Z",
  "workspace": "<your project>",
  "who": { "name": "Sam", "hash": "<12 hex characters>" },
  "command": "map",
  "request": "Read the repository into its map",
  "outcome": "success",
  "surface": { "files": [] },
  "repository_read": {
    "by": "init",
    "action": "written",
    "artifacts": ["docs/codelead/repository-map.json", "docs/codelead/repository-map.md"],
    "modules": 1,
    "files": 1,
    "files_scanned": 1,
    "written_at": "2026-11-20T07:05:42.112Z",
    "library_hash": "c25ac27deaba87179495fe018a19f58f2447845b1b0d218e52e2cd9f3cc8c76d",
    "encyclopedia": { "created": 6, "updated": 0, "downgraded": 0 }
  },
  "knowledge_ids": [
    "k-20261120070542-p242lg",
    "k-20261120070542-oo43sj",
    "k-20261120070542-2904td",
    "k-20261120070542-kjistd",
    "k-20261120070542-yx3j14",
    "k-20261120070542-5blsxr"
  ]
}
```

| Field of `repository_read` | What it holds |
|---|---|
| `by` | `init`, `map`, `session` or `build`. |
| `action` | `written`, `rewritten`, `refreshed` or `fresh`. |
| `artifacts` | The map's two files. |
| `modules`, `files`, `files_scanned` | What the map holds, and how many files the read scanned. |
| `head`, `dirty` | The commit that the read saw, and whether the tree had changes. |
| `written_at`, `library_hash` | The map's freshness stamp. |
| `encyclopedia` | The entries of the knowledge base that the map seeded, or `encyclopedia_error`. |

The record of a stage of the [modernization program](../modernization/the-program.md). Here the person skipped the Pin stage with [`codelead modernize skip`](../cli/codelead-modernize-skip.md):

<!-- described, not captured -->
```json
{
  "id": "ev-20261120101500-7mraux",
  "schema": "codelead.evidence/1",
  "created_at": "2026-11-20T10:15:00.000Z",
  "workspace": "<your project>",
  "who": { "name": "Sam", "hash": "4be1c0a9d27e" },
  "command": "modernize",
  "request": "Modernization program: Pin the contracts (codelead modernize skip pins)",
  "outcome": "success",
  "surface": { "files": [] },
  "modernize": {
    "stage": "pins",
    "status": "skipped",
    "mode": "approve",
    "summary": "Skipped by the person: The pins wait for a copy of the database; the team goes on with the transforms first.",
    "outputs": [],
    "details": {
      "skippedBy": "person",
      "reason": "The pins wait for a copy of the database; the team goes on with the transforms first.",
      "was": "failed",
      "problem": {
        "title": "Pin the contracts: stopped",
        "reason": "stopped by the person",
        "fix": "Run the stage again when you are ready; what it finished before the stop is kept",
        "retry": "codelead modernize run pins"
      }
    }
  },
  "knowledge_ids": []
}
```

| Field of `modernize` | What it holds |
|---|---|
| `stage` | The stage's id. See [Modernization state file](modernization-state-file.md#stages). |
| `status` | `done`, `failed`, `waiting` or `skipped`. |
| `mode` | `approve` or `unattended`. |
| `summary` | What the stage did, in one sentence. |
| `outputs` | The files that it wrote. |
| `problem` | For a stage that failed or waits: `title`, `reason`, `fix` and `retry`. |
| `details` | The stage's own facts. A skip by a person holds `skippedBy`, `reason`, `was` (the status before) and the `problem` it had. |

The outcome follows the status: `success` for done or skipped, `failure` for failed, `asked` for a stage that waits.

## Who reads records

- [`/pr`](../chat-commands/pr.md) and [`codelead pr`](../cli/codelead-pr.md) make the body of a pull request from the last landed record.
- [`codelead implement`](../cli/codelead-implement.md) writes the record's id on the card that it worked.
- The rebuild's plan reads the records of each part to know its state.
- The licence's seat count reads `who.hash` from the records of the last 30 days. See [Licence and plans](../concepts/licence-and-plans.md).
- The [audit pack](../modernization/audit-pack.md) gathers them. The extension's [Knowledge base view](../extension/knowledge-base-view.md) lists them.

## See also

- [Evidence records](../concepts/evidence-records.md)
- [Results and stops](../concepts/results-and-stops.md)
- [Repair](../concepts/repair.md)
- [`/implement`](../chat-commands/implement.md), [`/repair`](../chat-commands/repair.md) and [`/validate`](../chat-commands/validate.md)
- [`codelead pr`](../cli/codelead-pr.md)
- [Files and folders](files-and-folders.md)
