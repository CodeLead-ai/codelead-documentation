---
title: Build a new app, step by step
description: Build a task manager app one verified increment at a time, watch the gates and a repair, read the run report, and recover when an increment stops.
kind: tutorial
area: tutorials
date: 2026-11-20
---

# Build a new app, step by step

In this tutorial, you build a small task manager app from one request. CodeLead plans the app as a short list of increments: slices of the app that work on their own. It builds them one at a time. Each increment must pass its [gates](../concepts/validation-and-the-gate.md), the checks that decide that it is done, before CodeLead commits it to git.

**In short**
- You give CodeLead one request, and you read the plan before anything is built.
- You watch each increment pass its gates, and you follow a repair when a gate fails.
- You read the run report, the evidence records and the git history of the build.
- You know what to do when an increment stops.

In this tutorial, you:

> [!div class="checklist"]
> - Prepare an empty folder for CodeLead
> - Plan the app as increments
> - Build the first increment and read its gates
> - Follow a repair when an increment fails
> - Read the run report and the evidence
> - Open the app
> - Learn what to do when an increment stops

## Prerequisites

What you must know:

- You know how to open a terminal and run a command.
- You know what a git commit is.
- You do not need to know React or TypeScript. The planner chooses the stack for the app.

What you must have:

- CodeLead installed. See [Install CodeLead](../get-started/install.md). The command `codelead --version` prints a version.
- Node.js 22 and git.
- A model. That is a local model server, or a key for a remote provider. See [Set up local models](../how-to/set-up-local-models.md) or [Set up remote models](../how-to/set-up-remote-models.md).
- Playwright's Chromium, for the checks that run the app in a browser. [`codelead setup`](../cli/codelead-setup.md) installs it.
- About one hour. The time depends on the speed of your model.

> [!NOTE]
> **New to this?** An increment is a slice of the app that works on its own. The first increment is a skeleton: an app shell that builds and runs. Each later increment adds one capability, such as "add a task". See [How CodeLead works](../concepts/how-codelead-works.md).

## Prepare an empty folder

CodeLead builds a new app in an empty folder. It keeps its own settings in a folder named `.codelead`, and [`codelead init`](../cli/codelead-init.md) creates it.

1. Create a folder for the app, and open it:

   ```console
   mkdir task-manager
   cd task-manager
   ```

1. Prepare the folder for CodeLead:

   ```console
   codelead init
   ```

   You see output similar to the following:

   ```output
   ✔ Created <your project>/.codelead/config.json
   <your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
   Repository map: nothing to read yet (no source files); it is written when the first build finishes.
   ```

   The folder holds no code yet, so there is nothing to read. CodeLead writes the [repository map](../concepts/repository-map.md) when the build finishes.

1. Check the machine and the model:

   ```console
   codelead doctor
   ```

   Every line that starts with `FAIL` or `missing` names what to fix, after the arrow `→`. The line `Model server at …` must say `ok`. See [`codelead doctor`](../cli/codelead-doctor.md).

## Plan the app as increments

The plan says what CodeLead will build. You read it before any code exists. When a plan is wrong, you stop here, because changing a plan costs less than changing code.

1. Start a session:

   ```console
   codelead interactive
   ```

   You see output similar to the following:

   ```output
   ╭──────────────────────────────────────────────────────────────────────────────╮
   │                                                                              │
   │   CodeLead Interactive                                                       │
   │                                                                              │
   │   Repo: <your project>                                                       │
   │   Strategy: auto                                                             │
   │   Project type: greenfield                                                   │
   │   Ask about the code in plain words (the chat never changes code), or use    │
   │   /help for commands.                                                        │
   │   Use /exit or /quit to leave.                                               │
   │                                                                              │
   ╰──────────────────────────────────────────────────────────────────────────────╯
   codelead>
   ```

   A [session](../concepts/sessions.md) is one conversation with CodeLead in this folder. See [`codelead interactive`](../cli/codelead-interactive.md).

