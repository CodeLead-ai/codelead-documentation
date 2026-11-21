---
title: Stop and resume work
description: Stop a command in the chat or in a terminal, know what CodeLead keeps, and continue each kind of work later - a change, a new app, a pin, the modernization program and an unattended run.
kind: how-to
area: how-to
date: 2026-11-20
---

# Stop and resume work

This guide stops CodeLead in the middle of its work, and continues the work later. CodeLead writes its state at every step, so a stop loses little. Each kind of work has its own way to continue: this guide gives the one line for each.

**In short**
- In the chat, **Stop** asks CodeLead to stop after the current step. In a terminal, **Ctrl+C** ends the session.
- The session log, the evidence records and the state of each command stay on disk.
- A change continues with `/session --resume`, and a new app continues when you send its request again.
- The modernization program continues with `codelead modernize run <stage>` or `next`, and an unattended run continues when you start it again.

## Before you begin

- You know what a [session](../concepts/sessions.md) is: one conversation with CodeLead in one folder, with its log.
- You know which command runs: a change (`/implement`), a new app, a pin or a transform, the modernization program, or an unattended run.

## Stop in the chat

1. Select **Stop**. The button says **Stopping…**. CodeLead is asked to stop after the current step.
2. An open question gets no answer. A yes-or-no question is declined, and any other question takes its default.
3. When CodeLead is still busy 3 seconds later, the button becomes **End the session**. Select it to end the session now. A fresh session starts, and the history stays.

