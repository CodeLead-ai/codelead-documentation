---
title: /session
description: Export session logs as a zip, or pause, resume and discard an /implement across sessions.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /session

**Applies to:** CodeLead 0.1 and later

## Name

`/session` - Exports session logs, or pauses, resumes or discards an `/implement`.

## Synopsis

```console
/session --export
/session --export all
/session --export <SESSION_ID>
/session --export --out <PATH>
/session --resume [<SESSION_ID>]
/session --pause [<NOTE>]
/session --discard [<SESSION_ID>]
```

## Description

Every session writes a log to `.codeleadsessions/<session id>/`: what you typed, what CodeLead answered, each model call, and each command's result. See [Sessions](../concepts/sessions.md). `/session` does two jobs with these logs.

**Export.** `/session --export` writes a zip of a session's log, for the [Session Log Viewer](../how-to/read-a-session.md) or for a person who helps you. `all` exports every session of the project in one zip.

**Continue an `/implement`.** An [`/implement`](implement.md) saves its state at every step. When you stop in the middle, a later session can continue from there:

- `/session --pause <NOTE>` saves where the `/implement` stands, with your note, and ends the session.
- `/session --resume` loads the latest paused `/implement` and says what comes next. CodeLead warns when the files that the patch touches changed after the pause, or when git `HEAD` moved.
- `/session --discard` drops a paused `/implement`, so that it is not offered again. Its log and its evidence stay.

> [!NOTE]
> **New to this?** You do not have to pause by hand. When a session ends in the middle of an `/implement`, the next session in the same folder offers to continue it.

> [!WARNING]
> A session log holds the prompts sent to models, and so it can hold parts of your code. Read [Export a session](../how-to/export-a-session.md) before you send a zip outside your team.

## Arguments

- **`all`**

  With `--export`: every session of the project.

- **`<SESSION_ID>`**

  The name of a session folder, for example `session-2026-11-20T04-30-31-668Z`.

- **`<NOTE>`**

  With `--pause`: a note for whoever continues, in plain words.

## Options

- **`--export`**

  Writes a zip of the latest session's log to `.codeleadsessions/exports/<SESSION_ID>.zip`. Git ignores that folder. With `all`, it writes every session to `.codeleadsessions/exports/codelead-sessions.zip`. With a session id, it writes that session.

- **`--out <PATH>`**

  With `--export`: the path of the zip, in place of the exports folder.

- **`--resume`**

  Continues the latest paused `/implement`, or the one of the session that you name. It refuses when this session already has an `/implement` in progress.

- **`--pause`**

  Saves the `/implement` of this session, with the note that follows, and ends the session.

- **`--discard`**

  Drops the latest paused `/implement`, or the one of the session that you name.

`/session` alone is refused, and so are the forms without dashes, such as `/session export`.

## What you see

| Command | Headline |
|---|---|
| `--export` | `Exported 1 session (2 files) to .codeleadsessions/exports/<SESSION_ID>.zip` |
| `--resume` | `Resumed`, then the request, where it stopped, and `Next: …` |
| `--resume` | `No paused /implement session found.` |
| `--pause` | `Paused at ready_to_apply.`, then how to continue. The stage is the one that the session is really in, also after an `/edit` or a `/validate` by hand. |
| `--pause` | `Nothing to pause: no /implement has run in this session.` |
| `--discard` | `Discarded the paused /implement from <session> (<request>). Its session log and evidence are kept.` |
| `--discard` | `No paused /implement session to discard.` |

## Examples

- Export the latest session for the Session Log Viewer:

  ```console
  /session --export
  ```

  ```output
  Exported 1 session (2 files) to .codeleadsessions/exports/session-2026-11-20T09-47-51-152Z.zip

  Open this zip in the Session Log Viewer to review the session. A session holds prompts and code: keep the zip out of commits (/pr never stages one).
  ```

- Export every session of the project in one zip:

  ```console
  /session --export all
  ```

  <!-- described, not captured -->
  ```output
  Exported 7 sessions (25 files) to .codeleadsessions/exports/codelead-sessions.zip

  Open this zip in the Session Log Viewer to review the session. A session holds prompts and code: keep the zip out of commits (/pr never stages one).
  ```

- Pause an `/implement` while the model server is down, and continue it later:

  ```console
  /session --pause waiting for the model server
  ```

  ```output
  Paused at ready_to_apply.
  Start CodeLead in this repository again and type /session --resume to continue.
  Exiting CodeLead.
  ```

  In the next session:

  ```console
  /session --resume
  ```

  ```output
  Resumed
  Request: add a clearCompleted function that removes the done tasks, with a test
  Stopped at: ready_to_apply

  Next: review with /patch --show, apply with /edit, then /validate
  ```

- Pause after you applied the patch with `/edit`, and ran `/validate` by hand, once the `/implement` stopped. The line names the stage that the session is really in, and `/session --resume` continues from it:

  <!-- described, not captured -->
  ```output
  Paused at validation_failed (the /implement stopped at ready_to_apply; the patch was applied and validation failed by hand since).
  Start CodeLead in this repository again and type /session --resume to continue.
  Exiting CodeLead.
  ```

- Resume a session that has nothing paused:

  ```console
  /session --resume session-2026-11-20T00-00-00-000Z
  ```

  ```output
  Session session-2026-11-20T00-00-00-000Z has no paused /implement to resume.
  ```

- `/session` alone is refused:

  ```console
  /session
  ```

  ```output
  /session requires --export, --resume, --pause or --discard.

  Usage:
    /session --export
    /session --export all
    /session --export <sessionId>
    /session --export --out <path>
    /session --resume [<sessionId>]
    /session --pause [note]
    /session --discard [<sessionId>]
  ```

## Files

- Reads `.codeleadsessions/<session>/`. The folder's name can be changed with `CODELEAD_SESSIONS_DIR`.
- `--export` writes a zip: in `.codeleadsessions/exports/` by default, which git ignores, or at `--out`. [`/pr`](pr.md) never stages an exported zip.
- `--pause` writes `.codeleadsessions/<session>/implement/state.json`. `--resume` and `--discard` mark the saved state as resumed or discarded.

## See also

- [Sessions](../concepts/sessions.md)
- [Export a session](../how-to/export-a-session.md)
- [Read a session in the Session Log Viewer](../how-to/read-a-session.md)
- [Stop and resume work](../how-to/stop-and-resume.md)
- [`/implement`](implement.md)
- [`codelead sessions`](../cli/codelead-sessions.md)