1. Type the request with [`/build-incremental`](../chat-commands/build-incremental.md), and press **Enter**:

   ```console
   /build-incremental a task manager app with a list of tasks, a done checkbox and a count of the open tasks
   ```

   A model splits the request into increments. You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Build incremental
   Request: a task manager app with a list of tasks, a done checkbox and a count of the open tasks
   Increment plan (4 increments):
   1. [skeleton] Runnable app shell
      Render the page with a title and an empty task list, with no behavior behind them.
      Not in this increment: Add a task, Mark a task done, Open task count
   2. [capability] Add a task
      Add a task to the list from a text field and an Add button.
   3. [capability] Mark a task done
      Mark a task as done with its checkbox.
   4. [capability] Open task count
      Show how many tasks are not done.
   Build these increments one at a time? [Y/n]
   ```

   Your plan differs, because the model plans its own increments. The shape is the same.

1. Read each increment. Check its kind:
   - `skeleton`: the app shell, the first increment.
   - `capability`: one feature of the request.
   - `polish`: the look of the whole app, sometimes the last increment.

1. Read each line `Not in this increment`. It keeps the model from building everything at once.

1. Press **Enter** to accept the plan.

   The capital `Y` in `[Y/n]` is the recommended answer, and Enter gives it. See [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md).

> [!TIP]
> To build without git commits, and to stop after each step, use [`/create-app --plan`](../chat-commands/create-app.md), then [`/plan --approve`](../chat-commands/plan.md), then [`/build-step`](../chat-commands/build-step.md). The [quickstart](../get-started/quickstart-new-app.md) uses `/create-app --auto`, which builds every step without stopping.

## Build the first increment and read its gates

Each increment is one change. Inside it, CodeLead runs the same path as [`/implement`](../chat-commands/implement.md): a patch, [the patch check](../concepts/the-patch-check.md), the apply, and [the gate](../concepts/validation-and-the-gate.md). CodeLead gives every question inside an increment its recommended answer.

1. Read the first line after the plan. It names the run report:

   <!-- described, not captured -->
   ```output
   Run report: <your project>/.codelead/run-report.html — refreshed at every checkpoint.
   ```

1. Wait while CodeLead builds the skeleton. The first lines name the increment:

   <!-- described, not captured -->
   ```output
   Increment 1/4: Runnable app shell
   Render the page with a title and an empty task list, with no behavior behind them.
   Not in this increment: Add a task, Mark a task done, Open task count
   …
   Build after inc-1: passed (exit 0)
   Acceptance probe after inc-1: acceptance probe skipped: skeleton increment: deliberately inert by its non-goals; a skeleton is allowed to do nothing
   Increment inc-1 verified and checkpointed (committed).
   Continue to the next increment? [Y/n]
   ```

1. Read the gates that the increment passed. Each gate is a check that the increment must pass:

   | Gate | What it checks |
   |---|---|
   | The gate of the change | The project's own build and tests run after the patch is applied. |
   | Build | The whole app builds after the increment: `Build after inc-1: passed (exit 0)`. |
   | Acceptance probe | CodeLead starts the app in a browser and checks the increment's "done when" criteria. The skeleton is skipped, because it does nothing yet. |
   | Cumulative regression | When earlier increments are verified, CodeLead checks that they still work. |
   | Spec walk | When the plan wrote a spec for the increment, CodeLead follows it on the running app. |
   | Checkpoint | CodeLead commits the increment to git: `increment inc-1: Runnable app shell`. |

1. Press **Enter** to continue to the next increment.

> [!NOTE]
> **For team leads:** each increment is an `/implement` inside, so each attempt writes an [evidence record](../concepts/evidence-records.md). The checkpoint commits make the git history of the app match the plan.

## Follow a repair when an increment fails

An increment does not always pass on its first attempt. CodeLead then tries again by itself. The next attempt carries the gate's own words, so the model fixes what failed. Each increment has at most 3 attempts. See [Repair](../concepts/repair.md).

1. Watch the second increment. Suppose its first attempt leaves a compiler error. You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Increment 2/4: Add a task
   Add a task to the list from a text field and an Add button.
   …
   Build after inc-2: failed (exit 2)
   Increment inc-2 did not build. Retrying with the failure attached (attempt 2/3).

   Increment 2/4: Add a task
   …
   Build after inc-2: passed (exit 0)
   Acceptance probe after inc-2: acceptance probe passed (state_changed, visual_delta)
   Increment inc-2 verified and checkpointed (committed).
   Continue to the next increment? [Y/n]
   ```

   The second attempt's request holds the compiler's errors and the instruction `Fix ONLY the compiler errors below`. The attempt fixes the error, and every gate passes.

1. Press **Enter** to continue.

1. Let the other increments run in the same way. A failed acceptance probe works the same. The retry names the criteria that the probe could not observe:

   <!-- described, not captured -->
   ```output
   Increment inc-3 did not pass the acceptance probe. Retrying with the failed criteria named (attempt 2/3).
   ```

   The probe is a heuristic, so the build and the tests decide. When the probe rejects the second attempt too, CodeLead keeps the increment, because it builds and passes its tests. It commits the increment, with a warning:

   <!-- described, not captured -->
   ```output
   WARNING Increment inc-3 (Mark a task done): acceptance probe failed (visual_delta) — kept on its build and tests (probe authority: evidence; the probe's one retry is spent). The probe's account:
       The previous attempt built and changed code, but the app was then run and probed, and the capability could not be observed working.
       …
   …
   Increment inc-3 verified and checkpointed (committed).
   ```

   The run report shows such an increment as `Verified · warnings`. Open the app later, and try the feature by hand.

