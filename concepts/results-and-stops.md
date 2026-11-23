---
title: Results and stops
description: How every CodeLead command ends, what the five outcomes mean, and a table of the common stops with the way on for each.
kind: concept
area: concepts
date: 2026-11-20
---

# Results and stops

Every CodeLead command ends with one **result**: the outcome in a word, one sentence that says what happened, a few facts, and the way on. A **stop** is a run that ends before its job is done, for example before a change lands. A stop always says why, and what you can do next.

**In short**
- A result has an **outcome** (`passed`, `failed`, `stopped`, `done` or `nothing`), a **headline**, up to five **facts** and **the way on**.
- In the terminal, the result is the last line, with a mark: ✔ ✖ ■ ○. In the chat, it is a message with buttons for the next commands.
- Every stop names its reason and the way out. When you see no way out, that is a defect of CodeLead: report it.
- The table below lists the common stops, what each means and what to do.

## Why it exists

> [!NOTE]
> **New to this?** Read the last line of a command first. It says whether the command worked. When it did not, the words after `next:` are the first thing to try.

A command that prints a hundred lines and ends with "Done" leaves you to find out whether it worked. A failed test can hide in the middle of the output. CodeLead's rule is that the closing words of a command state its outcome, never only that it ran.

The same result goes to three places: your terminal, the extension's chat, and the session log. A script, a person and a reviewer all read the same verdict.

## How it works

### The five outcomes

| Outcome | Mark | Meaning | Example headline |
|---|---|---|---|
| `passed` | ✔ | The verdict is a pass. | `Validation passed: 2 commands` |
| `failed` | ✖ | The verdict is a failure, or the job could not be done. | `Validation failed: npm test exited 1` |
| `stopped` | ■ | It stopped before its job, with no failed verdict: you said no, or a step must come first. | `The change was not applied: the answer was no; nothing is changed` |
| `done` | ✔ | A command without a verdict did its job. | `A patch is ready: 1 file (+3 −1); nothing is changed yet` |
| `nothing` | ○ | There was nothing to do. | `Nothing to repair: no failed validation in this session` |

### A real result

Here is the result of [`/validate`](../chat-commands/validate.md) on the Basic Store sample on a Mac, as the session sends it to the extension. The gate cannot run, because Visual Studio's MSBuild is missing:

```json
{
  "outcome": "failed",
  "headline": "Validation failed: MSBuild could not run: Visual Studio's MSBuild is needed and was not found",
  "facts": [
    "Failed command: MSBuild sampleapp.slnx (Visual Studio's MSBuild: not found)",
    "First error: sampleapp.slnx needs Visual Studio's MSBuild: the solution holds a .NET Framework web project that the .NET SDK cannot build (webapp: non-SDK, v4.5, imports Microsoft.WebApplication.targ…",
    "Scope: repo root"
  ],
  "next": [
    "Run it on Windows with Visual Studio or its Build Tools (the web development build tools workload): …",
    "Or declare this scope's commands in its own .codelead/validation.json: …",
    "/validate"
  ],
  "detail": { "block": "validation" }
}
```

In the terminal, the same result is one closing line:

```output
✖ Validation failed: MSBuild could not run: Visual Studio's MSBuild is needed and was not found; next: Run it on Windows with Visual Studio or its Build Tools …
```

### Problem cards

A failure that you must act on also comes as a **problem**: a title, the reason, the ways out and the command to try again. In the chat it is a card with a **Retry** button. Here is the real problem of a new app's plan when the model server does not answer:

```json
{
  "type": "problem",
  "title": "Failed to create greenfield plan",
  "message": "Could not connect to lmstudio at http://127.0.0.1:9/v1.\nIf this is LM Studio, make sure it is open and the Local Server is running.",
  "next": ["Send the request again (Retry): the planner is asked again", "Or a larger model for the planner: Settings → Model roles"],
  "retry": "/create-app --template ../task-shell --plan a task manager app with a list of tasks and a done checkbox"
}
```

### The common stops

