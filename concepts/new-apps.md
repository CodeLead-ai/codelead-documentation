---
title: "New apps: steps, increments and the run report"
description: The two ways CodeLead builds a new app - plan steps with /create-app and /build-step, or verified and committed increments with /build-incremental - and the run journal and run report that an incremental build writes.
kind: concept
area: concepts
date: 2026-11-20
---

# New apps: steps, increments and the run report

CodeLead builds a new app in one of two ways. **Steps**: [`/create-app`](../chat-commands/create-app.md) plans the app as a list of steps, and [`/build-step`](../chat-commands/build-step.md) builds them one at a time. **Increments**: [`/build-incremental`](../chat-commands/build-incremental.md) plans the app as a few slices that each work on their own, verifies each slice, and commits it. An incremental build also writes a **run journal** and a live **run report**. The report is one page that says how each increment went. It gives the gate's own account of each problem, and what you can do about it.

**In short**
- A step is a small part of the plan. It passes [the patch check](the-patch-check.md) and its validation before the next step starts. The chat's **Create an app** uses this way, with `/create-app --auto`.
- An increment is a slice of the app that works on its own. CodeLead builds it in up to 3 attempts, runs its gates, and makes a checkpoint commit when it is verified.
- `/build-incremental` writes `.codelead/run-journal.json`, one event at a time, and builds `.codelead/run-report.html` from it.
- A warning or a failure in the report carries the gate's account and a list of options. Heuristic findings warn: they never undo code that builds and passes its tests.

## Why it exists

A model can write a whole app in one answer, but nobody can check that answer. CodeLead cuts the work into parts that can each be checked: a step, or an increment. Each part goes through the same loop as a change to existing code: a patch, the patch check, then your project's build and tests. See [How CodeLead works](how-codelead-works.md).

The two ways answer two needs. Steps are small and simple, and you can look at the app between them. Increments are larger and verified on the running app, and each one is a commit. When a later increment goes wrong, every earlier one is safe in git. With the run report, a person sees a long build at a glance. The person knows what to do about each problem without reading the session log.

## How it works

### The two ways side by side

| | Steps | Increments |
|---|---|---|
| Commands | [`/create-app`](../chat-commands/create-app.md), [`/plan`](../chat-commands/plan.md), [`/build-step`](../chat-commands/build-step.md) | [`/build-incremental`](../chat-commands/build-incremental.md) |
| The plan | A project specification and an execution plan of steps. | A short list of increments: a `skeleton`, then each `capability`, and an optional `polish`. |
| You approve | The plan with `/plan --approve`, then each step's execution plan, patch and validation. `--auto` gives each question its recommended answer. | The list of increments once, then "Continue to the next increment?" after each one. |
| The check of a part | The step's validation commands. | Up to 3 attempts, each through the increment's gates. |
| A commit | None by itself. | A checkpoint commit for each verified increment. |
| The record | The plan and the build's state in the session folder. | The increment ledger, the run journal and the run report. |
| In VS Code | **Create an app** sends `/create-app --auto <your words>`. | Type `/build-incremental <your words>` in the chat. |

### Steps

`/create-app --plan` asks a model for two things: the project specification, which says what the app does, and the execution plan, which lists the steps. You read them with [`/plan --show`](../chat-commands/plan.md) and [`/spec`](../chat-commands/spec.md), and approve them with `/plan --approve`.

Then `/build-step` builds the next step:

1. A model writes the step's execution plan: the files that it creates or changes, and how the step is checked.
2. A model writes the step's patch. The patch check reads it.
3. CodeLead applies the patch after your yes.
4. CodeLead runs the step's validation. When it passes, the step is complete.

