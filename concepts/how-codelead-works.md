---
title: How CodeLead works
description: The one page to read first. The loop that every change goes through, from your request to the evidence record, and what CodeLead is not.
kind: concept
area: concepts
date: 2026-11-20
---

# How CodeLead works

CodeLead lets AI models write code for you, under rules that you control. A model writes a change, CodeLead checks it, you approve it, your own build and tests judge it, and CodeLead records what happened. This page explains that loop. Every other page builds on it.

**In short**
- Every change goes through the same loop. A model proposes a [patch](requests-plans-and-patches.md) (the changes to the files), CodeLead checks it, and you approve it.
- A change is done only when your project's own build and tests pass. CodeLead calls these commands [the gate](validation-and-the-gate.md). A change that passes is a **landing**.
- Every run ends with an [evidence record](evidence-records.md): a file that says what was asked, what changed and what the checks said.
- CodeLead is local-first: by default the model runs on your machine or your network, and nothing leaves it.

## Why it exists

> [!NOTE]
> **New to this?** A *model* (a large language model) is a program that writes text and code from a request. A *model server* runs a model and answers requests from other programs, for example LM Studio on your computer.

Coding agents (AI tools that edit code for you) can write code fast. They can also change the wrong files, break a build, or claim that a change works when it does not. A person then has to read everything again.

CodeLead puts a fixed process around the model. The model does the writing. CodeLead decides what the model sees, checks what it writes, asks you before anything changes, and runs your project's own checks. At the end, a record says what was asked, what changed and what the checks said. You can trust the result because the process does not depend on the model telling the truth.

## How it works

### The loop

Here is the loop for one change to existing code:

```text
  your request
       |
       v
  preflight -----------> clarifying questions or assumptions
       |                 (you answer, or press Enter for the recommended answer)
       v
  plan
       |
       v
  patch  (a model writes it)
       |
       v
  the patch check  (no model) --- refused ---> a new patch, at most 2 times
       |
       v
  your approval:  "Apply this patch? [Y/n]"
       |
       v
  apply
       |
       v
  the gate  (your build and tests) --- refused ---> repair round:
       |                                            /repair, /edit, /validate
       v
  landing
       |
       v
  evidence record  (written at every end: landed or stopped)
```

The same steps, one line each:

| Step | Who does it | What happens |
|---|---|---|
| Request | You | You say what must change, in plain words. |
| [Preflight](requests-plans-and-patches.md) | CodeLead and a model | CodeLead finds the files that the request is about. A model reads them and the request. When something is not clear, it asks you. |
| Plan | A model | The preflight's result: the files to change, what must stay as it is, and how to check the change. |
| [Patch](requests-plans-and-patches.md) | A model | The changes to the files, as a diff: the lines to add and the lines to remove. Nothing is changed yet. |
| [The patch check](the-patch-check.md) | CodeLead, no model | Does every change fit the files as they are now? Is every path safe? Does the patch stay inside the plan? |
| Your approval | You | CodeLead shows the check's result and recommends an answer. Only a yes changes files. |
| Apply | CodeLead | The patch is written to your files. |
| [The gate](validation-and-the-gate.md) | Your project's commands | Your build and your tests run. A pass is a landing. |
| [Repair](repair.md) | A model, then CodeLead and you | When the gate refuses, a repair round asks for a small fix, checks it, applies it after your yes, and runs the gate again. |
| [Evidence record](evidence-records.md) | CodeLead | A file that says what was asked, what changed and what the checks said. CodeLead writes one at every end. |

### A small example

You type a request in a session:

```console
/implement add a clearCompleted function to src/tasks.js that removes the done tasks, with a test
```

The session prints each step as it runs, and it stops to ask you twice:

<!-- described, not captured -->
```output
Step 1/6: Running preflight...
Step 1/6: Preflight complete.
Step 2/6: Generating patch from preflight...
Step 3/6: Checking patch...
Step 5/6: Patch check passed.
…
The patch check passed. CodeLead recommends applying.

Apply this patch? [Y/n]
Patch applied successfully.
…
Run validation now? [Y/n]
…
Evidence record: <your project>/.codelead/evidence/ev-….json
✔ The change landed: applied to 2 files and validation passed
```

