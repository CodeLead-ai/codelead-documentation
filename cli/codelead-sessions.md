---
title: codelead sessions
description: List the session logs of a repository and export them as a zip archive for the Session Log Viewer or for a bug report.
kind: reference
area: cli
date: 2026-11-20
---

# codelead sessions

**Applies to:** CodeLead 0.1 and later

`codelead sessions` lists the session logs that CodeLead recorded for the current repository, and exports them as a zip archive.

## Name

`codelead sessions` - Inspects and exports CodeLead session logs.

## Synopsis

```console
codelead sessions list
codelead sessions export [<SESSION_ID>] [--all] [--out <PATH>]
codelead sessions -h|--help
```

## Description

Every [session](../concepts/sessions.md) writes a log: each line that you typed, each model call with its prompt and its response, each command's result. The logs live in `.codeleadsessions/` in the project, one folder for each session.

- `codelead sessions list` prints each session's id and when it started, the newest first.
- `codelead sessions export` writes a zip archive of one session, or of all of them. Open the archive in the Session Log Viewer, or attach it to a bug report.

Neither subcommand calls a model.

> [!WARNING]
> A session log holds your prompts, the model's responses and parts of your code. Read the archive before you send it to anyone. CodeLead never commits the logs: [`codelead pr`](codelead-pr.md) leaves `.codeleadsessions/` and every exported zip out of every commit.

> [!TIP]
> **For experienced users:** `CODELEAD_SESSIONS_DIR` moves the logs' folder, and both subcommands read it. The archive holds each session's `session.jsonl`, `summary.json` and `model/` files.

## Arguments

- **`<SESSION_ID>`**

  For `export`: the session folder to export, for example `session-2026-11-20T04-18-27-914Z`. The default is the latest session.

## Options

These options belong to `codelead sessions export`.

- **`--all`**

  Exports every session in one archive.

- **`--out <PATH>`**

  The path of the zip archive. The default is `.codeleadsessions/exports/<SESSION_ID>.zip`, or `.codeleadsessions/exports/codelead-sessions.zip` with `--all`. Git ignores that folder.

## Examples

- List the sessions of the current repository:

  ```console
  codelead sessions list
  ```

  ```output
  session-2026-11-20T04-18-27-914Z  (started 2026-11-20T04:18:27.914Z)
  session-2026-11-20T04-18-16-661Z  (started 2026-11-20T04:18:16.661Z)
  ```

- Export the latest session:

  ```console
  codelead sessions export
  ```

  ```output
  Exported 1 session (2 files, 9.5 KB) to .codeleadsessions/exports/session-2026-11-20T09-46-25-547Z.zip
  ```

- Export every session into one archive at a path that you choose:

  ```console
  codelead sessions export --all --out ../all-sessions.zip
  ```

  <!-- described, not captured -->
  ```output
  Exported 2 sessions (4 files, 97.9 KB) to <the folder above your project>/all-sessions.zip
  ```

- Ask for a session that does not exist. CodeLead lists the ones it knows:

  ```console
  codelead sessions export session-nope
  ```

  ```output
  Error:
  Session session-nope not found. Known sessions: session-2026-11-20T04-18-27-914Z, session-2026-11-20T04-18-16-661Z
  ```

- In a repository with no session yet:

  ```console
  codelead sessions list
  ```

  ```output
  No sessions found.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The list was printed (an empty list too), or the archive was written. |
| `1` | `export` found no session, or not the one that you named. |

## Files

| File | What `sessions` does with it |
|---|---|
| `.codeleadsessions/<session id>/` | Reads the logs. |
| `.codeleadsessions/exports/<SESSION_ID>.zip`, or the `--out` path | `export` writes the archive. Git ignores the exports folder. |

## See also

- [Export a session](../how-to/export-a-session.md)
- [Sessions](../concepts/sessions.md)
- [`codelead interactive`](codelead-interactive.md): the session that writes the logs.
- [`/session`](../chat-commands/session.md): the same export from inside a session.
- [The Sessions view](../extension/sessions-view.md)