A command that cannot be cancelled goes on to its end. The chat then says: `Interrupt: the command in flight cannot be cancelled; it goes on to its end.` See [The chat](../extension/chat.md#stop-and-end-the-session).

## Stop in a terminal

- **A session of [`codelead interactive`](../cli/codelead-interactive.md)**: press **Ctrl+C**. The session ends, and its log closes with the reason `process_exit`. The state of an `/implement` stays as its last step saved it.
- **A pause with a note**: type [`/session --pause`](../chat-commands/session.md) `<note>`. CodeLead saves where the `/implement` stands, with your note, and ends the session:

  ```output
  Paused at ready_to_apply.
  Start CodeLead in this repository again and type /session --resume to continue.
  Exiting CodeLead.
  ```

  When you ran `/edit` or `/validate` by hand after the `/implement` stopped, CodeLead pauses at the stage that those steps left. The line says so, for example `Paused at validation_failed (the /implement stopped at ready_to_apply; the patch was applied and validation failed by hand since).`

- **An unattended run** of [`codelead implement`](../cli/codelead-implement.md) or [`codelead rebuild run`](../cli/codelead-rebuild-run.md): the first **Ctrl+C** stops the run after the card or the part in flight. A second **Ctrl+C** stops it now, and the exit code is `130`. From another terminal, create the file `.codelead/STOP` in the folder of the run: the run stops after the card or the part in flight.
- **The modernization program**: in a second terminal, run [`codelead modernize stop`](../cli/codelead-modernize-stop.md). The running stage finishes its step in flight, then ends failed, "stopped by the person". No further stage runs.

> [!CAUTION]
> A second **Ctrl+C** in an unattended run can leave the card in flight in **In progress**, with no account on it. Read the card, then move it back to Backlog when it must run again.

## Know what CodeLead keeps

| What | Where | Kept by |
|---|---|---|
| Every line typed and written, every model call, every result | `.codeleadsessions/<session>/` | Every session, as it happens |
| What was asked, what changed, the verdicts | `.codelead/evidence/` | Every run, landed or stopped |
| Where an `/implement` stands | `.codeleadsessions/<session>/implement/state.json` | Each step of `/implement` |
| A new app's plan and its build | `.codeleadsessions/<session>/greenfield/` | Each step of the build |
| Each stage of the modernization program | `.codelead/modernization.json` | Each change of a stage |

A change that was applied stays in your files. CodeLead never undoes an applied patch by itself.

## Continue a change

Start a session in the same folder. CodeLead offers the `/implement` that did not finish:

<!-- described, not captured -->
```output
A paused /implement session exists (session-2026-11-20T04-30-31-668Z, saved 2026-11-20T04:41:02.118Z).
  Request: add a clearCompleted function that removes the done tasks, with a test
  Stopped at: ready_to_apply
  Next: review with /patch --show, apply with /edit, then /validate
  Working tree: the touched files are unchanged.
Continue with /session --resume, or drop it with /session --discard.
```

1. Type `/session --resume`. CodeLead loads the `/implement` and says what comes next.
2. Read the line `Working tree`. When the touched files changed after the stop, CodeLead names them. It also says when git's `HEAD` moved.
3. Do the next step that it names, for example `/edit`, then `/validate`.

To drop the saved `/implement`, type `/session --discard`. Its log and its evidence stay. See [`/session`](../chat-commands/session.md).

## Continue a new app

Send the same request again: the same [`/create-app`](../chat-commands/create-app.md) line, or **Retry** in the chat. CodeLead continues the build in progress, and does not plan again:

<!-- described, not captured -->
```output
A build of this folder is in progress: 3 of 6 steps done, step-4 next.
Continue it with /create-app --auto, one step with /build-step, change it with /plan --revise <what>, or plan anew with /create-app --start-over <request>.
```

- `/create-app --auto <request>` continues from there without stopping.
- [`/build-step`](../chat-commands/build-step.md) builds one step.
- `/create-app --start-over <request>` plans anew. Use it only when you want a new plan.
- An incremental build continues with [`/build-incremental`](../chat-commands/build-incremental.md) and the same request. Every completed increment is already committed.

## Continue a pin or a transform

Run the same line again, for example [`/modernize --pin`](../chat-commands/modernize.md#--pin) `webapp`. CodeLead first looks at what the last run left:

- **An increment applied and not validated**: it asks `An increment is applied and not validated: validate it first? [Y/n]`. Press **Enter**. A pass lands the increment.
- **A patch that you declined**: it offers three choices. `1` applies it, `2` shows it, `3` composes it again. **Enter** gives `2`, which changes nothing.
- **To compose anew on purpose**: add `--again`, for example `/modernize --pin webapp --again`.

## Continue the modernization program

1. Run [`codelead modernize status`](../cli/codelead-modernize-status.md). Its last line, `Next:`, names the line to run.
2. Run that line:
   - A stage that you stopped ended `failed`, "stopped by the person". Run it again: [`codelead modernize run`](../cli/codelead-modernize-run.md) `<stage>`. What it finished before the stop is kept.
   - A stage that waits for you: give what it waits for, then run [`codelead modernize next`](../cli/codelead-modernize-next.md). `next` runs a waiting stage again, and never a failed one. A waiting Pin stage is the exception: run `codelead modernize run pins`, as its `Next:` line says.
   - The recordings stage waits until you say that the recordings are made: [`codelead modernize continue`](../cli/codelead-modernize-continue.md).
3. When a stage must stay behind, go past it with your reason: [`codelead modernize skip`](../cli/codelead-modernize-skip.md) `<stage> --reason <why>`. The stage ends `skipped`, by the person, and `next` goes on.

See [The modernization program](../modernization/the-program.md).

## Continue an unattended run

Start the same command again, for example `codelead implement --queue` or `codelead rebuild run`. The run that found the stop file took it away. A run refuses to start while a stop file is there: `A stop file is there (.codelead/STOP): a run stops on it. Take it away to start one.`

The queue takes the tagged cards again, Backlog first. A rebuild run starts from the plan's next part.

## Check that it worked

- `codelead sessions list` shows the earlier session, and `/session --resume` finds its `/implement`.
- `codelead modernize status` shows the stage `running`, then `done` or `waiting`, and no longer `failed`.
- The newest evidence record in `.codelead/evidence/` is the continued run's, with its outcome.

## Details

- **Exit codes.** `130` means stopped by Ctrl+C. See [Exit codes](../reference/exit-codes.md).
- **A crash or a closed terminal** during a modernization stage leaves it `running` with nobody holding the run lock. The next run marks it failed, "did not finish", and keeps what the stage finished.
- **A closed chat tab** ends its session: the extension sends `exit`, and waits up to 5 seconds.
- **One run per folder.** The run lock `.codelead/run.lock` refuses a second run. See [Unattended runs and the run lock](../concepts/unattended-runs.md).

## See also

- [Sessions](../concepts/sessions.md)
- [`/session`](../chat-commands/session.md) and [`codelead sessions`](../cli/codelead-sessions.md)
- [`/create-app`](../chat-commands/create-app.md)
- [`codelead modernize`](../cli/codelead-modernize.md) and [`codelead modernize stop`](../cli/codelead-modernize-stop.md)
- [Fix a stop](fix-a-stop.md)
- [Unattended runs and the run lock](../concepts/unattended-runs.md)