| Stop | What it means | The way on |
|---|---|---|
| `Could not connect to lmstudio at …` | The model server does not answer. Nothing was sent and nothing changed. | Start the model server. Check it with [`codelead doctor`](../cli/codelead-doctor.md). Run the same command again. |
| `Remote model blocked by privacy settings.` | The role points to a remote provider, and remote models are not allowed. | Allow remote models when the code's owner agrees. See [Privacy and remote models](privacy-and-remote-models.md). |
| `The change did not land: the patch check refused the patch` | The [patch check](the-patch-check.md) found a blocking problem, also after the retries. | Read the problem. Run [`/patch --retry`](../chat-commands/patch.md), or say more in the request: the file and the function. |
| `The change was not applied: the answer was no; nothing is changed` | You answered no to `Apply this patch?`. The patch is kept in the session. | Apply it as it is with [`/edit`](../chat-commands/edit.md), look at it with `/patch --show`, or ask again. |
| `The change was applied (2 files); not validated yet` | The patch is in your files, and the gate did not run. | Run [`/validate`](../chat-commands/validate.md). |
| `The change was applied, but validation failed: npm test exited 1` | The gate refused the change on the code. | Run [`/repair`](../chat-commands/repair.md), or fix the code by hand and run `/validate`. |
| `Validation failed: MSBuild could not run: …` | A tool that the gate needs is missing. A repair cannot help. | Install the tool, build on the right system, or declare the commands in `.codelead/validation.json`. |
| `Validation failed: dotnet test timed out` | The command ran out of time. The result is inconclusive. | Run `/validate` again. |
| `The run ended after the preflight: the cause is not found, so no patch was made` | You asked to fix a bug, and the preflight did not find its cause. | Name the cause with its file and line in the request, or add `--force`. |
| `The model's reply was cut at its cap: …` | The model's answer stopped at its token limit. Nothing of it was applied, in a change and in a step of a new app. | Ask for less in one request, or raise the cap. Sometimes CodeLead tries once more by itself. |
| `The repair loop stopped: this repair returns <file> to its state after repair 1, …` | The repairs go in circles. | Fix the file by hand, then run `/validate`. See [Repair](repair.md). |
| `Automated build stopped: step <id> is blocked.` | A step of a new app could not be built. | Send the request again (Retry) to continue, revise the plan with [`/plan --revise`](../chat-commands/plan.md), or build one step with [`/build-step`](../chat-commands/build-step.md). |
| `This clone is in use: …` | Another unattended run holds the run lock of this folder. | Wait for it to end, or stop it. See [Unattended runs and the run lock](unattended-runs.md). |
| `The automated build stopped: <what the step does>, and nobody is here to answer` | An automated build or an unattended run came to a step that is hard to undo. Nobody was there, so CodeLead did not answer. The outcome is `stopped`. | Use the setting that allows the step, when the words name one. Or run the same command where a person answers: at the terminal or in the chat. |
| `The automated build did not start: <provider> was not allowed (the answer was no), …` | You answered no to the consent for a remote model at the start. Nothing was sent, and nothing ran. | Pick a local model for that role, allow remote models, or send the command again and answer `y`. See [Privacy and remote models](privacy-and-remote-models.md). |
| `The modernization program is in CodeLead's Pro plan; …` | A modernization command ran in a folder without the Pro plan. Nothing ran. | Bring a licence of the Pro plan into the folder. See [Licence and plans](licence-and-plans.md). |

The full list is on [Messages and what to do](../reference/messages.md). The steps for each case are in [Fix a stop](../how-to/fix-a-stop.md).

## What it is not

- **Not the full output.** The result is a summary. The full words are one click away: **Show details** in the chat, or the block above the closing line in the terminal.
- **Not a judgement of you.** `stopped` is the outcome of a "no": your own, or the recommended no that [`/create-app --auto`](../chat-commands/create-app.md) gives. It is also the outcome of a run that stopped at a step that is hard to undo, because nobody was there to answer. It is not shown as a failure.
- **Not always the end of the work.** Many stops wait for one command from you, and then the same task continues.

## Details

- **Exit codes**. On the command line, `0` means done, and a check found nothing to fix. `1` means a failure of a check, a gate or a step. `2` means refused before anything ran. `130` means Ctrl+C. See [Exit codes](../reference/exit-codes.md).
- **The closing line** is printed for the commands with a verdict: `/validate`, `/repair`, `/edit`, `/patch`, `/implement`, and a pin or an in-place transform. It is left out when the command's last line already says the same.
- **In the session log,** the result is the `result` field of the `command.completed` event, and a failure sets `status: "failed"`. The session protocol carries it the same way. See [Session protocol](../reference/session-io-protocol.md).
- **Composed requests** (the pins and the in-place transforms) name their stop by class: `model_unreachable`, `declined_patch`, `declined_validation`, `applied_not_validated`, `reply_cut`, `check_refused`, `gate_refused`, `repair_stopped`, `precondition`, `objection`, `other`. A declined patch or validation is `stopped`, not `failed`. Only `other` offers a stronger model.
- **The modernization program** keeps each stage's status: `pending`, `running`, `waiting`, `done`, `failed` or `skipped`. A stage that needs you has a problem with a title, a reason, a fix and a retry command. See [The modernization program](../modernization/the-program.md).
- **Every stop writes an evidence record** with its stop reason. See [Evidence records](evidence-records.md).

## Related content

- [`/implement`](../chat-commands/implement.md), [`/validate`](../chat-commands/validate.md), [`/repair`](../chat-commands/repair.md), [`/patch`](../chat-commands/patch.md), [`/edit`](../chat-commands/edit.md)
- [`/create-app`](../chat-commands/create-app.md) and [`/build-step`](../chat-commands/build-step.md)
- [`codelead doctor`](../cli/codelead-doctor.md)
- [Messages and what to do](../reference/messages.md)
- [Exit codes](../reference/exit-codes.md)
- [Fix a stop](../how-to/fix-a-stop.md)
- [Prompts and recommended answers](prompts-and-recommended-answers.md)
- [The chat](../extension/chat.md)
