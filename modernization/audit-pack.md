---
title: The audit pack
description: What the audit pack of a modernization holds, who reads it, and how to check that nothing in it was changed.
kind: concept
area: modernization
date: 2026-11-20
---

# The audit pack

The audit pack is one folder that proves what a modernization did. It holds every change that landed, with its request, its evidence record, its commit and its attempts. It also holds the gate, the survey before and after, the security findings, and every session with its secrets removed. CodeLead writes it in the last stage, Hand over.

**In short**
- You know what the audit pack holds, layer by layer, and which file to read first.
- You know who reads it, and how they check that no file in it was changed.
- You know how to run it, and how to keep secrets out of it.

## Why it exists

The people who review a modernization rarely watch it happen. A team lead, a security reviewer, an auditor or the application's owner checks the work afterwards. They do it without CodeLead, and without the person who ran it. The audit pack answers their questions from files: what changed, what proved each change, what was refused and why, and what is left.

## How it works

[`/modernize --audit-pack`](../chat-commands/modernize.md#--audit-pack) builds the pack in `docs/codelead/audit-pack/`. The program's Hand over stage runs the same action. It reads the repository's evidence records, its git history, the dossier, the declared gate and the sessions. It copies what it needs, removes secrets, and writes a manifest.

On the Basic Store sample, after the dossier and with no landed change yet:

```console
/modernize --audit-pack
```

```output
Audit pack
Audit pack: docs/codelead/audit-pack/ (47 files, 0.2 MB)
Increments: 0 landed; sessions: 1; stops: 0. Before `e41a806` → after `ab1dad7`.
Note: No landed evidence record; layer 2 is empty.

Read README.md first; manifest.json lists every file with its SHA-256.
```

After a full modernization, the pack holds one folder for each landed increment: each pin, each transform.

### The layers

| Folder | What it holds |
|---|---|
| `README.md` | The summary. It opens with the stages that a person skipped: who, when and why. Then the rebuild as it stands, when the program's Rebuild stage is done or skipped. Then what was done, the increments, the metrics before and after, the criteria and where each proof is. Read it first. |
| `0-dossier/` | The survey, the intake and the options that the work started from. |
| `1-program/` | The program's state file, and the evidence record of each skipped stage. When the Rebuild stage is done or skipped, also the rebuild's plan as it stands, `rebuild-plan.json`, and the evidence record of the person's `rebuild=enough` answer. |
| `2-increments/<nn>-<stage>-<id>/` | One folder for each landed increment: `request.txt`, `evidence-record.json`, `commit.txt` and `diff.patch` when a commit exists, and `attempts.md` with every record of the same request. |
| `3-baselines/` | The declared gate (`validation-gate.json`), the knowledge store, the branch log, and `metrics.json`: the survey run on the tree before and the tree after. |
| `4-sessions/<session>/` | The full session log of each session, with every prompt and response, and `INDEX.md` with the reason of each stop. |
| `5-security/` | The security scan on both trees, and each finding's state: fixed, open, waived or new (`DISPOSITIONS.md`). |
| `manifest.json` | Every file of the pack with its size and its SHA-256. |

### The rebuild as it stands

A person can end the Rebuild stage with parts left: `codelead modernize answer rebuild=enough` goes on with what is built. The pack then says what is not built. When the program's Rebuild stage is done or skipped, `README.md` holds the section `## The rebuild as it stands`. CodeLead reads it from the rebuild's plan and the stage's evidence record, with no model call. It holds:

- **How the stage ended.** After an answer: its date, what the stage waited for, and the answer's record. Or the skip, or the stage's own end.
- **How many parts are accepted**, of how many, and how many the plan dropped.
- **A table of each part that is not accepted**: its state, and why, in the plan's own words. A state is, for example, `parked for a person`, `planned, waiting` or `planned and never taken`.
- **The plan's blockers**, and its notes to a person.

The plan is copied as `1-program/rebuild-plan.json`, with its secrets removed. In this example from CodeLead's tests, the person answered `rebuild=enough` while a prerequisite was parked:

<!-- described, not captured -->
```output
## The rebuild as it stands

The rebuild stage ended by the person's answer `codelead modernize answer rebuild=enough` at 2026-11-24T08:00:…Z: go on with what is built. It waited: The rebuild waits for you: 4 parts of 6 are not accepted yet: identity-seam is parked (the answer's record: `1-program/ev-….json`).

From the rebuild's plan `../shop-rebuild/docs/codelead/rebuild/rebuild-plan.json`, copied as `1-program/rebuild-plan.json`: **2 of 6 parts accepted** (1 dropped by the plan); 3 parts not accepted:

| Part | State | Why, in the plan's words |
|---|---|---|
| identity-seam | parked for a person | parked after 2 attempts that did not land (the last: ev-seam-2, the patch check refused the same change twice): a later rule or a person reopens it |
| orders.index:screen | planned, waiting | no evidence record shows it landed; waits on identity-seam (parked for a person) |
| orders.edit:screen | planned and never taken | no evidence record shows it landed |
```

A rebuild with nothing left ends the line with `every part is accepted, satisfied, judged without a walk or dropped: nothing is left`. When CodeLead cannot read the plan, the section says so, with the plan's path relative to the old application's folder.

### Before and after

`README.md` compares the two trees with the same survey. On the Basic Store sample, the table reads:

```output
| Metric | Before `e41a806` | After `ab1dad7` |
|---|---|---|
| Projects SDK-style | 0 of 0 | 0 of 1 |
| Target frameworks | - | v4.5 |
| Runtimes out of support | 0 | 1 |
| Source files / lines / test files | 0 / 0 / 0 | 74 / 5,079 / 0 |
```

Here the "before" commit is the repository's first commit, which was empty, because nothing had landed yet. Give the commit before the work started with `--since`, so that "before" is the old application.

## Who reads it

- **A team lead** reads `README.md`: what landed, how many sessions, what stopped and why.
- **A reviewer** opens an increment's folder: the request, the diff, the validation commands and their exit codes in the evidence record.
- **A security reviewer** reads `5-security/DISPOSITIONS.md`: each finding fixed, open or waived, with the reason of each waiver.
- **An auditor** checks the pack itself: every file against its SHA-256 in `manifest.json`.

> [!NOTE]
> **For team leads:** the pack stands alone. Every number in `README.md` comes from files in the layers, so a reader can check it without CodeLead.

## Secrets

CodeLead removes secrets from every text that it copies:

- the repository's path and the home folder become placeholders;
- passwords in connection strings and provider keys are removed;
- the contents of each file that you name with `--secret-file <PATH>` are removed. You can name more than one file.

Read the sessions before you send the pack outside your team. A session holds the prompts and the model's answers, and they hold your code.

## Details

- **Options**: `--since <COMMIT>`, the commit before the work started (the default is the parent of the first checkpoint commit); `--out <FOLDER>`; `--secret-file <PATH>`.
- **The program**: the Hand over stage runs it in the old application: `codelead modernize run audit`, or `/modernize --run audit`. Run `/modernize --audit-pack` in the rebuild's own chat too, for the rebuild's increments.
- **Waivers**: a security finding that you decided not to fix is recorded with [`/modernize --waive`](../chat-commands/modernize.md#--waive) in `docs/codelead/security-waivers.json`, and the pack lists it in `5-security/`.
- **The manifest**: `codelead.audit-pack-manifest/1`, with `head`, `since`, and each file's `path`, `bytes` and `sha256`.
- **The catalog**: the pack's `README.md` is added to the repository's document catalog, `docs/codelead/CATALOG.md`.

## What it is not

- **The audit pack is not a report that a model wrote**. CodeLead assembles it from records, with no model call.
- **The audit pack does not hide stops**. A refused or stopped request is kept with its reason, in the sessions' index.
- **The audit pack is not updated by itself**. Run it again after more work, and it is written again.

## Related content

- [The eight stages](the-eight-stages.md#8-hand-over)
- [`/modernize --audit-pack`](../chat-commands/modernize.md#--audit-pack)
- [Evidence records](../concepts/evidence-records.md) and [Evidence record format](../reference/evidence-record-format.md)
- [Sessions](../concepts/sessions.md)
- [Export a session](../how-to/export-a-session.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
