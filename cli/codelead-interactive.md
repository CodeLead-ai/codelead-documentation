---
title: codelead interactive
description: Start a CodeLead session in the terminal, or a session over JSON lines that an editor drives.
kind: reference
area: cli
date: 2026-11-20
---

# codelead interactive

**Applies to:** CodeLead 0.1 and later

`codelead interactive` starts a CodeLead session in the current folder. In the session you type requests and chat commands, and CodeLead asks you before it changes anything.

## Name

`codelead interactive` - Starts a CodeLead interactive session.

## Synopsis

```console
codelead interactive [--json-io]
codelead interactive -h|--help
```

## Description

A [session](../concepts/sessions.md) is a conversation with CodeLead about one repository. In it you can:

- Ask about the code in plain words. Text without a leading `/` goes to the chat, which never changes code.
- Run [chat commands](../chat-commands/index.md), for example [`/implement`](../chat-commands/implement.md) to change code, [`/create-app`](../chat-commands/create-app.md) to build a new application, or [`/modernize`](../chat-commands/modernize.md).
- Answer the questions that CodeLead asks: "Apply this patch?", "Run validation now?".

Type `/help` to see the commands. Type `/exit` or `/quit` to leave.

The Visual Studio Code extension's chat runs the same session. See [The chat](../extension/chat.md).

> [!NOTE]
> **New to this?** Every question has a recommended answer. Press Enter to take it. A question about a step that is hard to undo has no answer for Enter: you type the answer. See [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md).

### The session log

Every session writes a log under `.codeleadsessions/<session id>/` in the project: each line you typed, each model call with its prompt and its response, each command's result. Use [`codelead sessions`](codelead-sessions.md) to list and export the logs. See [Export a session](../how-to/export-a-session.md).

### JSON lines for an editor

With `--json-io`, the session reads JSON lines on standard input and writes JSON lines on standard output. This is the protocol `codelead.session-io/1`. An editor, such as the Visual Studio Code extension, uses it to show the session as a chat. Standard output then holds only protocol lines, without colour. See [Session protocol](../reference/session-io-protocol.md).

> [!TIP]
> **For experienced users:** with `--json-io`, nobody can answer a consent question in the terminal. A remote provider must be allowed first: `privacy.allowRemoteModels`, or `CODELEAD_ALLOW_REMOTE_MODELS=1`. Otherwise the only line is `session.ended` with `reason: "error"` and the words.

### When the configuration is refused

When the [override rule](codelead-config.md#the-override-rule) refuses the environment's model, the session does not start, and the exit code is 2.

## Options

- **`--json-io`**

  Drives the session with JSON lines on standard input and standard output, for editors. The default is a terminal session.

## Examples

- Start a session in the terminal:

  ```console
  codelead interactive
  ```

  ```output
  ╭──────────────────────────────────────────────────────────────────────────────╮
  │                                                                              │
  │   CodeLead Interactive                                                       │
  │                                                                              │
  │   Repo: <your project>                                                       │
  │   Strategy: auto                                                             │
  │   Project type: existing                                                     │
  │   Ask about the code in plain words (the chat never changes code), or use    │
  │   /help for commands.                                                        │
  │   Use /exit or /quit to leave.                                               │
  │                                                                              │
  ╰──────────────────────────────────────────────────────────────────────────────╯
  codelead>
  ```

- Type `/help` in the session to list the commands:

  ```output
  │   <question>             Chat about this repository and CodeLead: how the    │
  │   code works, what CodeLead did to it and why, how to use its commands.      │
  …
  │   /scan <request>        Find likely relevant files without calling a        │
  │   model.                                                                     │
  …
  │   /implement <request>   Guide preflight, patch, check, edit, and            │
  │   validation.                                                                │
  …
  ```

- Drive a session with JSON lines, as an editor does. The client sends one command and then `exit`:

  ```console
  printf '{"type":"input","text":"/help"}\n{"type":"exit"}\n' | codelead interactive --json-io
  ```

  ```output
  {"type":"session.started","features":["chat-router"],"protocol":"codelead.session-io/1","sessionId":"session-2026-11-20T04-18-16-661Z","sessionDir":"<your project>/.codeleadsessions/session-2026-11-20T04-18-16-661Z","cwd":"<your project>","provider":"lmstudio","sources":{"provider":"file","model":"file","baseUrl":"env:CODELEAD_BASE_URL","apiKey":"file"},"plan":"free"}
  {"type":"commands","items":[{"command":"/patch","summary":"Generate a unified diff patch proposal without applying it.", …
  {"type":"output","text":"╭───…"}
  {"type":"command.started","commandId":"cmd-001","input":"/help","command":"help"}
  {"type":"output","text":"╭───…","commandId":"cmd-001"}
  {"type":"command.completed","commandId":"cmd-001","status":"completed","durationMs":66,"summary":"Processed help command.","result":{"outcome":"done","facts":[],"next":[]}}
  {"type":"command.started","commandId":"cmd-002","input":"/exit","command":"exit"}
  {"type":"output","text":"Exiting CodeLead.","commandId":"cmd-002"}
  {"type":"command.completed","commandId":"cmd-002","status":"exited","durationMs":0,"summary":"Exited interactive session.","result":{"outcome":"done","facts":[],"next":[]}}
  {"type":"session.ended","reason":"user_exit"}
  ```

- Ask about the code. Text without `/` goes to the chat. The answer depends on the model:

  ```console
  codelead> Where are the products saved?
  ```

  <!-- described, not captured -->
  ```output
  The chat answers from the repository: the controller that saves a product, the data context and the file and line of each, and the command to run when you want a change.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The session ended with `/exit`, `/quit` or the end of the input. |
| `1` | The session ended on an error. |
| `2` | The start was refused before anything ran: the override rule refused the configuration, or the privacy settings block a remote model. This holds at a terminal and with `--json-io`. With `--json-io`, the last line is `session.ended` with `reason: "error"`, and then the exit code is `2`. |

## Files

| File | What the session does with it |
|---|---|
| `.codelead/config.json`, `.env` | Reads the settings and the keys. |
| `.codeleadsessions/<session id>/session.jsonl` | Writes every event of the session. `CODELEAD_SESSIONS_DIR` moves the folder; `CODELEAD_DISABLE_SESSION_LOGGING=1` turns the log off. |
| `.codeleadsessions/<session id>/model/` | Writes each prompt and each response. |
| `.codelead/evidence/ev-*.json` | Each command that lands a change writes an evidence record. |

The commands that you run in the session read and write more files. Each chat command's page lists them.

## See also

- [Chat command reference](../chat-commands/index.md)
- [Sessions](../concepts/sessions.md)
- [The chat](../extension/chat.md): the same session in Visual Studio Code.
- [Session protocol](../reference/session-io-protocol.md): the `--json-io` lines.
- [`codelead sessions`](codelead-sessions.md): list and export the logs.
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
