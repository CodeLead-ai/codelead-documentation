---
title: Prompts and recommended answers
description: How CodeLead asks you questions, what the capital letter and the Enter key mean, which steps are hard to undo, and what happens when no answer can come.
kind: concept
area: concepts
date: 2026-11-20
---

# Prompts and recommended answers

CodeLead asks you before it does anything that changes your files or runs your commands. Every question has a **recommended answer**: the answer that CodeLead would give in your place. Press Enter to take it. A question about a step that is hard to undo has no recommended answer, and Enter gives nothing: you must type your answer.

**In short**
- In `[Y/n]` and `[y/N]`, the capital letter is the recommended answer, and Enter gives it.
- `[y/n]` with no capital marks a step that is hard to undo. Enter gives nothing, and CodeLead asks again: "Type y or n."
- A clarifying question shows a `Default:`. That default is the recommended answer, and Enter takes it.
- When no answer can come, for example when the input closes, a yes-or-no question is declined. Nothing is applied by an absent answer.

## Why it exists

> [!NOTE]
> **New to this?** A *prompt* is a question that waits for your answer. In the terminal, you type the answer and press Enter. In the extension's chat, you press a button.

Questions slow you down when every answer is the same. Questions are dangerous when Enter does the risky thing. CodeLead solves both: Enter always gives the answer that CodeLead recommends, and the recommendation is no wherever no is the safe choice. For the few steps that you cannot take back, Enter does nothing at all.

## How it works

### The bracket

| Bracket | CodeLead recommends | Enter gives | Example |
|---|---|---|---|
| `[Y/n]` | Yes | Yes | `Apply this patch? [Y/n]` after a patch check with no blocking finding |
| `[y/N]` | No, with the reason | No | `Mark validation as manually passed? [y/N]`: CodeLead did not see it pass |
| `[y/n]` | Nothing: the step is hard to undo | Nothing: "Type y or n." | `Delete the above and re-plan from scratch? [y/n]` |

Here is a real question of [`/validate`](../chat-commands/validate.md) on the Basic Store sample, as the session sends it to the extension. `recommended` and `emptyAnswer` are both `y`:

```json
{
  "type": "prompt",
  "promptId": "p-001",
  "question": "Run validation commands?\n\nValidation scope: repo root\n\nCommands:\n  1. MSBuild sampleapp.slnx (Visual Studio's MSBuild: not found)\n\nRun these commands? [Y/n]",
  "kind": "yes-no",
  "default": "y",
  "recommended": "y",
  "emptyAnswer": "y"
}
```

A recommended no carries its reason in `why`. Here is the real question for a template package that is not on CodeLead's trusted list:

```json
{
  "type": "prompt",
  "question": "Template: bringing in @example/create-app (an npm create package).\n\nRun `npm create @example/app .` from the registry? It is not on CodeLead's trusted list. [y/N]",
  "kind": "yes-no",
  "recommended": "n",
  "emptyAnswer": "n",
  "why": "it is not on CodeLead's trusted list"
}
```

### The common questions

| Question | Recommended | Why |
|---|---|---|
| `Apply this patch?` | Yes when the patch check passed with no blocking finding, otherwise no | A note never changes a yes. A blocking finding gives no, with "N blocking findings". |
| `Run validation now?` and `Run these commands?` | Yes | The gate decides whether the change is done. |
| `Run dependency install now?` | Yes | The patch changed `package.json`, and the rest of it waits on the install. |
| `Mark validation as manually passed?` | No | CodeLead did not see it pass. Say yes once you checked it yourself. |
| `Continue planning for this folder?` | No | The folder holds an existing project. |
| `Retry scaffold command?` | No | The last attempt failed. No falls back to files that a model writes. |
| `Run it now?` after the chat offers a command | Yes | The command asks again before it changes a file. |
| `Delete the above and re-plan from scratch?` | Nothing | It deletes files of the folder. |
| `Unapply it, putting those files back as they were before the patch?` | Nothing | The patch's changes are lost. |
| `Continue?` before code goes to a remote provider | Nothing | What is sent cannot be called back. |

