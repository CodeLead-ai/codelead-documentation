---
title: Export a session
description: Write a zip of a session's log with /session --export or codelead sessions export, keep it out of git, and read it before you send it to anyone.
kind: how-to
area: how-to
date: 2026-11-20
---

# Export a session

This guide writes the log of a [session](../concepts/sessions.md) into one zip file. A person who helps you can then read exactly what CodeLead was asked, what it did, and what the model answered. You can also open the zip in the Session Log Viewer.

**In short**
- [`/session --export`](../chat-commands/session.md) in the chat, or [`codelead sessions export`](../cli/codelead-sessions.md) in a terminal, writes the zip of the latest session.
- Add `all` (in the chat) or `--all` (in a terminal) for every session of the project.
- The zip goes to `.codeleadsessions/exports/`, which git ignores. `--out` writes it to another path.
- Read the zip before you send it: it holds the prompts, and so parts of your code.

## Before you begin

- The project has at least one session. Every [`codelead interactive`](../cli/codelead-interactive.md) session and every chat in the extension writes one, under `.codeleadsessions/`.
- You know who may read your code. A session log is as private as the repository.

> [!WARNING]
> A session log keeps every prompt that CodeLead sent to a model. The prompts hold the files that CodeLead selected, and build output. Treat the zip as you treat the repository. Never attach it to a public issue.

## Find the session

List the sessions of the project, newest first:

```console
codelead sessions list
```

```output
session-2026-11-20T07-30-43-625Z  (started 2026-11-20T07:30:43.625Z)
```

The name of each session is its start time in UTC.

## Export it

Choose one way.

- **In the chat or a terminal session**, type:

  ```console
  /session --export
  ```

  ```output
  Exported 1 session (2 files) to .codeleadsessions/exports/session-2026-11-20T09-47-51-152Z.zip

  Open this zip in the Session Log Viewer to review the session. A session holds prompts and code: keep the zip out of commits (/pr never stages one).
  ```

- **In a terminal, outside a session**, run:

  ```console
  codelead sessions export
  ```

  ```output
  Exported 1 session (2 files, 9.5 KB) to .codeleadsessions/exports/session-2026-11-20T09-46-25-547Z.zip
  ```

Without `--out`, the zip goes to `.codeleadsessions/exports/`, named after the session. Git ignores the sessions folder, so a later `git add -A` does not pick up the zip. [`/pr`](../chat-commands/pr.md) never stages an exported zip, also one that `--out` wrote to another place.

To write the zip to another place, for example to send it, give `--out`:

```console
codelead sessions export --out ../session-for-review.zip
```

```output
Exported 1 session (2 files, 7.1 KB) to <the folder above your project>/session-for-review.zip
```

To export another session, name it: `codelead sessions export session-2026-11-20T07-30-43-625Z`. To export every session of the project in one zip, use `codelead sessions export --all`, or `/session --export all` in a session. That zip is `.codeleadsessions/exports/codelead-sessions.zip`.

## Read it before you send it

1. Open the zip, or list it:

   ```console
   unzip -l ../session-for-review.zip
   ```

   ```output
   Archive:  ../session-for-review.zip
     Length      Date    Time    Name
   ---------  ---------- -----   ----
       18829  11-19-2026 23:30   session-2026-11-20T07-30-43-625Z/model/chat-prompt-001.md
       10362  11-19-2026 23:30   session-2026-11-20T07-30-43-625Z/session.jsonl
         638  11-19-2026 23:30   session-2026-11-20T07-30-43-625Z/summary.json
   ---------                     -------
       29829                     3 files
   ```

   On Windows, open the zip in File Explorer.

2. Read the files under `model/`. Each prompt file is exactly what left your machine for that call.
3. Remove the zip, and export again without that session, when a prompt holds code that the reader may not see.

## Check that it worked

- The command printed `Exported <n> session` and the path of the zip.
- The zip holds one folder for each session, with `session.jsonl` and `summary.json` in it.
- The reader opens it in the Session Log Viewer, or reads `session.jsonl` as text. See [Read a session in the Session Log Viewer](read-a-session.md).

## Details

- **What the zip holds.** Every file of each session's folder. That is `session.jsonl`, `summary.json` and the prompts and responses under `model/`. It is also the state of an `/implement` or of a new app's build, when there is one.
- **What is removed.** The log already leaves out the password of a connection string that you typed as an answer. Nothing else is removed from the zip.
- **Another sessions folder.** `CODELEAD_SESSIONS_DIR` moves the folder, and the export reads the same variable.
- **A modernization.** The audit pack holds a copy of every session too, in `4-sessions/`. Read it as you read a zip. See [The audit pack](../modernization/audit-pack.md).

## See also

- [Sessions](../concepts/sessions.md)
- [`/session`](../chat-commands/session.md) and [`codelead sessions`](../cli/codelead-sessions.md)
- [Read a session in the Session Log Viewer](read-a-session.md)
- [The Sessions view](../extension/sessions-view.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
