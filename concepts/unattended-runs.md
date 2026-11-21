---
title: Unattended runs and the run lock
description: Learn how CodeLead works without a person, who answers its questions, how a run is capped and stopped, and why one clone holds one run at a time.
kind: concept
area: concepts
date: 2026-11-20
---

# Unattended runs and the run lock

An unattended run is CodeLead working while nobody answers its questions: overnight, in a scheduled job, or while you do other work. CodeLead answers each question by a fixed policy, records every answer, and stops at the caps (the limits) that you set. The **run lock** makes sure that only one such run works in a clone (one copy of the repository) at a time.

**In short**
- Three commands run unattended: [`codelead implement`](../cli/codelead-implement.md), [`codelead rebuild run`](../cli/codelead-rebuild-run.md) and [`codelead modernize`](../cli/codelead-modernize.md) in its unattended mode.
- A fixed policy answers the questions. It says yes to apply, to validate and to run commands. It says no to a manual pass.
- A run stops at a cap, at a stop file, or on Ctrl+C. The card or the part that is running finishes first.
- A run takes the lock `.codelead/run.lock`. A second run in the same clone is refused, with the owner's name.

## Why it exists

Some work is long and repetitive: twenty cards on a board, eighty parts of a rebuild. A person cannot press Enter all night. An unattended run does that work, and it keeps every rule of an attended one. Each change still passes the [patch check](the-patch-check.md) and [the gate](validation-and-the-gate.md), and writes an [evidence record](evidence-records.md). A person reads the result in the morning.

Two runs in one clone would overwrite each other's files: the run journal, the live report, the sessions. The lock prevents this.

## How it works

### Who answers

Nobody types in an unattended run. CodeLead answers each question itself:

- **Yes** to "Apply this patch?", "Run validation now?" and "Run these commands?".
- **No** to "Mark validation as manually passed?". No person checked it, so CodeLead never says that a person did.
- **The default** of each clarifying question of the preflight.
- **Nothing** to a step that is hard to undo. CodeLead asks three times, gets no answer, and declines.

A composed request (a pin or a transform of a modernization) asks no questions at all. It takes each default and shows the list once as "CodeLead assumed". The evidence record keeps every assumption. See [Prompts and recommended answers](prompts-and-recommended-answers.md).

### The runners

| Command | What it works | Its caps |
|---|---|---|
| `codelead implement --card <id>` or `--queue` | The board's cards tagged `for:codelead`, one session for each card, with up to 2 repair rounds each | `--max-cards`, `--max-minutes`, `--until`, `--repair-rounds` |
| `codelead rebuild run` | The parts of a rebuild's plan: each composed, validated, walked and judged | `--max-parts`, `--max-cost-usd`, `--max-minutes`, `--until`, or `run.unattended` in the rebuild's manifest |
| `codelead modernize start --unattended`, then `next` | The stages of a modernization program, with the transforms' approvals answered yes | The caps of each stage, such as the pins' increments |

### How a run stops

- At its end: no card is left, or the queue has nothing left to do.
- At a cap, such as a number of cards, a time, a cost or a number of parts.
- When a stop file appears: `.codelead/STOP` for `implement` and `rebuild run`, `.codelead/modernization.stop` for the program. [`codelead modernize stop`](../cli/codelead-modernize-stop.md) writes the second one.
- On Ctrl+C. The card or the part that is running finishes first. For `rebuild run`, a second Ctrl+C stops at once.

### Check first

`codelead implement` and `codelead rebuild run` have a dry run, `--dry-run`, that checks and changes nothing. Both dry runs also check that the model server answers, and refuse with exit code 2 when it does not. Run a dry run before every real run. A dry run of `codelead implement` with an empty queue:

```console
codelead implement --queue --dry-run
```