`/create-app --auto` does all of this for every step, without stopping, and gives each question its recommended answer. A no is included: in a folder that holds a project, the command stops before it plans. A step that is hard to undo is never answered: the build stops there, with the reason and the way out. A consent for a remote model is asked of you at the start, before the first model call. When every step is done, it says "Automated build completed: all plan steps are done." A step whose validation fails needs [`/build-step --repair`](../chat-commands/build-step.md). [`/greenfield`](../chat-commands/greenfield.md) shows where the plan and the build stand.

Here is a step that passed:

```output
✅ 👍 Step 2 of 6 completed. Validation passed.
Manual validation and remaining visual/product criteria may still be outstanding.

Next step:
<next step id> — <next step title>

Run /build-step again to continue.
```

The plan and the build's state live in the session folder: `.codeleadsessions/<session>/greenfield/`, with `state.json`, `spec.json`, `plan.json` and `build.json`. A later session in the same folder continues the build. See [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md).

### Increments

`/build-incremental <request>` asks a model for a short list of increments, and shows it:

<!-- described, not captured -->
```output
Increment plan (3 increments):
1. [skeleton] …
2. [capability] …
3. [capability] …
Build these increments one at a time? [Y/n]
```

After your yes, CodeLead builds each increment in turn:

1. A model writes the increment's patch. The patch check reads it, and CodeLead applies it.
2. CodeLead runs the increment's gates: the build, the tests, the probes of the running app, and the walk of its spec.
3. When the gates pass, the increment is verified. CodeLead commits the folder: `increment inc-2: <title>`. It runs `git init` first when the folder has no repository.
4. CodeLead asks "Continue to the next increment?". Enter continues.

An increment has at most 3 attempts. Each attempt that fails gives the next one the reason. An increment that still fails is recorded as failed, and the run goes on with the next increment. The run ends with a count:

```output
Incremental build complete: 3/3 increments built, verified, and checkpointed.
```

A run with a failed increment ends with the way on: `Build each failed increment with /implement <the increment's objective>; the increments above are committed`.

> [!NOTE]
> **New to this?** A *checkpoint commit* is a git commit that CodeLead makes for you after an increment passes. You can look at the app at any increment with `git log` and `git checkout`.

### The gates of an increment

Each attempt records the result of each gate that ran. A gate's result is `passed`, `failed`, `skipped`, `unavailable` or `flagged`.

| Gate | What it checks |
|---|---|
| **Patch check** | The patch fits the files. A `flagged` result means that a stale block was placed again in the file as it is now. |
| **Prohibitions** | The code avoids what the request forbids, for example a CSS framework in a request that says "plain CSS". |
| **Build** | The app compiles. |
| **Tests** | The app's tests pass. |
| **Acceptance probe** | CodeLead opens the running app in a browser and looks for what the increment's criteria describe. |
| **Regression probe** | The criteria of earlier increments can still be seen on the running app. |
| **Spec walk** | The increment's spec, drafted into a journey, is walked on the running app. |
| **Checkpoint** | The commit of the verified increment. |
| **Inner loop** | Only when the inner loop is on: the model's own build and test turns inside the increment. |

The build and the tests decide. The probes are heuristics: they can be wrong. When the acceptance probe rejects code that builds and passes its tests, the increment gets one more attempt. After a second rejection, CodeLead keeps the code with a warning. A finding of the regression probe is a warning at once. A failed spec walk is also a warning, never a revert. One finding is not a heuristic: a page of an earlier verified increment that renders nothing now. That is a hard regression: CodeLead retries with the reason, and refuses the increment on the third attempt.

