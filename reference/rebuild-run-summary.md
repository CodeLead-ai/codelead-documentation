---
title: Rebuild run summary
description: The format of .codelead/rebuild-run-<stamp>.json and .md, codelead.rebuild-run/1 - what an unattended codelead rebuild run did, part by part, session by session, with its cost and why it stopped.
kind: reference
area: reference
date: 2026-11-20
---

# Rebuild run summary

**Applies to:** CodeLead 0.1 and later

An unattended [`codelead rebuild run`](../cli/codelead-rebuild-run.md) builds the parts of a [rebuild](../modernization/rebuild.md) one session after another. After every session, and at the end, it writes a summary. The summary says what each part came to, and lists every session. It gives the cost on each rung of the model ladder, and the parts that it leaves for a person. The summary is a JSON file and a markdown file with the same stamp.

## The files

| Fact | Value |
|---|---|
| Paths | `.codelead/rebuild-run-<stamp>.json` and `.codelead/rebuild-run-<stamp>.md`, in the rebuild. |
| Stamp | The run's start time in UTC, such as `20261120T060000Z`. |
| Schema | `codelead.rebuild-run/1`, in the JSON file. |
| Beside them | `.codelead/rebuild-run-<stamp>/`: the console of each session. `.codelead/run-journal.json` and `.codelead/run-report.html` follow the parts while the run goes. |
| Written by | `codelead rebuild run`, and the program's Build and verify stage, which runs it. A dry run writes none. |
| In git | No, ignored. `codelead rebuild run` adds the files to `.codelead/.gitignore`. |

The JSON file says `"status": "running"` while the run goes, and `"ended"` at the end. Read it during a run to see how far the run is.

## The markdown summary

<!-- described, not captured -->
```markdown
# Rebuild run 2026-11-20 06:00Z

Lanes taken, in order: foundations, clients.

## The numbers

| | |
| Parts taken | 6 |
| Accepted at the first walk | 3 |
| Accepted after repair rounds | 1 |
| Accepted on the repair rung | 1 |
| Accepted by the intervention | 0 |
| Landed, no walk judged it | 0 |
| **Asks: left for a person** | **1** |
| Not settled when the run stopped | 0 |
| Repair rounds | 4 |
| Interventions (accepted) | 0 (0) |
| Sessions | 9 |

## Asks

- `clients.edit:screen` (ladder-ask, board, record `ev-20261120081530-t4n2kx`): the repair rung did not get it accepted.

## Accepted parts whose contract does not hold

A part its walk accepted whose endpoint, screen's route or tests are no longer in the code: accepted no more, planned again to be rebuilt by the next run. The part whose landing removed it is named.

None.

## Cost per rung

| Rung | Calls | Priced | Prompt tokens | Completion tokens | Cost |
…

## Parts

| Part | Outcome | Sessions | Repair rounds | Interventions | Minutes | Cost | Compose / repair / intervene |
…

## Sessions

| # | Part | What | Rung | Minutes | Cost | Records | Console |
…

## Settings

- Ladder: …
- Acceptance: …
- The walk's passwords: …
- The day reset: …
- Repair rounds per session: at most 8. Clean tree required: yes.
- Caps: 12 parts, $20, 480 minutes.
- CodeLead's switches for the run: `CODELEAD_SKIP_PREFLIGHT=1`.
```

The sections appear in this order. A section of lanes that wait on other lanes appears only when there is such a wait. The licence's warnings, when the run had any, come first.

The numbers table has two more rows when the run had such a part:

- `Accepted when judged again by the run (the page its step acts on accepted since)`: the count of `accepted-judged-again`.
- `Accepted on what regresses, its steps waiting on another part's page`: the count of `waits-on-page`.

The section of accepted parts whose contract does not hold is always there, with `None.` when every accepted part holds. Each line names the part, the record that accepted it, and what is gone. For example:

<!-- described, not captured -->
```markdown
- `clients.create-get:endpoint` (accepted by `<RECORD>`): its endpoint GET /api/clients/create is no longer in the code: the part clients.create-post:endpoint removed it in <RECORD> (checkpoint <COMMIT>).
```

## The JSON summary

