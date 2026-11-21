---
title: Session protocol
description: The line protocol codelead.session-io/1 between CodeLead and a host such as the VS Code extension - every message in both directions with its fields, the command result, and the prompt model with recommended, emptyAnswer, why and consequence.
kind: reference
area: reference
date: 2026-11-20
---

# Session protocol

**Applies to:** CodeLead 0.1 and later

The session protocol, `codelead.session-io/1`, lets a program drive a CodeLead session and show it in its own way. The [chat of the extension](../extension/chat.md) uses it. It is the same [interactive session](../cli/codelead-interactive.md) that a terminal runs, driven by JSON lines on standard input and answered by JSON lines on standard output. This page lists every message in both directions, with real lines.

> [!NOTE]
> **New to this?** You need this page only to write a host: an editor plug-in, a test harness, a script. To use CodeLead in VS Code, see [The chat](../extension/chat.md).

## The transport

| Fact | Value |
|---|---|
| Start | `codelead interactive --json-io`, in the project's folder. |
| Framing | One JSON object per line, UTF-8, ended by `\n`. A `\r` before the `\n` is dropped. |
| Standard output | Protocol lines only. |
| Standard input | The host's lines. |
| Colours | The session strips terminal escapes from the text that it sends. The extension also starts it with `FORCE_COLOR=0` and `NO_COLOR=1`. |
| Order | The host's `input` lines run one at a time, in order. |
| End | `exit`, or the end of standard input. |

A typical exchange:

```text
host → CodeLead                 CodeLead → host
                                session.started
                                commands
                                output            (the banner)
input /validate
                                command.started
                                prompt p-001
answer p-001 "y"
                                output …
                                command.completed  (with the result)
exit
                                command.started    (/exit)
                                command.completed  (exited)
                                session.ended      (user_exit)
```

## Lines that the host writes

| `type` | Fields | What it does |
|---|---|---|
| `input` | `text` | Queues one line, as if it were typed: a command such as `/validate`, or free text for the chat. |
| `answer` | `promptId`, `text` | Answers the open question. An answer to another `promptId` is dropped. |
| `interrupt` | none | Answers the open question with no answer, and stops the servers that a walk started. A command in flight goes on to its end. |
| `exit` | none | Queues `/exit`. Every later `input` is dropped. |

```json
{"type":"input","text":"/validate"}
{"type":"answer","promptId":"p-001","text":"y"}
{"type":"interrupt"}
{"type":"exit"}
```

A line that is not of the protocol is dropped, and the session says so in an `output` line:

```json
{"type":"output","text":"Dropped a line that is not of the protocol (the type \"bogus\" is not of the protocol): {\"type\":\"bogus\"}"}
{"type":"output","text":"Dropped a line that is not of the protocol (not JSON): not json"}
{"type":"output","text":"Dropped the answer to p-009: no question is open."}
{"type":"output","text":"Interrupt: nothing is running."}
```

## Lines that CodeLead writes

