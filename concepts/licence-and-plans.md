---
title: Licence and plans
description: Learn what a CodeLead licence file is, what the free and the pro plans include, how seats are counted, and what CodeLead does when something is wrong.
kind: concept
area: concepts
date: 2026-11-20
---

# Licence and plans

A CodeLead licence is a signed file in the repository: `docs/codelead/licence.json`. It names the holder, the project, the term, the seats (how many people can use it) and the **plan**. The plan decides which parts of CodeLead a folder can use: **free** or **pro**. A folder without a licence file is on the free plan, and CodeLead does not warn about it.

**In short**
- The free plan has projects, the chat, sessions, the board, settings, the evidence and the knowledge base. It also has the spec walk: CodeLead following a written spec on the running app.
- The pro plan adds the Analyst and the Explorer, which record and crawl a running application, and modernization.
- CodeLead checks the licence's signature on your machine. It never calls a server.
- A licence problem is a warning. It never stops a run. One exception: the pro features refuse on the free plan. They are the Analyst, the Explorer and the modernization program.

## Why it exists

A team needs to know what it may use, and the people who make CodeLead need to know that a licence is respected. The licence is a file, not an online account, because CodeLead works local first. Any clone of the repository carries the file, and every copy of CodeLead can check it with no network.

## How it works

### The plans

| Plan | What it includes |
|---|---|
| `free` | Projects and the chat, Sessions, the Board, Settings, Evidence, Knowledge, and the spec walk. Every command that changes code, such as [`/implement`](../chat-commands/implement.md) and [`/create-app`](../chat-commands/create-app.md). |
| `pro` | All of free, and the [Analyst](../analyst/index.md), the Explorer and [Modernization](../modernization/index.md). |

CodeLead's own commands check the plan before a pro feature runs. A real refusal on the free plan:

```console
codelead analyst check journeys.json
```

```output
The Analyst is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.
```

The modernization program's commands refuse in the same words. A real refusal, in a folder with no licence file:

```console
codelead modernize status
```

```output
The modernization program is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.
```

The exit code is 2. Nothing ran, and nothing was written.

On the free plan, the extension still lists the pro views. Each one shows a single row, for example "Explorer is in CodeLead's Pro plan", and its buttons are hidden. That is a convenience. The check in CodeLead's commands is what counts.

### Reading the licence

[`codelead licence`](../cli/codelead-licence.md) prints the licence, its warnings and how the seats were counted. A real report in a folder with no licence file:

```console
codelead licence
```

```output
Licence (docs/codelead/licence.json)
  No licence file. A project without one is not warned.
  Plan: free (no licence file)

Warnings
  None.

How the count was made
  Window: the last 30 days, 2026-10-21T05:11:45Z to 2026-11-20T05:11:45Z (UTC).
  Evidence records read: 1 (.codelead/evidence), 1 in the window.
  Board events read: 0 (docs/codelead/board/events.jsonl), 0 in the window.
…
  Active people: 1
    - <your name>: last seen 2026-11-20T04:57:58Z; evidence
  Left out: 0 (viewers write nothing and are never read)

  Seats: none (no licence that can be used), so nothing to be over.
```

The extension reads the plan with `codelead licence --json`:

```output
{
  "state": "none",
  "plan": "free"
}
```

### Seats

A licence sells seats in blocks of 10. An evaluation licence has 0 blocks and a short term. CodeLead counts the people who were active in the last 30 days. It reads two sources: the [evidence records](evidence-records.md) of the repository, and the events of its board. A person is counted by a hash of their git email, never by the email itself. Viewers write nothing, so they are never counted.

### Warnings

CodeLead warns in four places: the console at the start of a run, the run report's header, [`codelead doctor`](../cli/codelead-doctor.md), and the audit pack's summary. It warns when:

- the file cannot be used: it cannot be read, it is malformed, or it changed after it was issued;
- this repository is not among the licence's repositories;
- more people were active in the last 30 days than the licence has seats;
- the term ends within 14 days, or has ended.

A warning never stops a run and never changes a result.

> [!NOTE]
> **For team leads:** commit `docs/codelead/licence.json` in each repository of the project, so that every clone has it. Edit nothing in it: a changed file no longer verifies.

## What it is not

- **Not an online check**. Nothing in the licence calls the network. The term is compared with today's date in UTC.
- **Not a lock**. Over the seats, CodeLead warns. It does not hold anybody's work.
- **Not stored in an account**. The file is the licence. Delete it, and the folder is on the free plan.

## Details

- **The file**. `docs/codelead/licence.json`, schema `codelead.licence/1`. Its fields: `holder`, `project`, `repositories`, `seatBlocks`, `term` (`from` and `to`, both days included), `issued`, `onOverage`, `plan` and `signature`.
- **The signature**. An ed25519 signature over every other field. CodeLead carries the public key that checks it.
- **The repositories**. The project is named by the root commits of its repositories: a root commit never changes across clones, branches or forks. An empty list means every repository of the holder.
- **The plan field**. `free` or `pro`. A licence without the field is free.
- **`onOverage`**. `warn` is the only behaviour built today.
- **JSON**. `codelead licence --json` prints `state` (`none`, `valid` or `invalid`) and `plan`. A valid licence adds `repositoryListed`, `holder`, `project`, `term`, `seatBlocks` and `issued`. An invalid one adds `why`.
- **Where the plan is checked**. Each pro command checks it once, before it runs:
  - the Analyst's commands and the Explorer ([`codelead analyst`](../cli/codelead-analyst.md)), exit code 1;
  - every subcommand of [`codelead modernize`](../cli/codelead-modernize.md) except `stop`, and the chat's [`/modernize`](../chat-commands/modernize.md);
  - [`codelead rebuild`](../cli/codelead-rebuild.md) (`--plan`, `run` and `init`), [`codelead model`](../cli/codelead-model.md), [`codelead register`](../cli/codelead-register.md), [`codelead seed`](../cli/codelead-seed.md), [`codelead parity`](../cli/codelead-parity.md), [`codelead rebuild-report`](../cli/codelead-rebuild-report.md) and [`codelead walk --rebuild`](../cli/codelead-walk.md).
- **The exit codes of the program's refusal**. `2`, except `codelead rebuild run` and `codelead rebuild --plan`, which exit with `1`.
- **What stays free**. Every `--help`. `codelead modernize stop`, so that a person can always stop what runs on the machine. [`codelead walk`](../cli/codelead-walk.md) on a spec, [`codelead spec`](../cli/codelead-spec.md), `codelead licence` and `codelead doctor`.
- **A rebuild's plan**. A rebuild is a new repository with its own root commit, which a licence of the old application does not list. So a rebuild is on the Pro plan when its own folder is, or when the old application that its manifest names is.
- **Unattended runs** check the plan once, at their start: `codelead rebuild run`, the program's run and the board queue.
- **The doctor**. `codelead doctor --licence` prints the same report after the machine's checks.

## Related content

- [`codelead licence`](../cli/codelead-licence.md) and [`codelead doctor`](../cli/codelead-doctor.md)
- [`codelead analyst`](../cli/codelead-analyst.md)
- [`codelead modernize`](../cli/codelead-modernize.md)
- [The Explorer view](../extension/explorer-view.md) and [The Analyst view](../extension/analyst-view.md)
- [Evidence records](evidence-records.md)
- [Files and folders](../reference/files-and-folders.md)
- [Glossary](glossary.md)
