---
title: Chat command reference
description: Every slash command you type in a CodeLead session, grouped by purpose, with how a command ends and how its questions work.
kind: overview
area: chat-commands
date: 2026-11-20
---

# Chat command reference

A CodeLead session is a conversation with CodeLead about one project folder. You type a request in plain words, or a chat command that starts with `/`, such as [`/implement`](implement.md) or [`/validate`](validate.md). This reference has one page for each chat command, with the same headings on every page.

**In short**
- You know the two places where you type chat commands, and what happens to text without a `/`.
- You can find the command for a task: read the code, change it, check it, repair it, build an app or modernize one.
- You know how every command ends: with one result that says what happened and what to do next.

## Choose where to start

| You are | Read first | Then |
|---|---|---|
| **New to CodeLead** | [`/implement`](implement.md) | [`/validate`](validate.md) and [`/repair`](repair.md) |
| **Already a user of coding agents** | [`/preflight`](preflight.md) and [`/copy`](copy.md) | [`/patch`](patch.md) and [`/edit`](edit.md) |
| **A team lead or an architect** | [`/pr`](pr.md) and [`/log`](log.md) | [`/modernize`](modernize.md) |

## Where you type chat commands

You can type the same commands in two places. Both run the same engine.

| Place | How to open it | Best for |
|---|---|---|
| The terminal | Run [`codelead interactive`](../cli/codelead-interactive.md) in the project folder. | Work in a terminal, scripts that drive a session, remote machines. |
| The chat in Visual Studio Code | Open the CodeLead extension and select the chat. See [The chat](../extension/chat.md). | Most people. Questions appear as buttons, and results appear as messages. |

> [!NOTE]
> **New to this?** A chat command is a word that starts with `/`, then options that start with `--`. For example, `/validate --scope repo` runs the command `/validate` with the option `--scope` and the value `repo`. Type `/help` at any time to see the list.

A session starts with a welcome box like this one:

```output
CodeLead Interactive

Repo: <your project>
Strategy: auto
Project type: existing
Ask about the code in plain words (the chat never changes code), or use
/help for commands.
Use /exit or /quit to leave.
```

## Text without a slash

Text that does not start with `/` goes to the chat about the code. For example, you type `how is the order total calculated?`. CodeLead reads the repository and answers with the files and lines that its answer rests on. The chat never changes code.

When your words ask for a change, the chat answers with the command to run, for example `/implement add a due date to tasks`. Then it asks `Run it now? [Y/n]`. The command asks again before it changes any file. See [`/chat`](chat.md) and [`/ask`](ask.md).

## The commands by purpose

### Understand the code

- [`/chat`](chat.md): asks a question about this repository, about what CodeLead did, or about CodeLead itself.
- [`/ask`](ask.md): the same as `/chat`; `/ask --record` writes the answer to a file.
- [`/scan`](scan.md): finds the files that a request is likely about, with no model call.
- [`/map`](map.md): reads the repository into its map: the stack, the validation commands, the modules.
- [`/lint`](lint.md): checks a request before you run it, with no model call.

### Plan and change code

- [`/implement`](implement.md): the whole change in one command: preflight, patch, patch check, apply, validation, evidence record.
- [`/preflight`](preflight.md): turns a request into a precise prompt for a coding agent.
- [`/patch`](patch.md): asks for a patch, shows it, checks it, or asks for a new one.
- [`/edit`](edit.md): applies the last checked patch, after it asks you.
- [`/strategy`](strategy.md): shows or sets how much planning the preflight does.

### Check and repair

- [`/validate`](validate.md): runs the project's validation commands and records the verdict.
- [`/repair`](repair.md): asks for a small patch that fixes the last failed validation.

### Build a new app

- [`/create-app`](create-app.md): plans a new app, and can build it step by step without stopping.
- [`/plan`](plan.md): shows, approves or revises the plan of the new app.
- [`/spec`](spec.md): shows the project specification of the new app.
- [`/build-step`](build-step.md): builds the next approved step of the plan.
- [`/build-incremental`](build-incremental.md): builds a new app one verified increment at a time.
- [`/greenfield`](greenfield.md): shows where the new app's plan and build stand.