| Key | Type | What it holds |
|---|---|---|
| `schema` | string | `codelead.rebuild-run/1`. |
| `target` | string | The name of the rebuild's folder. Never a path of the machine. |
| `targetFromSource` | string | The rebuild's folder as seen from the old application's folder, for example `../webapp-rebuild`. Present when the rebuild records where the old application is. |
| `status` | `running` or `ended` | Whether the run still goes. |
| `startedAt`, `endedAt` | string | ISO times in UTC. |
| `minutes` | number | The run's wall clock so far. |
| `stopped` | object | Why the run stopped: `reason` and `words`. See [Stop reasons](#stop-reasons). |
| `options` | object | The command's options: `lane`, `parts`, `until`, `maxParts`, `maxCostUsd`, `maxMinutes`, `scope`. |
| `lanes` | list | The lanes that the run took, in order: `lane`, `parts`, `claimed`, `released`, `branch`, `from`. |
| `laneWhy` | string | Why the run took those lanes. |
| `passedBy` | list | The lanes that `--lane all` passed by, because another clone's run claimed them: `lane`, `why`. |
| `sync` | object | Whether the lanes' claims synced with the remote, its name, and why. |
| `waits` | list | Parts that still wait on a part of another lane: `part`, and `on` with each part, lane and state. |
| `contractsBroken` | list | Every part that an evidence record accepted and whose endpoint, screen's route or tests are no longer in the code, as the run's last plan read it: `part`, `record` (the record that accepted it), `words`, and `by`, the part whose landing removed it, when the commits tell. Empty when every accepted part holds. The plan plans each one again. |
| `settings` | object | `ladder`, `repairsPerSession`, `cleanTree`, `env`, `reset`, `passwords`, `acceptance`: the run's settings, in words. Never a password. |
| `totals` | object | The counts of the numbers table. |
| `cost`, `costByRung` | object | The cost of all calls, and of each rung: `compose`, `repair`, `intervene`. |
| `asks` | list | Every part that the run leaves for a person. |
| `parts` | list | Each part that the run took. |
| `sessions` | list | Each session. |
| `files` | object | `json`, `markdown`, `runJournal`, and `runReport` and `rebuildReport` when they were written. |
| `licenceWarnings` | list of strings | The licence's warnings at the start of the run, when there were any. |

### totals

`partsTaken`, `acceptedAtFirstWalk`, `acceptedAfterRepair`, `acceptedOnRepairRung`, `acceptedByIntervention`, `landedNotWalked`, `asks`, `unsettled`, `repairRounds`, `interventions`, `interventionsAccepted` and `sessions`. Each is a whole number. Two more keys are there only when the count is more than 0: `acceptedJudgedAgain` and `waitsOnPages`.

`asks` is the honest number of a run: every part that a person must look at.

### A cost

| Key | What it holds |
|---|---|
| `calls` | The model calls. |
| `pricedCalls` | The calls whose provider gives a price. A local model's calls are not priced. |
| `promptTokens`, `completionTokens` | The tokens, as the provider counted them. |
| `costUsd` | The cost in dollars of the priced calls. |

### A part

| Key | What it holds |
|---|---|
| `id`, `title`, `kind` | The part, as the plan names it, such as `clients.index:endpoint`. |
| `outcome` | What the part came to. See below. |
| `words` | The outcome in words. |
| `state` | The part's state in the plan after the run: `planned`, `accepted`, `parked`, and so on. |
| `sessions`, `repairRounds`, `interventions`, `minutes` | The counts for the part. |
| `cost`, `costByRung` | Its cost, in all and by rung. |
| `records` | The [evidence records](evidence-record-format.md) of the part, in order. |

The outcomes of a part:

| `outcome` | Meaning |
|---|---|
| `accepted-first-walk` | It landed, and the first acceptance walk accepted it. |
| `accepted-after-repair` | Repair rounds made the walk accept it. |
| `accepted-on-repair-rung` | The repair rung of the ladder made the walk accept it. |
| `accepted-by-intervention` | The intervention rung made the walk accept it. |
| `accepted-judged-again` | The run judged it again by itself, with no model call, and accepted it. Its walk had failed only on a step whose control is on another part's page, and that part was accepted since. |
| `landed-not-walked` | It landed, and no walk judged it. |
| `waits-on-page` | It landed, and is accepted on what regresses. Its own steps wait on the page that another part builds. The run judges it again once that part is accepted. It is not an ask. |
| `asked` | The ladder asked a person. |
| `refused` | The composition was refused. The run passes the part by for the rest of the run. |
| `repairs-left` | It landed, and its repair rounds were left when the session ended. |
| `parked` | Its attempts ran out, and the plan parked it. |
| `not-accepted` | The walk did not accept it, with the ladder off. |
| `unsettled` | It had not settled when the run stopped. |

### An ask

| Key | What it holds |
|---|---|
| `part` | The part. |
| `kind` | `ladder-ask`, `refused`, `repairs-left`, `parked` or `not-accepted`. |
| `where` | Where the ask is: a card on the board, or a line of the plan. |
| `why` | Why, in words. |
| `record` | The evidence record of the ask, when there is one. |

### A session

| Key | What it holds |
|---|---|
| `index` | 1, 2, and so on. |
| `part` | The part that the session worked. |
| `action` | `compose`, `judge-again`, `reopen`, `intervene` or `asks`. |
| `rung` | `compose`, `repair` or `intervene`. |
| `startedAt`, `endedAt`, `minutes` | When, and how long. |
| `cost` | The session's cost. |
| `records` | The evidence records that the session wrote. |
| `refused` | The queue command's refusal, in its words, when it failed. |
| `log` | The session's console, as a person would have read it. |
| `sessionDir` | The session log's folder. |

## Stop reasons

| `stopped.reason` | Meaning | Exit code |
|---|---|---|
| `done` | The queue has no part left. | 0 |
| `until` | The part that `--until` names is settled. | 0 |
| `max-parts`, `max-cost`, `max-minutes` | A cap of the command or of `run.unattended`. | 0 |
| `stop-file` | You wrote `.codelead/STOP`. | 0 |
| `lane-claimed` | Another clone claimed the lane meanwhile. | 0 or 1 |
| `interrupted` | You pressed Ctrl+C. The session in flight finished first. | 130 |
| `blocked` | The plan is blocked. | 1 |
| `port-held` | Another process holds a server's port. | 1 |
| `no-progress` | A session made no progress. | 1 |
| `plan-refused`, `branch-refused` | The plan, or the lane's branch, was refused. | 1 |

The caps stop the run between sessions: the session in flight always finishes. See [Run profile](run-profile.md#rununattended) and [Exit codes](exit-codes.md).

## See also

- [`codelead rebuild run`](../cli/codelead-rebuild-run.md)
- [The rebuild](../modernization/rebuild.md)
- [Run profile](run-profile.md)
- [Unattended runs and the run lock](../concepts/unattended-runs.md)
- [Evidence record format](evidence-record-format.md)
- [Files and folders](files-and-folders.md)