### Clarifying questions and assumptions

The [preflight](requests-plans-and-patches.md) asks in two forms. A **clarifying question** says why it matters and shows a default:

<!-- described, not captured -->
```output
Clarification 1/2

Should the error also appear when the order is edited, or only when it is created?

Why it matters:
ValidateOrderAsync runs for both forms.

Default:
Both: the rule applies to every order.

Answer, press Enter to use default:
```

Type your own answer, or press Enter for the default. A list of **assumptions** comes as one question:

<!-- described, not captured -->
```output
CodeLead assumes:
1. The rule compares product ids, not product names. (risk: low)
2. The first line with a product stays valid; the error goes on the later line. (risk: low)

Press Enter to accept them all; answer no to reject them all; or name the number and what you want instead (for example "1: resume the plan").
```

Press Enter to accept all of them. Type `no` to reject all of them. Type `2: put the error on both lines` to change only the second.

### When no answer can come

Sometimes nobody can answer: the editor closed the session, the input of a script ended, or you pressed Ctrl+C. Then a yes-or-no question is declined, and nothing is applied by default. The session says so in one line. Here is a real one:

```output
No answer can come (stdin is closed): p-001 is declined (no answer; nothing is done by default).
```

Three answers that are neither yes nor no also decline: "No valid answer received; assuming no."

## What it is not

- **Not a way for CodeLead to decide for you.** The recommended answer is a suggestion. Your typed answer always wins.
- **Not the same in unattended runs.** A run that nobody watches answers by fixed rules, and it records each answer. See [Unattended runs and the run lock](unattended-runs.md).
- **Not asked again for every remote call.** CodeLead asks for the consent to send code once for each remote provider in a session.

> [!TIP]
> **For experienced users:** a pin or an in-place transform writes its own request, and its preflight asks nothing. It takes each default and shows the list once as "CodeLead assumed". The list is kept in the evidence record's `preflight.assumptions`.

## Details

- **The typed prompt**. Each question has a `kind`: `yes-no`, `choice`, `enter-default` or `text`. It has a `recommended` answer and an `emptyAnswer`, which is what Enter gives, or `null`. It can also have a `why`, and a `consequence` for a hard step. The session protocol's `prompt` line carries these fields. `default` stays for older clients. See [Session protocol](../reference/session-io-protocol.md).
- **Choices.** A numbered list marks the recommended option, and Enter gives it: `Choose validation scope [1-3], or press Enter for 1:`.
- **The session log.** The `user.input` event of an answer keeps the question's words, `recommended`, `emptyAnswer`, and `answered: false` when no answer came. Passwords in answers are redacted.
- **`--auto` and the unattended runners.** [`/create-app --auto`](../chat-commands/create-app.md) gives each question its recommended answer, also a no. It says each no in one line: `Automated build: answered no, as CodeLead recommends (<why>).` The unattended runners answer yes to apply, to validate and to run, and no to a manual pass.
- **The question mark.** A question prints one question mark, before the bracket: `Apply this patch? [Y/n]`.

## Related content

- [`/implement`](../chat-commands/implement.md), [`/edit`](../chat-commands/edit.md), [`/validate`](../chat-commands/validate.md)
- [`/create-app`](../chat-commands/create-app.md) and [`/build-step`](../chat-commands/build-step.md)
- [`/chat`](../chat-commands/chat.md)
- [`codelead interactive`](../cli/codelead-interactive.md) and [`codelead setup`](../cli/codelead-setup.md)
- [Requests, plans and patches](requests-plans-and-patches.md)
- [Privacy and remote models](privacy-and-remote-models.md)
- [Session protocol](../reference/session-io-protocol.md)
- [The chat](../extension/chat.md)