### Modernize

- [`/modernize`](modernize.md): the modernization commands: the dossier, the pins, the transforms, the rebuild, the program and the audit pack.

### Project documents and hand-off

- [`/pin`](pin.md): lists what waits on you, adds an item, or marks one as done.
- [`/log`](log.md): shows the project log, or writes the project documents again from every session.
- [`/pr`](pr.md): opens a pull request for the last change that landed, with its evidence record as the body.

### Session and clipboard

- [`/session`](session.md): exports session logs, or pauses, resumes or discards an `/implement`.
- [`/copy`](copy.md): copies the last coding-agent prompt or the last patch to the clipboard.
- [`/paste`](paste.md): reads the clipboard into the session, shows it, clears it, or analyzes it.

### Help

- [`/help`](help.md): lists every command, or shows the usage lines of one command.
- [`/status`](status.md): shows the state of the session.
- [`/exit`](exit.md) and [`/quit`](quit.md): end the session.

## How every command ends

Every command ends with one result. The result has an outcome, a headline, up to five facts, and the way on. See [Results and stops](../concepts/results-and-stops.md).

| Outcome | Mark | Meaning |
|---|---|---|
| `passed` | ✔ | The verdict is a pass: validation passed, the patch check passed, the change landed. |
| `done` | ✔ | A command without a verdict did its job: a patch is ready, the map is written. |
| `failed` | ✖ | The verdict is a failure, or the command could not do its job. |
| `stopped` | ■ | The command stopped before its job, without a failure: you answered no, or a step must come first. |
| `nothing` | ○ | There was nothing to do: no patch to apply, no failed validation to repair. |

In the terminal, the commands with a verdict end with one closing line. These commands are `/validate`, `/repair`, `/edit`, `/patch`, `/implement`, `/modernize --pin` and the in-place `/modernize --transform`. For example:

```output
✖ Validation failed: npm test exited 1; next: /repair
```

In the chat in Visual Studio Code, the same result is one message at the end of the turn. The message shows the headline and the facts. Each next step that starts with `/` is a button. **Show details** opens the full output.

> [!TIP]
> **For experienced users:** the result is also in the field `result` of each `command.completed` line, in the session log and over JSON. A result with the outcome `failed` makes the status `failed`. See [Session protocol](../reference/session-io-protocol.md).

## How questions work

Some commands ask before they act: "Apply this patch?", "Run these commands?". The bracket shows what Enter gives.

- `[Y/n]`: CodeLead recommends yes. Enter gives yes.
- `[y/N]`: CodeLead recommends no, and says why. Enter gives no.
- `[y/n]`: the step is hard to undo. Enter gives nothing, and CodeLead asks again. You must type `y` or `n`.

In the chat, each question has buttons, and the recommended answer is the highlighted one. A session that cannot get an answer never applies anything by default. See [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md).

## The daily loop

Most changes to existing code follow these steps. [`/implement`](implement.md) runs steps 1 to 4 for you, and it asks before each step that changes something. You can also run each step by hand.

| Step | What happens | Inside `/implement` | By hand |
|---|---|---|---|
| 1 | A model plans the change and writes a patch. | Runs by itself. | `/preflight <request>`, then `/patch --from-preflight` |
| 2 | The patch check reads the patch against the files. | Runs by itself. | `/patch --check` |
| 3 | CodeLead applies the patch to the files. | Asks `Apply this patch? [Y/n]`. | `/edit` |
| 4 | CodeLead runs the build and the tests. | Asks `Run validation now? [Y/n]`. | `/validate` |
| 5 | Only when validation failed: a small patch that fixes the failure. | Not part of it. | `/repair`, then `/edit` and `/validate` |

## See also

- [Quickstart: change existing code](../get-started/quickstart-existing-code.md)
- [How CodeLead works](../concepts/how-codelead-works.md)
- [Command-line reference](../cli/index.md)
- [The chat](../extension/chat.md)
- [Sessions](../concepts/sessions.md)
- [Glossary](../concepts/glossary.md)
