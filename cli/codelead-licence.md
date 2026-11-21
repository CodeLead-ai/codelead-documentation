---
title: codelead licence
description: Show this folder's licence and its plan, free or pro, with no model call and no network.
kind: reference
area: cli
date: 2026-11-20
---

# codelead licence

**Applies to:** CodeLead 0.1 and later

`codelead licence` shows the licence of the current folder and its plan. It calls no model and uses no network.

## Name

`codelead licence` - Shows this folder's licence and its plan (free or pro).

## Synopsis

```console
codelead licence [--json]
codelead licence -h|--help
```

## Description

A CodeLead licence is a signed file, `docs/codelead/licence.json`, in the repository. It names the holder, the project, the term, the seats and the plan. The **plan** decides which parts of CodeLead a folder can use. The pro plan adds the [Analyst](../analyst/index.md), the Explorer and the modernization views to the free plan. See [Licence and plans](../concepts/licence-and-plans.md).

`codelead licence` reads that file and checks its signature. It never sends anything anywhere.

> [!NOTE]
> **New to this?** A folder without a licence file is not an error. Its plan is free, and CodeLead does not warn about it.

### The report

Without `--json`, the command prints the report:

- **Licence**: the file, the holder and the term, or "No licence file".
- **Warnings**: for example, more active people than seats. A warning never stops a command.
- **How the count was made**: the people active in the last 30 days, from the evidence records and the board's events. A person is counted by the hash of their git email, never by the email itself.

### The JSON for a client

With `--json`, the command prints only the state and the plan. The Visual Studio Code extension runs it on each start to know which sections to show. The signature is never printed.

> [!TIP]
> **For experienced users:** `--json` prints `state` (`none`, `valid` or `invalid`) and `plan` (`free` or `pro`). A valid licence adds `repositoryListed`, `holder`, `project`, `term`, `seatBlocks` and `issued`. An invalid one adds `why`.

`codelead licence` runs before the first-run acceptance of a packaged release, like [`codelead doctor`](codelead-doctor.md).

## Options

- **`--json`**

  Prints the licence's state and plan as JSON, for a client such as the Visual Studio Code extension. The default is the report in words.

## Examples

- Show the licence of a folder that has none:

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
    Window: the last 30 days, 2026-10-21T04:12:56Z to 2026-11-20T04:12:56Z (UTC).
    Evidence records read: 1 (.codelead/evidence), 1 in the window.
    Board events read: 0 (docs/codelead/board/events.jsonl), 0 in the window.
    Repositories read: 1 (paths relative to the workspace)
      - basic-store: root commit c5a5ce8920b8
      Only the board's participants of kind person count.
    One person: on the evidence records, the hash of their git email.
  …
    Active people: 1
      - <your name>: last seen 2026-11-20T04:10:50Z; evidence
    Left out: 0 (viewers write nothing and are never read)

    Seats: none (no licence that can be used), so nothing to be over.
  ```

- Read the state and the plan as JSON:

  ```console
  codelead licence --json
  ```

  ```output
  {
    "state": "none",
    "plan": "free"
  }
  ```

- See the plan gate at work. A pro command in a free folder says which plan the folder has:

  ```console
  codelead analyst check journeys.json
  ```

  ```output
  The Analyst is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The licence was read, in every state: none, valid or invalid. |
| `1` | The report could not be written out. |

## Files

`codelead licence` writes nothing. It reads:

- `docs/codelead/licence.json`, the licence.
- `.codelead/evidence/`, the evidence records, for the seat count.
- `docs/codelead/board/events.jsonl`, the board's events, for the seat count.
- The repository's root commit, which ties the licence to the repository.

## See also

- [Licence and plans](../concepts/licence-and-plans.md)
- [`codelead doctor`](codelead-doctor.md): `--licence` prints the same report after the machine's checks.
- [`codelead analyst`](codelead-analyst.md): a command of the pro plan.
- [The Explorer view](../extension/explorer-view.md)
