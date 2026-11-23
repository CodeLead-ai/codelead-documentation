---
title: Fix a stop
description: For each kind of stop - a model that does not answer, a cut reply, a refused patch check, a refused gate, a repair loop that stopped, a pin or a transform that did not land - what you see, what it means and what to do.
kind: how-to
area: how-to
date: 2026-11-20
---

# Fix a stop

A **stop** is a run that ends before its job is done, for example before a change lands. Every stop says why, and names the way on. This guide lists the stops by kind. For each one, it shows what you see, what it means, and what to do.

**In short**
- Read the last line of the command first. It is the [result](../concepts/results-and-stops.md): the outcome, the headline, and the way on after `next:`.
- A stop that is about a tool or a server needs a fix of the machine. A repair cannot help it.
- A stop that is about the code needs a repair round, or a clearer request.
- A stop that comes from your own "no" keeps the work. One command continues it.

## Before you begin

- You know where the result is. In a terminal, it is the closing line, with a mark: ✔ ✖ ■ ○. In the [chat](../extension/chat.md), it is a message with buttons, or a red problem card with **Retry**.
- [`codelead doctor`](../cli/codelead-doctor.md) works in the folder. Many stops start with a machine problem, and the doctor finds most of them.

> [!NOTE]
> **New to this?** A stop is not a crash. Nothing is applied that you did not approve, and every stop writes an [evidence record](../concepts/evidence-records.md) with its reason. For the full list of messages, see [Messages and what to do](../reference/messages.md).

## Find the stop

1. Read the closing line. For example:

   ```output
   ✖ Validation failed: type check exited 2; next: /repair
   ```

2. Find the kind of stop below by its words.
3. When the way on is not clear, read the block above the closing line. In the chat, select **Show details**.
4. To see what the model was asked and what it answered, read the session log. See [Read a session in the Session Log Viewer](read-a-session.md).

## Reach the model server

**What you see**: one of these lines.

```output
Could not connect to lmstudio at http://127.0.0.1:9/v1.
If this is LM Studio, make sure it is open and the Local Server is running.
```

- `The change did not land: unable to complete preflight`, after `/implement`.
- `Failed to create the increment plan`, or `Failed to create greenfield plan`, for a new app.
- `The model server of the preflight row did not answer: lmstudio at http://localhost:1234/v1`, for a pin or a transform.

**What it means**: the model server did not answer at the address of the configuration. Nothing was sent, and nothing changed.

**What to do**:

1. Run `codelead doctor`. The line `Model server at …` says `FAIL`, with the fix after the arrow `→`.
2. Start the server, or correct the address. See [Set up local models](set-up-local-models.md).
3. Run the same command again. In the chat, select **Retry**.

## Allow a remote model

**What you see**: `Remote model blocked by privacy settings.`, the provider, and the setting to change. In the extension, the chat's session does not start, and says why.

**What it means**: a role points to a remote provider, and remote models are not allowed. Nothing was sent.

**What to do**: when the code's owner agrees, set `privacy.allowRemoteModels` to `true`. Or move the role back to a local provider. See [Set up remote models](set-up-remote-models.md#allow-remote-models).

A run that nobody watches also needs `CODELEAD_ALLOW_REMOTE_MODELS=1`. Without it, the run is refused before its first call.

## Fit the prompt to the model

**What you see**:

<!-- described, not captured -->
```output
Model call failed: context length exceeded.

The model rejected the request because the prompt was larger than the loaded context window.

No files were changed.
```

**What it means**: the prompt was larger than the context length that the model was loaded with. A prompt also stops before the call when it is above CodeLead's own budget: `The model call was not sent.`

**What to do**: choose one.

