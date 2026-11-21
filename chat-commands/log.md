---
title: /log
description: Show the project log, one entry per session, or write the project documents again from every session on disk.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /log

**Applies to:** CodeLead 0.1 and later

## Name

`/log` - Shows the project log, or writes the project documents again from every session.

## Synopsis

```console
/log
/log --rebuild
```

## Description

The project log, `docs/codelead/PROJECT-LOG.md`, says what happened to this project through CodeLead, one entry per session, newest first. Each entry lists the commands of the session, the number of model calls, the CodeLead build, and how each request ended. A table at the top lists the stops (the requests that did not land) and what followed each one.

`/log` prints the log with its five newest entries. `/log --rebuild` reads every session log under `.codeleadsessions/` and writes three documents again:

| File | What it holds |
|---|---|
| `docs/codelead/PROJECT-LOG.md` | What happened to the project, one entry per session. |
| `docs/codelead/PINNED.md` | What waits on a person. See [`/pin`](pin.md). |
| `docs/codelead/CATALOG.md` | Every document that CodeLead created or keeps in this repository, with its purpose. |

You use `/log` to remember where a project stands. You use `/log --rebuild` when the documents are missing or old, for example after you copied session logs from another machine.

> [!NOTE]
> **For team leads:** the log is a readable view of the session logs. The full record of each session, with every model call, stays in `.codeleadsessions/`. See [Sessions](../concepts/sessions.md).

## Options

- **`--rebuild`**

  Writes the three project documents again from every session on disk. The form without dashes, `/log rebuild`, is refused.

## What you see

| Command | Headline |
|---|---|
| `/log` | `The project log: the 2 latest sessions, newest first (PROJECT-LOG.md)`. The log is the output, under its title `# Project log`. |
| `/log` | `The project log has no session yet (PROJECT-LOG.md)` |
| `/log --rebuild` | `Rebuilt docs/codelead: 3 session entries, 1 open pins.` |
| `/log --rebuild` | `No sessions found to rebuild from.` |

## Examples

- Show the log of a project:

  ```console
  /log
  ```

  ```output
  # Project log

  What happened to this project through CodeLead, one entry per session, newest first. Built from the session logs; the full record of each session is under `.codeleadsessions/`.

  ## Stops and what followed them

  17 requests in 5 sessions: 0 landed, 1 stopped. …

  | Session | Request | Stop reason | Followed by |
  |---|---|---|---|
  | session-2026-11-20T04-31-22-083Z | add a clearCompleted function that removes the done tasks | preflight failed | not relaunched |

  ## 2026-11-20 04:36 UTC — session-2026-11-20T04-36-31-425Z

  Commands: /session ×2, /edit ×3, /patch, /validate, /repair. Model calls: 0. CodeLead 0.1.0 (…).

  - **/patch** "--check" → paused (ready_to_apply). Files: src/tasks.js, test/tasks.test.js. Validation: failed (1 command).
  - Resumed the /implement paused in session-2026-11-20T05-00-00-000Z.
  …
  ```

- Show the log of a project with no session yet:

  ```output
  # Project log

  What happened to this project through CodeLead, one entry per session, newest first. Built from the session logs; the full record of each session is under `.codeleadsessions/`.

  No sessions recorded yet.
  ```

- Write the documents again from every session:

  ```console
  /log --rebuild
  ```

  ```output
  Rebuilt docs/codelead: 3 session entries, 1 open pins.
  ```

## Files

- Reads `docs/codelead/PROJECT-LOG.md`.
- With `--rebuild`, reads `.codeleadsessions/*/session.jsonl` and writes `docs/codelead/PROJECT-LOG.md`, `docs/codelead/PINNED.md` and `docs/codelead/CATALOG.md`.

## See also

- [`/pin`](pin.md)
- [`/session`](session.md)
- [Sessions](../concepts/sessions.md)
- [Files and folders](../reference/files-and-folders.md)
