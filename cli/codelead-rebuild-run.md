---
title: codelead rebuild run
description: Run a rebuild's queue without a person, part by part, with caps on parts, cost and time, and a dry run that checks first.
kind: reference
area: cli
date: 2026-11-20
---

# codelead rebuild run

**Applies to:** CodeLead 0.1 and later

`codelead rebuild run` runs the rebuild's plan without a person. Each part is composed, validated, walked and judged. A part that CodeLead cannot finish is parked for a person, and the run continues. Run it in the rebuild repository that [`codelead rebuild init`](codelead-rebuild-init.md) made.

## Name

`codelead rebuild run` - Runs the rebuild's queue unattended to its end.

## Synopsis

```console
codelead rebuild run [--target <REPO>] [--lane <AREA>] [--parts <IDS>] [--scope <PREFIXES>]
                     [--until <ID>] [--max-parts <N>] [--max-cost-usd <X>] [--max-minutes <M>]
                     [--dry-run] [--json]
codelead rebuild run -h|--help
```

## Description

The rebuild's plan is a queue of **parts**: an endpoint of the new API, a screen of the new web application, a prerequisite. See [`codelead rebuild`](codelead-rebuild.md). For each part whose needs are met, the run:

1. **composes** it: a session asks the model for the part's code, as [`/implement`](../chat-commands/implement.md) does;
2. **validates** it with the declared gate (the build and the tests);
3. **walks** it: [the walk](../modernization/walk-and-parity.md) follows the part's journeys on the running application;
4. **judges** it: the part is accepted, repaired, or parked.

When a check fails, the **model ladder** decides what comes next. First a repair round, then an intervention by a stronger model, as the run profile allows. A part that the ladder gives up on is **parked** for a person, with the reason. The run continues with the next part.

> [!NOTE]
> **New to this?** *Unattended* means that nobody answers questions during the run. CodeLead answers "Apply this patch?" itself, and records every answer in the evidence records. A person reads the result after the run.

### Lanes

A run claims a **lane** (an area of the plan, or the foundations), never a single part. Two clones can run two lanes at the same time. A lane that another clone's run claims is passed by. One run works in a clone at a time, with one walk database.

Without `--lane`, the run takes the foundations while they have work left, then the first lane that nobody claims. `--lane all` takes the foundations, then every lane in the plan's order.

### When the run stops

The run ends at the queue's end, at `--until`, at a cap, when `.codelead/STOP` appears, or on Ctrl+C. In every case, the part that is running finishes first. A second Ctrl+C stops at once.

### Check first

`--dry-run` checks the run profile and lists what would run. It checks three things: each role's model server answers with its list of models, the walk has its passwords, and the plan has no blocker. Nothing is run or written. Run it before every real run.

> [!WARNING]
> A real run calls models for each part. With a paid provider, set a cost cap: `--max-cost-usd`, or `run.unattended.maxCostUsd` in the manifest.

> [!TIP]
> **For experienced users:** the run profile is the `run` block of `docs/codelead/rebuild/rebuild.json`. It holds the ladder, the caps (`run.unattended`), the day reset and the source of the walk's passwords. See [Rebuild manifest](../reference/rebuild-manifest.md). On macOS, the run keeps the machine awake with `caffeinate -i -s`.

## Options

- **`--target <REPO>`**

  The rebuild repository. The default is the current folder.

- **`--lane <AREA>`**

  The lane that the run claims and takes: an area of the plan, `foundations`, or `all`. With `all`, the run takes the foundations, then every lane in the plan's assignment order, one after another. A lane that another clone's run claims is passed by. The default is the foundations while they have work left, then the first lane that nobody claims.

- **`--parts <IDS>`**

  The parts that the run may take, separated by commas, inside its lane. The default is every part of the lane.

- **`--until <ID>`**

  Stops once this part is settled (accepted, or parked for a person), or when the queue has nothing left to do for it.

- **`--max-parts <N>`**

  The most parts that the run takes. The default is `run.unattended.maxParts`, else no cap.