- Load the model with a larger context length, and set `contextTokens` in its role. See [Set up local models](set-up-local-models.md#choose-the-model).
- Ask for a smaller change: one file, one function.
- Use a model with a larger window for that role.

## Get a reply that is not cut

**What you see**: `The model's reply was cut at its cap: …; the patch stops mid-file.` The words carry the counts, such as the tokens of the reply and how many of them were thinking.

**What it means**: the model's answer stopped at its token limit. CodeLead applies nothing of a cut reply, because part of a patch is not a patch. When most of the reply was thinking, CodeLead tries once more by itself, and asks for less thinking.

The same holds for a step of a [new app](../concepts/new-apps.md): its patch, its retry and its validation repair. The patch check refuses the cut reply with these words: `…; the patch stops mid-file, so nothing of it is applied. Send the whole patch again, shorter: fewer files or tests in one reply.` The step's next request to the model carries the same words.

**What to do**: choose one.

- Ask for less in one request: fewer files, fewer tests.
- Raise the cap of the strategy: `strategySettings.<strategy>.maxTokens` in `.codelead/config.json`. The result names the key. When the cap is at the provider's ceiling, the result says that only a smaller request fits.
- For a pin, pin fewer units in one increment: [`/modernize --pin`](../chat-commands/modernize.md#--pin) `webapp --units <unit>`.

## Pass the patch check

**What you see**: `The change did not land: the patch check refused the patch`, with `First problem: …` in the facts.

**What it means**: [the patch check](../concepts/the-patch-check.md) is CodeLead's own reading of the patch against your files, with no model. It found a problem that blocks the patch, also after the retries. Nothing changed.

**What to do**:

1. Read the first problem. It names the file and the line.
2. Run [`/patch --retry`](../chat-commands/patch.md). The model gets the check's own words.
3. Or say more in the request: the file, the function, and the exact words of a message.

A close cousin is `The change did not land: the patch did not apply: …`. The patch passed its check, but your files changed before the apply. Run `/patch --check`, then `/patch --retry`.

## Pass the gate

[The gate](../concepts/validation-and-the-gate.md) is your project's own build and tests. Its stop has three forms.

| What you see | What it means | What to do |
|---|---|---|
| `The change was applied, but validation failed: npm test exited 1` | The change is in your files. The code fails the gate. | Run [`/repair`](../chat-commands/repair.md). Or fix it by hand, then run [`/validate`](../chat-commands/validate.md). |
| `Validation failed: MSBuild could not run: Visual Studio's MSBuild is needed and was not found` | A tool that the gate needs is missing. A repair cannot help. | Install the tool, or run on the right system. Or declare the commands. See [Build .NET Framework projects on Windows](build-on-windows.md). |
| `Validation failed: dotnet test timed out` | The command ran out of time. The verdict is not known. | Run `/validate` again. |

Two more forms are rare:

- `… stopped on the environment: …`: something outside the code failed, such as a database that does not answer. CodeLead runs no repair round for it. Do what the words say, then run `/validate`.
- `No validation commands found`: CodeLead found no build or test. [Declare the commands](declare-validation-commands.md), or run your checks by hand and type `/validate --override`.

## Get out of a repair loop

**What you see**:

<!-- described, not captured -->
```output
✖ The repair loop stopped: this repair returns <file> to its state after repair 1, which failed with …
```

The facts list what each round tried.

**What it means**: the repairs go in circles. A repair would bring a file back to a state that an earlier round tried, or the gate is back at an earlier failure. CodeLead stops the loop, and asks no model for a further round. See [Repair](../concepts/repair.md).

**What to do**: choose one.

- Read the failure and the files that it names. Fix it by hand, then run `/validate`. A new failure starts the rounds afresh.
- Say what to change and where: `/patch <the change, in which file>`.
- Run the request again, with a narrower scope.
- Give the repairs a stronger model. See [Use a stronger model for repairs](use-a-stronger-model-for-repairs.md).

## Continue after your own no

**What you see**: `The change was not applied: the answer was no; nothing is changed`, with the outcome `stopped` and the mark ■.

**What it means**: you answered no to `Apply this patch?`. That is your choice, not a failure. The patch is kept in the session.

**What to do**: apply it as it is with [`/edit`](../chat-commands/edit.md), look at it with `/patch --show`, or run the request again. A no to `Run validation now?` works the same: the change is applied, and `/validate` judges it later.

## Read the stop of a pin or a transform

A pin (a test that records what the code does today) and an in-place transform are **composed requests**: CodeLead writes the request itself. When one does not land, its result names the class of the stop. A choice of yours is `stopped`, never `failed`.

| Class | Outcome | What you see, for example | What to do |
|---|---|---|---|
| `model_unreachable` | `failed` | `The model server of the preflight row did not answer: lmstudio at http://localhost:1234/v1` | `codelead doctor`, then the same command again. |
| `declined_patch` | `stopped` | `You declined the patch; nothing changed` | `/edit` applies the kept patch. `/patch --show` shows it. |
| `declined_validation` | `stopped` | `The pins were applied and not validated: the gate did not run` | `/validate`. A pass lands the increment. |
| `applied_not_validated` | `stopped` | `Transform T1 was applied and not validated: the gate did not run` | `/validate`. |
| `reply_cut` | `failed` | `… did not land`, with the counts of the cut reply | Pin fewer units: `--units <unit>`. |
| `check_refused` | `failed` | `… did not land: the patch check refused …` | `/patch --retry`, or run it again. |
| `gate_refused` | `failed` | `The pins of webapp did not land: 1 of 8 tests failed: <test>: <message>`. A build failure reads `…: dotnet test exited 1`. | `/repair` in this session. |
| `repair_stopped` | `failed` | The gate still refuses after the repair rounds. | Read what each round tried. Fix it by hand, then `/validate`. |
| `precondition` | `failed` | `The pins of webapp need Visual Studio's MSBuild, and it is not on this machine` | Do what the words say, before any model call. |
| `objection` | `failed` | The preflight's own words: the request cannot be done. | Change the request. |
| `other` | `failed` | Any other cause. | Read the words. Only this class offers a stronger model. |

When you run the same pin again over an applied increment that the gate did not judge, CodeLead asks `An increment is applied and not validated: validate it first? [Y/n]`. Over an increment whose gate refused it, it asks `An increment is applied and its gate refused it: run its gate again first? [Y/n]`. After a declined patch, it offers to apply it, show it, or compose it again. `--again` composes anew on purpose. See [Stop and resume work](stop-and-resume.md).

A few pins can fail only because the test process cannot run them. Then CodeLead offers `Set the 1 failing pin aside and land the 7 that pass? [Y/n]`. See [Pins](../modernization/pins.md#a-pin-that-cannot-run-in-the-test-process).

In the program, a Pin stage can fail on its gate. Its ways out work from a terminal: `codelead modernize run pins`, the chat opened with `codelead interactive`, or `codelead modernize answer pins=enough`.

## Name the cause for the preflight

**What you see**: `The run ended after the preflight: the cause is not found, so no patch was made`.

**What it means**: you asked to fix a bug, and the preflight (the analysis before a change) did not find its cause in the code. CodeLead does not guess a fix.

**What to do**: name the cause with its file and line in the request, and run [`/implement`](../chat-commands/implement.md) again. Or add `--force` to make the patch anyway.

## Unblock a new app's step

**What you see**: `Automated build stopped: step <id> is blocked.`, or `Incremental build stopped: increment inc-1 (Runnable app shell) did not leave the app buildable.`

**What it means**: a step of a [new app](../concepts/new-apps.md) failed after its attempts. The steps before it are kept.

**What to do**: send the request again (in the chat, **Retry**): CodeLead continues the build in progress. Or change the plan with [`/plan --revise`](../chat-commands/plan.md), or build one step with [`/build-step`](../chat-commands/build-step.md). A step blocked by the patch check says so: a plan revision cannot fix a check, so read the check's words.

## Unblock a modernization stage

**What you see**: [`codelead modernize status`](../cli/codelead-modernize-status.md) shows a stage `failed` or `waiting`, with its problem: the title, the reason, the way out and the line that runs it again.

**What it means**: `waiting` is your turn: the stage needs an answer, a recording or a reason. `failed` is a cause that is not a question for you.

**What to do**:

1. Read the problem's reason and its way out. `codelead modernize status` ends with `Next:` and the line to run.
2. Do what the way out says, then run that line, for example `codelead modernize run pins`.
3. When a stage cannot run on this machine, and the program must go on without it, skip it with your reason:

   ```console
   codelead modernize skip in-place --reason "no MSBuild on this machine; the data layer is moved by hand later"
   ```

   <!-- described, not captured -->
   ```output
   Transform in place: skipped by the person (no MSBuild on this machine; the data layer is moved by hand later); it was failed: The transform T1 did not land.
   Then: codelead modernize next (Extract the model).
   ```

   The state file and the skip's evidence record keep the reason. A skip without `--reason` is refused, and the recordings are never skipped. See [`codelead modernize skip`](../cli/codelead-modernize-skip.md).

4. When the stage waits on security findings, `9 security findings are decisions for a person`, decide each one. See [Waive a security finding](waive-a-security-finding.md).

## Free the folder's run lock

**What you see**: `This clone is in use: …. One run per clone: wait for it to end, or stop it …`

**What it means**: another unattended run holds the run lock of this folder, `.codelead/run.lock`.

**What to do**: wait for that run to end, or stop it. See [Unattended runs and the run lock](../concepts/unattended-runs.md) and [Stop and resume work](stop-and-resume.md).

## Check that it worked

- The same command, run again, ends with ✔ and the outcome `passed` or `done`.
- `codelead doctor` ends with `Ready: every required check passed.`
- The newest evidence record in `.codelead/evidence/` has the outcome `landed`, and no `stop_reason`.

## See also

- [Results and stops](../concepts/results-and-stops.md)
- [Messages and what to do](../reference/messages.md)
- [Exit codes](../reference/exit-codes.md)
- [Stop and resume work](stop-and-resume.md)
- [`codelead doctor`](../cli/codelead-doctor.md)
- [Repair](../concepts/repair.md) and [Validation and the gate](../concepts/validation-and-the-gate.md)
- [Troubleshoot the extension](../extension/troubleshooting.md)
