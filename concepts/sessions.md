---
title: Sessions
description: Learn what a CodeLead session is, what its log records, where the log lives, and how to pause, resume and export your work.
kind: concept
area: concepts
date: 2026-11-23
---

# Sessions

A session is one conversation with CodeLead about one repository. It starts when you run [`codelead interactive`](../cli/codelead-interactive.md) or open the chat in the extension, and it ends when you type `/exit` or close the chat. Every session writes a **session log**: what you typed, what CodeLead answered, each model call with its prompt and its response, and each command's result.

**In short**
- A session's log is a folder: `.codeleadsessions/<session id>/`, with `session.jsonl`, `summary.json` and the model calls under `model/`.
- The log is the full record of a session. An [evidence record](evidence-records.md) is the short record of one change.
- An `/implement` saves its state at each step, so a later session can continue it.
- [`/session --export`](../chat-commands/session.md) and [`codelead sessions export`](../cli/codelead-sessions.md) write a zip of the log.

## Why it exists

A model's answer cannot be checked later unless its question was kept. The session log keeps both. When a change goes wrong, you or a person who helps you can read what was asked, what the model saw and what it answered. A person who did not watch the session can read it afterward.

## How it works

Every line of `session.jsonl` is one event, as JSON. These lines come from a real session in which a person typed `/strategy`, then `/validate`, and pressed Enter at the question. The lines are shortened:

```output
{"type":"session.started","timestamp":"2026-11-20T05:10:28.191Z","schema":"codelead.session-log/1","sessionId":"session-2026-11-20T05-10-28-190Z","repoRoot":"<your project>"}
{"type":"user.input","timestamp":"2026-11-20T05:10:28.217Z","commandId":"cmd-001","inputKind":"command","input":"/strategy","command":"/strategy"}
{"type":"command.completed","timestamp":"2026-11-20T05:10:28.217Z","commandId":"cmd-001","status":"completed","durationMs":1,"summary":"Processed strategy command.","result":{"outcome":"done","headline":"Current strategy: auto","facts":[],"next":[]}}
…
{"type":"user.input","timestamp":"2026-11-20T05:10:31.556Z","commandId":"cmd-002","inputKind":"prompt_response","input":"","prompt":"Run these commands? [Y/n]","emptyAnswer":"y","recommended":"y"}
…
{"type":"command.completed","timestamp":"2026-11-20T05:10:31.868Z","commandId":"cmd-002","status":"completed","durationMs":3650,"summary":"Processed validate command.","result":{"outcome":"passed","headline":"Validation passed: 2 commands","facts":["Passed: npm run test","Passed: npm run build","Scope: repo root"],"next":[],"detail":{"block":"validation"}}}
{"type":"session.ended","timestamp":"2026-11-20T05:10:36.573Z","reason":"user_exit"}
```

The answer line keeps the question, the answer that CodeLead recommended and what Enter gave. Here the person pressed Enter, so the input is empty and the answer was the recommended yes. See [Prompts and recommended answers](prompts-and-recommended-answers.md).

Each command ends with its result: the outcome, the headline, the facts and the way on. See [Results and stops](results-and-stops.md).

### Continue an /implement later

An [`/implement`](../chat-commands/implement.md) writes its state at each step to `implement/state.json` in the session's folder. When a session ends in the middle of an `/implement`, the next session in the same folder offers to continue it.

- `/session --pause <note>` saves where the `/implement` stands, with your note, and ends the session.
- `/session --resume` loads the latest paused `/implement`, and says what comes next. CodeLead warns when the files that the patch touches changed after the pause, or when git's `HEAD` moved.
- `/session --discard` drops a paused `/implement`, so that it is not offered again. Its log and its evidence stay.

> [!WARNING]
> A session log holds the prompts sent to models, and so it holds parts of your code. Treat the folder as you treat the repository. Read the zip before you send it outside your team.

## What it is not

- **Not committed**. CodeLead adds the sessions folder to `.gitignore`. [`/pr`](../chat-commands/pr.md) never stages it.
- **Not the evidence record**. The record is short, has no code, and is meant for a reviewer. The log is long and complete, and is meant for a person who investigates.
- **Not sent anywhere**. The log stays on your machine. Only you send a zip, by hand.

## Details

- **The folder**. `.codeleadsessions/session-<start time>/`, with:
  - `session.jsonl`: one event for each line, schema `codelead.session-log/1`.
  - `summary.json`: the counts of commands by status, the model calls, the last command and the state of a new app's plan.
  - `model/`: one prompt file and one response file for each model call, such as `<agent>-prompt-001.md` and `<agent>-response-001.md`.
  - `implement/state.json` and `greenfield/`: the state of an `/implement` and of a new app's build.
- **The events** include `session.started`, `session.parameters`, `user.input`, `command.started`, `command.parsed`, `codelead.output`, `model.request`, `model.response`, `model.error`, `validation.result`, `command.completed` and `session.ended`. `session.parameters` holds the providers, the models, the roles and the switches of the session. A build adds its own events, such as `increment.started`.
- **Each model event** names the role, the provider, the model asked for and the model that answered.
- **Secrets**. The log removes secrets from your answers to CodeLead's questions before it writes them. It uses the rules of the audit pack: passwords, keys and tokens, but not the paths. The other events are written as they are. The audit pack sanitizes them when it copies the session. See [The audit pack](../modernization/audit-pack.md#secrets).
- **Switches**. `CODELEAD_SESSIONS_DIR` moves the folder. `CODELEAD_DISABLE_SESSION_LOGGING=1` writes no log. `CODELEAD_READ_ONLY=1` leaves `.gitignore` untouched.
- **A new app**. In an empty folder, the session's folder is made after the scaffold, so that the scaffold sees an empty folder.
- **Reading a log**. The Session Log Viewer opens an exported zip. The extension's **Session** tab shows the same events as cards. The editor protocol is [Session protocol](../reference/session-io-protocol.md).

A real list of the sessions of a folder:

```console
codelead sessions list
```

```output
session-2026-11-20T05-10-28-190Z  (started 2026-11-20T05:10:28.190Z)
```

## Related content

- [`/session`](../chat-commands/session.md)
- [`/log`](../chat-commands/log.md): the project log, one entry for each session
- [`codelead sessions`](../cli/codelead-sessions.md) and [`codelead interactive`](../cli/codelead-interactive.md)
- [Export a session](../how-to/export-a-session.md)
- [Read a session in the Session Log Viewer](../how-to/read-a-session.md)
- [Stop and resume work](../how-to/stop-and-resume.md)
- [The Sessions view](../extension/sessions-view.md)
- [Session protocol](../reference/session-io-protocol.md)
- [Evidence records](evidence-records.md)
- [Glossary](glossary.md)