The last line is the **result**. Every command ends with one. It says the outcome in your terms, and it says what to do next when something is left. See [Results and stops](results-and-stops.md).

### Two kinds of work

- **A change to existing code.** [`/implement`](../chat-commands/implement.md) runs the loop above once.
- **A new application.** [`/create-app`](../chat-commands/create-app.md) and [`/build-incremental`](../chat-commands/build-incremental.md) plan the app as steps or increments. Each step or increment goes through the same loop: a patch, the patch check, the gate.

The [modernization program](../modernization/index.md) uses the same loop too. It first records what the old application does, then changes it.

### Three ways in

All three run the same CodeLead:

- [CodeLead for VS Code](../extension/index.md): buttons, a chat and the board inside the editor. Most people start here.
- A terminal session with [`codelead interactive`](../cli/codelead-interactive.md): you type requests and [chat commands](../chat-commands/index.md).
- The [command line](../cli/index.md): one command with a clear end, for scripts and long unattended runs.

### Local-first

By default, CodeLead calls a model server on your machine or on your network. Your code, your prompts and your logs stay there. A remote provider, such as a hosted model on the internet, is your explicit choice. CodeLead refuses to call one until you give your consent. See [Privacy and remote models](privacy-and-remote-models.md).

## What it is not

- **Not an autonomous agent that edits freely.** A model never writes to your files. It proposes a patch. CodeLead checks it, and only your yes applies it. An unattended run (a run that nobody watches) answers for you, but by fixed rules, and it records every answer.
- **Not a model.** CodeLead contains no model. It works with the model that you choose: a local server, or a key for a remote provider. See [Models and roles](models-and-roles.md).
- **Not a replacement for your build and tests.** CodeLead runs your project's own commands. When your project has no tests, the gate can only say that it builds.
- **Not a cloud service.** There is no account and no CodeLead server. No data about your use is sent anywhere. The licence check works with no network. See [Licence and plans](licence-and-plans.md).

> [!NOTE]
> **For team leads:** every run ends with an evidence record, landed or stopped. A reviewer can follow the request, the files, the check and the gate from the record alone. See [Evidence records](evidence-records.md).

## Details

- **Where things are kept.** The configuration is `.codelead/config.json`. Evidence records are `.codelead/evidence/ev-*.json`. Session logs are `.codeleadsessions/<session id>/`. The repository map is `docs/codelead/repository-map.md` and `.json`. See [Files and folders](../reference/files-and-folders.md).
- **The steps of `/implement`.** Preflight, patch, the patch check, up to 2 patch retries, apply, validation. Step 4 appears only when a retry runs.
- **What the model sees.** The request, the plan, the files that CodeLead selected, the [repository map](repository-map.md) and the [conventions](conventions.md). This is the call's context package. See [The knowledge base](knowledge-base.md).
- **The strategy.** How much the preflight does, from no questions to a deep analysis, is a [strategy](strategies.md). The default, `auto`, picks one from the words of the request.
- **Checkpoint commits.** `/implement` commits a landing only when `CODELEAD_CHECKPOINT_COMMITS=1` is set. [`/build-incremental`](../chat-commands/build-incremental.md) commits every verified increment.
- **Exit codes.** On the command line, `0` means done, `1` means a check or a gate failed, `2` means refused before anything ran. See [Exit codes](../reference/exit-codes.md).

## Related content

- [`/implement`](../chat-commands/implement.md), [`/patch`](../chat-commands/patch.md), [`/edit`](../chat-commands/edit.md), [`/validate`](../chat-commands/validate.md), [`/repair`](../chat-commands/repair.md)
- [`/create-app`](../chat-commands/create-app.md) and [`/build-incremental`](../chat-commands/build-incremental.md)
- [`codelead interactive`](../cli/codelead-interactive.md) and [`codelead implement`](../cli/codelead-implement.md)
- [Requests, plans and patches](requests-plans-and-patches.md)
- [The patch check](the-patch-check.md)
- [Validation and the gate](validation-and-the-gate.md)
- [Evidence records](evidence-records.md)
- [Glossary](glossary.md)
- [Quickstart: change existing code](../get-started/quickstart-existing-code.md)