1. When the last increment is verified, read the last lines:

   <!-- described, not captured -->
   ```output
   Run report: <your project>/.codelead/run-report.html
   Incremental build complete: 4/4 increments built, verified, and checkpointed.
   Run the app in the target folder to exercise it.
   Repository map written: 1 module, 14 files; docs/codelead/repository-map.md
   ```

   CodeLead writes the repository map of the new app, because the folder was initialized.

## Read the run report and the evidence

The run report is the summary of the build, for a person. The evidence records and the session log are the full record. Read them in this order.

1. Open `.codelead/run-report.html` in a web browser. While the build runs, the page says `Live — refreshes every minute while the build runs.` When the build ends, it says `Finished`, the reason and the time.

1. Read the totals at the top: the increments, `verified`, `with warnings`, `failed`, `tests deferred` and `pending`.

1. Read the table `Increments`. Each row has the increment, its status, its attempts, its time and its gates.

   | Status | Meaning |
   |---|---|
   | Verified | Every gate passed. |
   | Verified · warnings | The increment is kept, and a gate flagged something for a person to look at. |
   | Failed | The increment is not built. |
   | In progress | The increment is building now. |
   | Pending | The increment has not started. |

1. Read a warning or a failure in a row, when there is one. Each one carries the gate's own account and the ways to fix it.

1. Close the session:

   ```console
   /exit
   ```

1. Read the history of the build:

   ```console
   git log --oneline
   ```

   <!-- described, not captured -->
   ```output
   7c41d2e increment inc-4: Open task count
   5b09a13 increment inc-3: Mark a task done
   2e8f6c0 increment inc-2: Add a task
   91d3b47 increment inc-1: Runnable app shell
   ```

1. List the evidence records:

   ```console
   ls .codelead/evidence
   ```

   Each file is one attempt of one increment, landed or stopped. An increment that needed two attempts has two records. See [Evidence records](../concepts/evidence-records.md).

> [!TIP]
> **For experienced users:** the run report is made from `.codelead/run-journal.json`, the timeline of every gate and attempt. The session log under `.codeleadsessions/` holds every prompt and every model answer. See [Sessions](../concepts/sessions.md).

## Open the app

The completion line says that the app runs in the folder. Run its development script.

1. Start the app:

   ```console
   npm run dev
   ```

1. Open the address that the script prints, for example `http://localhost:5173`.

1. Add a task, mark it done, and check that the count of open tasks changes.

1. Stop the app with **Ctrl+C**.

## Learn what to do when an increment stops

Sometimes an increment still fails after its 3 attempts. The result says what happened and what comes next. This table shows the three stops of an increment build, and the way on for each one.

| What you see | What it means | What to do |
|---|---|---|
| `Increment inc-3 (Mark a task done) FAILED (not built): did not leave the app buildable.`, then `Continuing to the next increment.` | The increment is not built. Every earlier increment is committed. The run goes on. | At the end, build the missing feature with [`/implement`](../chat-commands/implement.md). Use the increment's objective as the request. |
| `Incremental build stopped: increment inc-1 (Runnable app shell) did not leave the app buildable.` | The skeleton failed. There is no app for the next increment to build on. | `Next: fix the failure with /implement, then rerun /build-incremental.` |
| `Incremental build paused after 2/4 increments.` | You answered no to `Continue to the next increment?`. The app builds, and every completed increment is committed. | `Next: rerun /build-incremental to continue, or use /implement for the rest.` |

A build with a failed increment ends with this summary:

<!-- described, not captured -->
```output
Incremental build finished with failures: 3/4 increments built, verified, and checkpointed.
- FAILED (not built): inc-3 (Mark a task done)
Run the app in the target folder to exercise what was built; the failed increments are still open.

Next:
- Build each failed increment with /implement <the increment's objective>; the increments above are committed
```

> [!TIP]
> When the model server does not answer, the plan fails before anything is built: `Failed to create the increment plan`. Run `codelead doctor`, start the server, and type the request again. See [Fix a stop](../how-to/fix-a-stop.md) and [Messages and what to do](../reference/messages.md).

## Clean up

The app is a normal folder with a git repository. To remove everything that this tutorial made, delete the folder:

```console
cd ..
rm -rf task-manager
```

On Windows, in PowerShell, use `Remove-Item -Recurse -Force task-manager`.

## Next steps

> [!div class="nextstepaction"]
> [Change existing code with evidence](change-existing-code.md)

- [Start a new app from a template](start-from-a-template.md)
- [How CodeLead works](../concepts/how-codelead-works.md)
- [`/build-incremental`](../chat-commands/build-incremental.md) reference
- [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md)