This rule is the default [probe authority](glossary.md#p), `evidence`. With the probe authority `gate`, every probe finding refuses the attempt, and the increment fails when its attempts are spent. Set it with `greenfield.probeAuthority` in `.codelead/config.json`, or with `CODELEAD_PROBE_AUTHORITY`, which wins over the file. See [Configuration](../reference/configuration.md#greenfield).

An increment ends with one of these statuses: **Verified**, **Verified · warnings**, **Code committed · tests deferred** or **Failed**. A polish increment that changed nothing ends **Closed (no-op)**.

### The run journal

The run journal is `.codelead/run-journal.json`, in the app's folder, beside the [evidence records](evidence-records.md). CodeLead adds one event each time something happens to an increment, and writes the file again. Here is a short journal, built from the format's fields:

<!-- described, not captured -->
```json
{
  "schema": "codelead.run-journal/1",
  "version": 1,
  "events": [
    { "type": "run_started", "at": "2026-11-20T09:30:00.000Z", "request": "a task manager app with a list of tasks and a done checkbox",
      "plannedIncrements": [{ "id": "inc-1", "title": "Task list", "kind": "skeleton" }, { "id": "inc-2", "title": "Done checkbox", "kind": "capability" }] },
    { "type": "attempt_started", "at": "2026-11-20T09:35:00.000Z", "incrementId": "inc-1", "attempt": 1, "maxAttempts": 3 },
    { "type": "gate", "at": "2026-11-20T09:35:07.000Z", "incrementId": "inc-1", "attempt": 1, "gate": "build", "status": "passed", "summary": "build passed" },
    { "type": "gate", "at": "2026-11-20T09:35:08.000Z", "incrementId": "inc-1", "attempt": 1, "gate": "checkpoint", "status": "passed", "summary": "checkpoint committed" },
    { "type": "increment_settled", "at": "2026-11-20T09:35:08.000Z", "incrementId": "inc-1", "status": "verified", "attempts": 1 },
    { "type": "run_ended", "at": "2026-11-20T10:01:00.000Z", "exitReason": "all_increments_done", "wallClockMs": 1860000 }
  ]
}
```

The main events:

| Event | When CodeLead writes it |
|---|---|
| `run_started` | At the start: the request and the planned increments. |
| `attempt_started` | At each attempt of an increment, with the attempt's number and the maximum. |
| `gate` | At each gate's result, with a summary, and the gate's own account in `detail` when there is one: the compiler's output, the probe's observation. |
| `increment_settled` | When an increment ends: its status, its attempts and the files that it touched. |
| `walk_steps` | After a spec walk: each step, who walked it, and whether it passed. |
| `remote_provider` | When a role uses a remote provider: what leaves the machine, in words. |
| `run_ended` | At the end: why the run ended and how long it took. |

The journal records what happened. It is between the terminal's output and the [session log](sessions.md) in detail. The session log stays the full record.

### The run report

The run report is `.codelead/run-report.html`. CodeLead builds it from the journal and the increment ledger, at the start of the run, after every checkpoint, and at the end. The session prints where it is:

```output
Run report: <your project>/.codelead/run-report.html — refreshed at every checkpoint.
```

Open it in a browser. While the build runs, the page says "Live — refreshes every minute while the build runs." and reloads itself every 60 seconds. At the end it says "Finished", the exit reason and the time.

The page shows, from top to bottom:

- The request, the model and the start time. When a role uses a remote provider, a box says which and what left the machine.
- The licence's warnings, when there are any. See [Licence and plans](licence-and-plans.md).
- The totals: increments, verified, with warnings, failed, tests deferred, pending.
- One row for each increment: its status, its attempts, its duration, each gate's result, and the files that it touched.
- The spec walks, step by step, and the coverage of the specs when CodeLead wrote it.

Each warning and each failure of a row is a box with three parts. The title names the attempt, the gate, the gate's result at that attempt, and what went wrong. **What the gate saw** holds the gate's own account, folded. **What you can do** lists the options. For example, a probe that saw nothing change on the first attempt:

<!-- described, not captured -->
```output
Attempt 1 · Acceptance probe failed: The probe interacted and saw nothing change
What the gate saw
  …
What you can do
  - Confirm the probe was on the right page: an increment at a route like /listings/:slug needs a link the probe can follow from the home page, or a `route` in the plan.
  - If the interaction the criteria describe needs a control the probe cannot operate (a select, a date picker), the verdict is uninformative — rely on the tests.
  - Open the app and perform the interaction by hand; if it works, keep the code.
```

The journal records each rejection of the acceptance probe as `failed`, also the one that CodeLead keeps. The box is a warning when the increment was tried again or kept, and an error when the increment failed on it. A heuristic finding of the regression probe reads `flagged`, for example `Attempt 1 · Regression probe flagged: …`.

Where it helps to build the increment again, an option names how to do it in the chat: `/implement <the increment's objective>`.

> [!NOTE]
> **For team leads:** the run report is the page to keep open during a long build. Every verified increment is also a commit, so a reviewer can read each increment's diff with `git show`.

### In VS Code

- The chat runs both ways. **Create an app** in the [Projects view](../extension/projects-view.md) sends `/create-app --auto <your words>`. For increments, type `/build-incremental <your words>` in the [chat](../extension/chat.md).
- **Run journal** appears under the project in the Projects view when `.codelead/run-journal.json` exists. It opens a page with the increments, their attempts and their gates, and links the run report.
- **Open the run report**, in the Projects view's **…** menu, opens `.codelead/run-report.html` in your browser.

## What it is not

- **Not two products.** Both ways use the same models, the same patch check and the same validation. Only the size of a part and the record differ.
- **Not a report for the step flow.** `/create-app` and `/build-step` write no run journal and no run report. Their record is the session log and the build's state.
- **Not a judge of taste.** The gates check that the app builds, passes its tests and shows what the criteria say. How good the app looks is for a person to judge. The review pass can help. See [Settings](../extension/settings.md#greenfield-builds).
- **Not a revert machine.** A heuristic finding never undoes code that builds and passes its tests. It warns, with its reason in the report.

## Details

- **The files.** `.codelead/run-journal.json` (schema `codelead.run-journal/1`), `.codelead/run-report.html`, and the ledger `.codeleadsessions/increment-ledger.json` (schema `codelead.increment-ledger/1`). The specs of the increments go to `docs/codelead/specs/<id>.md`. See [Files and folders](../reference/files-and-folders.md).
- **Attempts.** At most 3 for each increment, for every cause of failure together.
- **The probe's retry.** It counts the acceptance probe's rejections, not the attempts. A first rejection on an increment's last attempt has no attempt left for the retry, so the increment fails.
- **A cut reply.** When the model's reply for a step stops at its cap, CodeLead applies nothing of it. This holds for the step's patch, its retry and its validation repair. The next request to the model says that the reply was cut, and asks for a shorter patch.
- **The polish and the review pass.** With the `repair` role set, `greenfield.polishOnRepairRole` puts the polish step or the polish increment on the repair role's model. `greenfield.reviewPassAfterBuild` adds one last step or increment, `Quality and UX review`, in which that model reviews the app. A review pass that does not land never makes the build fail.
- **A pause.** "No" to "Continue to the next increment?" pauses the run: "Incremental build paused after 2/4 increments. The app builds and every completed increment is committed." Run `/build-incremental` again to continue.
- **The rebuild.** The rebuild of a [modernization](../modernization/rebuild.md) writes the same run journal and run report in the new application's folder.
- **The repository map.** When a build finishes in an initialized folder, CodeLead writes the new app's [repository map](repository-map.md).

## Related content

- [`/create-app`](../chat-commands/create-app.md), [`/plan`](../chat-commands/plan.md), [`/spec`](../chat-commands/spec.md) and [`/build-step`](../chat-commands/build-step.md)
- [`/build-incremental`](../chat-commands/build-incremental.md) and [`/greenfield`](../chat-commands/greenfield.md)
- [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md)
- [Quickstart: build a new app](../get-started/quickstart-new-app.md)
- [Start a new app from a template](../tutorials/start-from-a-template.md)
- [Validation and the gate](validation-and-the-gate.md)
- [Models and roles](models-and-roles.md)
- [The Projects view](../extension/projects-view.md)