- **`--max-cost-usd <X>`**

  The most that the run's model calls may cost, in dollars. The default is `run.unattended.maxCostUsd`, else no cap.

- **`--max-minutes <M>`**

  The longest that the run may take, in minutes. The default is `run.unattended.maxMinutes`, else no cap.

- **`--scope <PREFIXES>`**

  The plan's scope: feature ids or prefixes, separated by commas. It is the same scope that `/modernize --next --scope` takes.

- **`--dry-run`**

  Checks the run profile and lists what would run. Nothing is run or written.

- **`--json`**

  Prints one JSON object for each event, in place of words.

## Examples

- Check the run before you start it. Here the plan still has blockers, and the model server does not answer:

  ```console
  codelead rebuild run --dry-run
  ```

  ```output
  04:16:32Z The run is refused; nothing was run:
  - The plan has 36 blockers: nothing is composed until they are resolved.
  - the extraction gate: unexplained-orphan: entry-point account.confirm-email is neither live nor dead: …
  …
  - The model of the default role, the preflight role, the architect role, the repair role does not answer: lmstudio (http://127.0.0.1:9/v1) answered no list of its models (Could not connect to lmstudio at http://127.0.0.1:9/v1. If this is LM Studio, make sure it is open and the Local Server is running.).
  ```

- A dry run that passes. This output is described from CodeLead's code and tests:

  ```console
  codelead rebuild run --dry-run --lane products --max-parts 5
  ```

  <!-- described, not captured -->
  ```output
  Dry run: the checks passed; nothing is run or written.
  Model: the default role, the preflight role, the architect role: lmstudio qwen/qwen3.6-35b-a3b.
  Ladder: …
  Acceptance: …; passwords from …; day reset: node scripts/reset-sample-seed.mjs; 8 repair rounds at most per session.
  Lane: products; 14 parts in products.
  First: …
  Then, in the plan's order as their needs are met (5 parts of 14: --max-parts cuts the rest):
    1. products.create-get:screen: …
  …
  The run would stop …
  ```

- Run the products lane, with a cap of $5:

  ```console
  codelead rebuild run --lane products --max-cost-usd 5
  ```

  <!-- described, not captured -->
  ```output
  …
  The run stops: $5.10 spent, of the $5.00 this run may spend: the caps are reached, and the next part waits for the next run.
  …
  ```

  The run prints one line for each event, then where the run summary is: `.codelead/rebuild-run-<stamp>.md` and `.json`.

- Stop a run from another terminal. The part that is running finishes first:

  ```console
  touch .codelead/STOP
  ```

  <!-- described, not captured -->
  ```output
  The run stops: the stop file .codelead/STOP was found (and taken away): the run stops after the part in flight.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The run ended: the queue's end, `--until`, a cap, or the stop file. A dry run whose checks passed. |
| `1` | The run was refused, or stopped on a blocker, a held port or no progress. |
| `130` | Ctrl+C. The part that was running finished first. |

## Files

| File | What the run does with it |
|---|---|
| `docs/codelead/rebuild/rebuild.json` | Reads the manifest and the run profile. |
| `docs/codelead/rebuild/rebuild-plan.json`, `.md` | Reads the plan, and writes the claims and the parts' states. |
| `.codelead/rebuild-run-<stamp>.json`, `.md` | Writes the run summary: each part, its outcome, its cost. |
| `.codelead/STOP` | Stops the run after the part that is running when this file appears. The run removes it. |
| `.codelead/evidence/ev-*.json` | Each part's session writes its evidence records. |
| `docs/codelead/parity/` | The walks write their reports here. |
| The rebuild's source files | The parts' patches change them. |

## See also

- [The rebuild](../modernization/rebuild.md)
- [`codelead rebuild init`](codelead-rebuild-init.md): the step before.
- [`codelead rebuild`](codelead-rebuild.md): print the plan and its lanes.
- [`codelead rebuild-report`](codelead-rebuild-report.md): the progress report of the run.
- [`codelead parity`](codelead-parity.md): the gate for calling the rebuild finished.
- [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md)
- [Stop and resume work](../how-to/stop-and-resume.md)