| `type` | When |
|---|---|
| [`session.started`](#sessionstarted) | First, once. |
| [`session.moved`](#sessionmoved) | The session's folder moved. |
| [`commands`](#commands) | Once, after `session.started`: the catalog of commands. |
| [`output`](#output) | Text that the session writes. |
| [`command.started`](#commandstarted-and-commandcompleted) | A command starts. |
| [`command.completed`](#commandstarted-and-commandcompleted) | A command ends, with its result. |
| [`prompt`](#prompt) | The session asks a question. |
| [`spinner`](#spinner) | A long step starts, moves on, or ends. |
| [`model.request`, `model.response`, `model.error`](#model-lines) | A model call. |
| [`problem`](#problem) | Something stopped short of its goal, with the way out. |
| [`session.ended`](#sessionended) | Last, once. |

### session.started

| Field | Type | What it holds |
|---|---|---|
| `protocol` | string | `codelead.session-io/1`. |
| `sessionId` | string | The session's id, such as `session-2026-11-20T07-18-21-503Z`. Empty when the start is refused. |
| `sessionDir` | string | The session's log folder. Empty when the start is refused. |
| `cwd` | string | The project's folder. |
| `provider`, `model` | string | The default role's provider and model. `model` is absent when no model is named. |
| `sources` | object | Where the default role's `provider`, `model`, `baseUrl` and `apiKey` came from: `file`, `default`, `env:<VARIABLE>`, `ladder`, or `none` for a key. See [Configuration file](configuration.md#the-codelead-config-command). |
| `features` | list of strings | What this CodeLead offers the host. `chat-router`: free text goes to the chat. |
| `plan` | `free` or `pro` | The [licence's plan](../concepts/licence-and-plans.md) of the folder. |

```json
{"type":"session.started","features":["chat-router"],"protocol":"codelead.session-io/1","sessionId":"session-2026-11-20T07-18-21-503Z","sessionDir":"<your project>/.codeleadsessions/session-2026-11-20T07-18-21-503Z","cwd":"<your project>","provider":"lmstudio","sources":{"provider":"file","model":"file","baseUrl":"env:CODELEAD_BASE_URL","apiKey":"file"},"plan":"free"}
```

A key's value is never sent. `sources.apiKey` says only where the key comes from.

### session.moved

`sessionDir`: the new folder. A new app's build moves its session into the app's folder once the scaffold is made.

### commands

`items`: one entry for each chat command, for completion in the host. Each entry:

| Field | What it holds |
|---|---|
| `command` | The command, such as `/validate`. |
| `summary` | What it does, in one line. |
| `options` | Each option: `flag`, `value` (what follows it: `none`, `path`, `card`, `spec`, `strategy`, `command`, `session`, `evidence`, `branch` or `text`), `values` (the words that it takes, when they are few) and `usage`. |
| `usage` | Each usage line: `line` and `meaning`. |

```json
{"command":"/validate","summary":"Run detected project validation commands.","options":[{"flag":"--scope","value":"path","values":["repo","current"],"usage":"/validate --scope repo"},{"flag":"--override","value":"none","usage":"/validate --override"}],"usage":[{"line":"/validate","meaning":"Run detected project validation commands."},{"line":"/validate --scope repo","meaning":"Validate at repo root."}]}
```

### output

| Field | What it holds |
|---|---|
| `text` | The text, without terminal escapes. It can hold several lines. |
| `commandId` | The command that wrote it. Absent for the session's own text, such as the banner. |
| `block` | The named block that the text belongs to, such as `validation`, `patch-check` or `apply`. A result's `detail.block` points to it. |

```json
{"type":"output","text":"Running 2/2: tests","commandId":"cmd-002"}
```

### command.started and command.completed

`command.started`:

| Field | What it holds |
|---|---|
| `commandId` | `cmd-001`, `cmd-002`, and so on. |
| `input` | The line as it was sent. |
| `command` | The kind: `validate`, `implement`, `chat`, `exit`, `unknown`, and so on. |

`command.completed`:

| Field | What it holds |
|---|---|
| `commandId` | The same id. |
| `status` | `completed`, `failed`, `cancelled` or `exited`. |
| `durationMs` | How long the command ran. |
| `summary` | One line, for older hosts. |
| `result` | The command's result. See [The command result](#the-command-result). |

The status is `failed` when the result's outcome is `failed`, or when the command sent a `problem` that is not information. `exited` comes after `/exit`.

```json
{"type":"command.started","commandId":"cmd-002","input":"/validate","command":"validate"}
{"type":"command.completed","commandId":"cmd-002","status":"failed","durationMs":102,"summary":"Processed validate command.","result":{"outcome":"failed","headline":"Validation failed: tests exited 1","facts":["Failed command: tests (exit 1)","Scope: repo root","Ran: 1 of 2 commands passed before it"],"next":["/repair","Or fix it by hand, then /validate"],"detail":{"block":"validation"}}}
```

### prompt

| Field | Type | What it holds |
|---|---|---|
| `promptId` | string | `p-001`, `p-002`, and so on. The `answer` names it. |
| `commandId` | string | The command that asks. |
| `question` | string | The words just before the question and the question itself, at most 40 lines, without escapes. |
| `kind` | string | `yes-no`, `choice`, `enter-default` or `text`. |
| `options` | list | For a `choice`: each `value` and `label`, and `recommended: true` on at most one. |
| `recommended` | string | The answer that CodeLead recommends, when it recommends one. |
| `emptyAnswer` | string or `null` | What an empty answer (Enter) gives. `null`: the step is hard to undo, and an empty answer gives nothing. |
| `why` | string | The reason for the recommendation, when it is not obvious. |
| `consequence` | string | For a step that is hard to undo: what it does. |
| `default` | string | For older hosts: the same as `emptyAnswer`, absent when that is `null`. |

See [The prompt model](#the-prompt-model).

```json
{"type":"prompt","promptId":"p-001","commandId":"cmd-002","question":"Run validation commands?\n\nValidation scope: repo root\n\nCommands:\n  1. build\n  2. tests\n\nRun these commands? [Y/n]","kind":"yes-no","default":"y","recommended":"y","emptyAnswer":"y"}
```

### spinner

| Field | What it holds |
|---|---|
| `state` | `start`, `update`, `succeed`, `fail` or `stop`. |
| `agent` | The agent of a model call, such as `preflight`. |
| `text` | The words, without the counts that a terminal adds. |
| `elapsedMs`, `tokens` | On an `update` of a model call, every 3 seconds: the time so far and the tokens so far. |

```json
{"type":"spinner","state":"start","agent":"preflight","text":"Asking model to analyze the request..."}
{"type":"spinner","state":"fail","agent":"preflight","text":"Asking model to analyze the request..."}
```

A `prompt` stops every running spinner. The spinners start again after the answer.

### Model lines

`model.request`, `model.response` and `model.error` come from the session log.

| Field | What it holds |
|---|---|
| `agent` | The agent: `preflight`, `patch`, `repair`, `chat`, and so on. |
| `provider`, `model` | The provider and the model, when known. |
| `promptPath`, `responsePath` | The prompt and the answer in the session's folder, such as `model/preflight-prompt-001.md`. |
| `message` | The error's words, on `model.error`. |

```json
{"type":"model.request","agent":"preflight","provider":"lmstudio","promptPath":"model/preflight-prompt-001.md"}
{"type":"model.error","agent":"preflight","provider":"lmstudio","promptPath":"model/preflight-prompt-001.md","message":"Could not connect to lmstudio at http://127.0.0.1:9/v1.\nIf this is LM Studio, make sure it is open and the Local Server is running."}
```

A `model.error` becomes a `problem` too, unless a `model.response` follows it first: a retry that got an answer is no problem. The refusal of the question before a remote call is not sent twice: its own `problem` says it.

### problem

| Field | What it holds |
|---|---|
| `commandId` | The command, when one runs. |
| `title` | What stopped, in a few words. |
| `message` | Why. |
| `next` | The ways out, one line each. |
| `retry` | The line that runs it again. By default, the command as it was typed. |
| `kind` | `information` for a card that tells and stops nothing, such as what a pin's preflight assumed. It has no `retry`, and does not make its command `failed`. |

```json
{"type":"problem","commandId":"cmd-003","title":"The model call failed","message":"Could not connect to lmstudio at http://127.0.0.1:9/v1.","next":["If this is LM Studio, make sure it is open and the Local Server is running."],"retry":"/preflight Show the stock on the order form"}
```

A host shows a problem as a card with the title, the message, the ways out and a **Retry** button. See [Messages and what to do](messages.md).

### session.ended

| Field | What it holds |
|---|---|
| `reason` | `user_exit` (after `/exit`), `eof` (standard input ended) or `error`. |
| `message` | The error's words, on `error`. |

```json
{"type":"session.ended","reason":"user_exit"}
```

## The command result

Every command ends with one result, in the person's terms. It is the `result` of `command.completed`, and the same object in the session log's `command.completed` event.

| Field | Type | What it holds |
|---|---|---|
| `outcome` | string | `passed`, `failed`, `stopped`, `done` or `nothing`. |
| `headline` | string | One sentence, such as `Validation failed: tests exited 1`. Absent only on `done` for a command whose output is the answer itself, such as `/help`. |
| `facts` | list of strings | At most five short lines: the failing command, the first error, the files, the counts. |
| `next` | list of strings | The way on, one line each: a command to type, such as `/repair`, or a short instruction. Empty when nothing is next. |
| `detail` | object | Where the full words are: `block`, the output block, and `file`, a record such as an evidence record. |

| `outcome` | Meaning | Examples |
|---|---|---|
| `passed` | The verdict is a pass. | `Validation passed: 2 commands`, `Patch check passed: 2 files can apply cleanly`, `The change landed: …` |
| `failed` | The verdict is a failure, or the job could not be done. | `Validation failed: tests exited 1`, `The patch check refused the patch: …`, `Unknown command: /nope` |
| `stopped` | It stopped short of its job without a failed verdict: you answered no, or a step must come first. | `Validation not run: the answer was no`, `The patch passed its check and was not applied` |
| `done` | A command with no verdict did its job. | `/status`, a patch that waits for its check, `A patch is ready: …` |
| `nothing` | There was nothing to do. | `Nothing to repair: no failed validation in this session`, `No validation commands found` |

Real results:

```json
{"outcome":"nothing","headline":"Nothing to repair: no failed validation in this session","facts":[],"next":["/validate"]}
{"outcome":"stopped","headline":"Validation not run: the answer was no","facts":["Scope: repo root"],"next":["/validate"]}
{"outcome":"failed","headline":"Unknown command: /nope","facts":[],"next":["/help"]}
```

See [Results and stops](../concepts/results-and-stops.md).

## The prompt model

CodeLead marks the answer that it recommends, everywhere. Enter gives the recommended answer, except for a step that is hard to undo: there Enter gives nothing, and you must choose.

| Field | A yes-no that CodeLead recommends | A step that is hard to undo |
|---|---|---|
| `kind` | `yes-no` | `yes-no` |
| `recommended` | `y` or `n` | absent |
| `emptyAnswer` | the same as `recommended` | `null` |
| `default` | the same | absent |
| `consequence` | absent | what the step does |
| The terminal's bracket | `[Y/n]` or `[y/N]` | `[y/n]` |

A hard-to-undo question, such as the one at `/exit` with a rebuild part whose gate has not passed:

<!-- described, not captured -->
```json
{"type":"prompt","promptId":"p-004","commandId":"cmd-012","question":"…\nUnapply it, putting those files back as they were before the patch? [y/n]","kind":"yes-no","emptyAnswer":null,"consequence":"this puts 3 files back as they were before the patch; the patch's changes are lost"}
```

- **Enter on a hard-to-undo question** asks it again: `Type y or n.`
- **Three answers that are neither yes nor no** decline: `No valid answer received; assuming no.`
- **No answer** is not an empty answer. When standard input ends, or on `interrupt`, a typed yes-no is declined: nothing is applied by default. Any other question takes its empty answer, `""`. The session says which:

```json
{"type":"output","text":"Interrupt: the open question p-001 is declined (no answer; nothing is done by default).","commandId":"cmd-001"}
```

- **A `choice`** marks its recommended option: `{"value":"1","label":"…","recommended":true}`.
- **An `enter-default`** question, such as a clarification of the preflight, recommends its default, and Enter gives it.

A question that a command asks without a typed model is classified by its words:

- a numbered list with `[1-N]` or "Choose" is a `choice`;
- `[y/N]`, `(yes/no)` or "answer yes or no" is a `yes-no`;
- "press Enter to use default" is an `enter-default`;
- anything else is `text`.

Such a line has `default` only, and no `recommended`.

See [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md).

## A refused start

When the configuration is refused, the session does not start. No session folder is made, and no model is called.

The [override rule](environment-variables.md#the-override-rule): `session.started` with an empty `sessionId`, the refusal as a `problem`, then `session.ended`:

```json
{"type":"session.started","protocol":"codelead.session-io/1","sessionId":"","sessionDir":"","cwd":"<your project>","provider":"lmstudio","model":"qwen/qwen3.6-35b-a3b","sources":{"provider":"file","model":"env:CODELEAD_MODEL","baseUrl":"file","apiKey":"file"}}
{"type":"problem","title":"The model from the environment was refused","message":"CODELEAD_MODEL names qwen/qwen3.6-35b-a3b, but the repair role's provider is anthropic (the default role's is lmstudio): a generic model variable does not apply across providers.","next":["Unset CODELEAD_MODEL (codelead doctor shows where it is set).","Or set CODELEAD_MODEL_REPAIR for that role.","Or remove the role from the file.","Then start the chat again: the environment is read when CodeLead starts."]}
{"type":"session.ended","reason":"error","message":"CODELEAD_MODEL names qwen/qwen3.6-35b-a3b, but the repair role's provider is anthropic (the default role's is lmstudio): a generic model variable does not apply across providers. Unset CODELEAD_MODEL, or set CODELEAD_MODEL_REPAIR for that role, or remove the role from the file."}
```

A remote provider that the privacy settings block: one `session.ended` line only, with the reason and the way out:

```json
{"type":"session.ended","reason":"error","message":"A remote model is configured but the privacy settings block remote models, so the session does not start. Turn on \"Allow remote models\" in the settings (privacy.allowRemoteModels in .codelead/config.json), or set CODELEAD_ALLOW_REMOTE_MODELS=1. the repair role goes to anthropic, a remote provider the privacy settings block: once the code's owner has agreed that it may be sent there, set CODELEAD_ALLOW_REMOTE_MODELS=1 (the consent an unattended run gives, in the environment or the .env), or privacy.allowRemoteModels in .codelead/config.json."}
```

The process ends with exit code 2 in both cases, after the last line: nothing ran. A host reads `session.ended` to know the reason.

## Rules for a host

- Read standard output line by line, and parse each line as JSON. Pass over a `type` that you do not know: a later release can add one.
- Pass over a field that you do not know.
- Answer a `prompt` with its `promptId`. Show `recommended`, never a `default` that is not recommended.
- Treat `command.completed` with `status: "failed"` as a failure, and show its `result`.
- Send `exit` to end. Close standard input only after it.

## See also

- [`codelead interactive`](../cli/codelead-interactive.md)
- [The chat](../extension/chat.md) in the extension
- [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md)
- [Results and stops](../concepts/results-and-stops.md)
- [Sessions](../concepts/sessions.md)
- [Messages and what to do](messages.md)