<!-- described, not captured -->
```output
Dry run of codelead implement --queue: the checks passed; nothing is run or written.
Board: docs/codelead/board.
Model: the default role, the preflight role, the fast strategy, the repair role, the clarify strategy, the architect strategy, the deep strategy: lmstudio (the server's loaded model).
No card is tagged for:codelead outside In progress, For review and Done: a run would end at once.
Each card: one session driven with /implement --card <id>, then up to 2 repair rounds, then /exit; the session writes the card's trail as codelead.
Caps: no caps; up to 2 repair rounds per card; the stop file .codelead/STOP and Ctrl+C stop the run after the card in flight.
Lock: free: a run started now takes it.
```

### The run lock

A run writes `.codelead/run.lock`. The file names the machine, the process, the person, the command and the start time. For a rebuild, it also names the lane: the area of the rebuild's plan that the run works. The run writes a heartbeat in the file every minute. A second run in the same clone is refused, in words like these:

<!-- described, not captured -->
```output
This clone is in use: codelead rebuild run --lane reports-admin by <your name> on <machine> (pid 4242), started 2026-11-20 01:12Z, its last heartbeat 1 minute ago. One run per clone: wait for it to end, or stop it (a rebuild run stops after its part in flight on .codelead/STOP). A lock with no heartbeat for 10 minutes is taken over.
```

A lock is **stale** when it has had no heartbeat for 10 minutes, or when its process on this machine has ended. The next run takes it over, and says so in its run journal. The lock is removed at a clean end, and on Ctrl+C.

> [!NOTE]
> **For team leads:** git keeps two people's clones apart. The lock keeps two runs apart in one clone. Two clones can run two lanes of one rebuild at the same time.

## What it is not

- **Not a way around consent**. Nobody can answer a consent question. So a run whose model is remote is refused, unless `CODELEAD_ALLOW_REMOTE_MODELS=1` is set or the configuration allows remote models with no question. See [Privacy and remote models](privacy-and-remote-models.md).
- **Not a lock for reading**. The plan, the chat, [`codelead doctor`](../cli/codelead-doctor.md) and the reports never take the lock.
- **Not a lock across machines**. The run lock is in the clone. A walk database has its own lock on the machine.

## Details

- **The walk database lock**. The walk database is the database that the walks of a rebuild use. When the rebuild's run profile names it, the run also locks that database on this machine, in `~/.codelead/locks/walk-<key>.lock`. `CODELEAD_LOCKS_DIR` moves the folder. The key is a hash: no connection string is written. Two walks on one database at once would corrupt each other's evidence.
- **Who takes the lock**. `codelead rebuild run`, `codelead implement`, the modernization program, and the interactive `/modernize --transform T9 --next`, `--judge-again` and `--intervene` for the rest of their session. A session that a run drives belongs to the run and takes nothing.
- **Remote models**. A run is also refused when the configuration asks before every remote call, because nobody answers. Set `CODELEAD_ALLOW_REMOTE_MODELS=1`, or `privacy.confirmBeforeSendingRepoContextToRemote` to `false`.
- **The terms**. A packaged release asks once for a typed `yes` to its evaluation terms. Set `CODELEAD_ACCEPT_TERMS=yes` for a scheduled job.
- **Sleep**. On macOS, a run keeps the machine awake with `caffeinate -i -s`. A Mac that sleeps makes a run very slow.
- **Evidence**. Each change of the run writes its evidence record. Each card's thread says how the card ended, with the id of its evidence record.

## Related content

- [`codelead implement`](../cli/codelead-implement.md)
- [`codelead rebuild run`](../cli/codelead-rebuild-run.md) and [`codelead rebuild`](../cli/codelead-rebuild.md)
- [`codelead modernize`](../cli/codelead-modernize.md), [`codelead modernize start`](../cli/codelead-modernize-start.md) and [`codelead modernize stop`](../cli/codelead-modernize-stop.md)
- [Prompts and recommended answers](prompts-and-recommended-answers.md)
- [Privacy and remote models](privacy-and-remote-models.md)
- [Stop and resume work](../how-to/stop-and-resume.md)
- [Work with agents](../pinboard/work-with-agents.md)
- [Run profile](../reference/run-profile.md)
- [Glossary](glossary.md)
