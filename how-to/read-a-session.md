---
title: Read a session in the Session Log Viewer
description: Open a session's log in the extension's Session tab, in the Session Log Viewer or as a file, and find the command, the question, the model call and the result that matter.
kind: how-to
area: how-to
date: 2026-11-20
---

# Read a session in the Session Log Viewer

This guide reads the log of a [session](../concepts/sessions.md): every line typed, every line that CodeLead wrote, every model call and every result. You read it to find why a command stopped, what a model was asked, or what you answered. Three tools show the same log: the Session tab in VS Code, the Session Log Viewer, and any text editor.

**In short**
- In VS Code, select a session in the **Sessions** view. The **Session** tab shows the commands and the events as cards.
- The Session Log Viewer is a desktop app. It opens a zip from [`/session --export`](../chat-commands/session.md), or a project's sessions folder.
- The log itself is `.codeleadsessions/<session>/session.jsonl`: one JSON event on each line.
- The events that matter most are `command.completed`, `user.input`, `model.request`, `model.response` and `model.error`.

## Before you begin

- The project has a session. Every chat and every [`codelead interactive`](../cli/codelead-interactive.md) session writes one.
- For the Session Log Viewer: the app on your machine, and a zip from [Export a session](export-a-session.md), or access to the project folder.

> [!NOTE]
> **New to this?** A *prompt* is the text that CodeLead sends to a model. A *response* is what the model answers. The log keeps both, each in its own file under `model/`.

## Open the session

Choose one tool.

- **In VS Code**: open the **Sessions** view in the CodeLead side bar, and select a session. The tab `Session <time>` opens. From a chat, select **Session log** in the chat's header. See [The Sessions view](../extension/sessions-view.md).
- **In the Session Log Viewer**: select **Open zip file**, and choose the zip. Or select **Browse folders**, go to the project folder, and select **Open this folder**. The viewer shows the sessions, then the commands, then the timeline of the command that you choose.
- **As a file**: open `.codeleadsessions/<session>/session.jsonl` in a text editor.

Both views show five kinds of card, each with its label. In VS Code, the labels are **CodeLead → You**, **You → CodeLead**, **CodeLead → Model**, **Model → CodeLead** and **System / Other**. The Session Log Viewer says **User** in place of **You**. Filter chips show or hide each kind, and a search keeps the matching cards. Every card has a raw JSON view.

## Find the command that stopped

1. In the list of commands, find the command and its mark. A command that failed has the mark ✖.
2. Select it. The timeline scrolls to it.
3. Read its last card: the result, with the outcome, the headline, the facts and the way on.

In the file, the result is the `result` field of the `command.completed` event. In a terminal, find every result of a session:

```console
grep '"command.completed"' .codeleadsessions/session-2026-11-20T07-38-24-341Z/session.jsonl
```

```output
{"type":"command.completed","timestamp":"2026-11-20T07:38:24.562Z","commandId":"cmd-001","status":"failed","durationMs":200,"summary":"Processed validate command.","result":{"outcome":"failed","headline":"Validation failed: type check exited 2","facts":["Failed command: type check (exit 2)","Scope: repo root","Ran: 1 of 2 commands passed before it"],"next":["/repair","Or fix it by hand, then /validate"],"detail":{"block":"validation"}}}
{"type":"command.completed","timestamp":"2026-11-20T07:38:24.563Z","commandId":"cmd-002","status":"exited","durationMs":1,"summary":"Exited interactive session.","result":{"outcome":"done","facts":[],"next":[]}}
```

In PowerShell, use `Select-String -Path <the file> -Pattern command.completed`.

## Read what you answered

Each answer to a question is a `user.input` event with `"inputKind":"prompt_response"`. It keeps the question, what you typed, the answer that CodeLead recommended, and what Enter gives:

```output
{"type":"user.input","timestamp":"2026-11-20T07:38:17.978Z","commandId":"cmd-001","inputKind":"prompt_response","input":"y","prompt":"Run these commands? [Y/n]","emptyAnswer":"y","recommended":"y"}
```

An empty `input` means that you pressed Enter, so the answer was `emptyAnswer`. See [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md).

## Read a model call

1. Find the **CodeLead → Model** card of the call. Its event is `model.request`, with the `agent` (the part of CodeLead that called), the `role`, the `provider` and the `model`.
2. Select it. The view loads the prompt from `promptPath`, for example `model/chat-prompt-001.md`.
3. Read the **Model → CodeLead** card that follows. It is `model.response`, with the answer in `responsePath`, and `servedModel`: the model that really answered.
4. When the call failed, the card is `model.error`, with a `kind` and the message. For example, a server that did not answer:

   ```output
   {"type":"model.error","timestamp":"2026-11-20T07:30:45.079Z","commandId":"cmd-003","agent":"chat","promptPath":"model/chat-prompt-001.md","kind":"provider_error","message":"Could not connect to lmstudio at http://127.0.0.1:9/v1.\nIf this is LM Studio, make sure it is open and the Local Server is running.","provider":"lmstudio","role":"deep"}
   ```

## Know the events that matter

| Event | What it tells you |
|---|---|
| `session.started`, `session.ended` | When the session began and ended, and why it ended, such as `user_exit`. |
| `session.parameters` | The providers, the models, the roles and the `CODELEAD_*` switches of the session. |
| `user.input` | A command that you typed (`inputKind: "command"`), or an answer (`"prompt_response"`). |
| `command.started`, `command.completed` | The start and the end of a command. `command.completed` holds the `status` and the `result`. |
| `codelead.output` | A block that CodeLead wrote, word for word. |
| `model.request`, `model.response`, `model.error` | Each model call: who called, on which role, provider and model, and the files of the prompt and the response. |
| `model.call.completed` | The call's time, its thinking tokens and its `usage`: the tokens, and `costUsd` when the cost is known. It also names the role, the provider and the model of the call. |
| `model.prompt_budget` | The size of a patch, retry or repair prompt, its budget, and each part that was cut to fit. |
| `model.reply_cut` | A reply that stopped at its cap. |
| `validation.result` | Each command of the gate, with its status, its exit code and the start of its output. |

A new app's build adds its own events, such as `increment.started`.

## Check that it worked

- You found the command, its result and its way on.
- For a model call, you read the prompt and the response, and you know which model answered.
- The counts in the view match `summary.json`: the commands by status, and the model calls.

## Details

- **The folder.** `.codeleadsessions/session-<start time>/` holds `session.jsonl`, `summary.json` and `model/`. An `/implement` adds `implement/state.json`, and a new app adds `greenfield/`.
- **The format.** Each line is one JSON object with `type` and `timestamp`. The events of one command share its `commandId`. The schema is `codelead.session-log/1`.
- **Secrets.** The log writes no password of a connection string that you typed as an answer. CodeLead never writes a key into the log.
- **Another folder.** `CODELEAD_SESSIONS_DIR` moves the sessions folder.
- **Not committed.** CodeLead adds the sessions folder to `.gitignore`.

## See also

- [Sessions](../concepts/sessions.md)
- [The Sessions view](../extension/sessions-view.md)
- [Export a session](export-a-session.md)
- [`/session`](../chat-commands/session.md) and [`codelead sessions`](../cli/codelead-sessions.md)
- [Session protocol](../reference/session-io-protocol.md)
- [Fix a stop](fix-a-stop.md)
