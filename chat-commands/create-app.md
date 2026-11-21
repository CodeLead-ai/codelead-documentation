---
title: /create-app
description: Plan a new app from a request, and build it step by step or without stopping, optionally from a template.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /create-app

**Applies to:** CodeLead 0.1 and later

## Name

`/create-app` - Plans a new app from a request, and can build every step of the plan without stopping.

## Synopsis

```console
/create-app --plan <REQUEST>
/create-app --auto <REQUEST>
/create-app --auto-after-plan <REQUEST>
/create-app --from-clipboard --plan
/create-app --template <TEMPLATE> --auto <REQUEST>
/create-app --start-over --plan <REQUEST>
```

## Description

`/create-app` starts a new app in an empty folder. A model reads your request and writes two things. The project specification says what the app does: its features, data and screens. The execution plan lists the steps that build it. See [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md).

The plan runs in four steps that the session prints: `Analyzing request`, the clarifying questions, `Project specification generated` and `Execution plan generated`. CodeLead checks the plan's quality and repairs it once when it must.

What happens next depends on the option:

- `--plan` stops after the plan. You read it with [`/plan --show`](plan.md) and [`/spec`](spec.md), approve it with `/plan --approve`, then build one step at a time with [`/build-step`](build-step.md).
- `--auto` plans and builds every step without stopping. CodeLead gives each question its recommended answer, and a no is included. The gates still run at every step.
- `--auto-after-plan` asks you the plan's questions, shows the plan, and asks `Start the automated build from this plan?`. A yes builds every step without stopping.

Each step is a patch that passes the patch check and the step's validation before the next step starts. When every step is done, the session says `Automated build completed: all plan steps are done.`

A folder that already holds a build in progress is continued, not planned again. Use `--start-over` to plan anew.

> [!NOTE]
> **New to this?** Open an empty folder first. CodeLead asks before it plans in a folder that already holds a project, and it recommends no. For a request, say what the app is for and what a person does in it: `a task manager app with a list of tasks and a done checkbox`.

> [!TIP]
> **For experienced users:** [`/build-incremental`](build-incremental.md) builds in verified increments, with a commit after each one and a run report. The `greenfield` settings of `.codelead/config.json` decide the polish and review passes on the repair model.

## Arguments

- **`<REQUEST>`**

  The app to build, in plain words.

## Options

- **`--plan`**

  Plans the app and stops. Nothing is created in the folder except the session's own files.

- **`--auto`**

  Plans the app, approves the plan and builds every step, without stopping. CodeLead gives each question its recommended answer. See [What CodeLead asks](#what-codelead-asks).

- **`--auto-after-plan`**

  Plans the app with you, then asks `Start the automated build from this plan?`. A yes builds every step without stopping.

- **`--from-clipboard`**

  Takes the request from the clipboard text. Use it with `--plan`.

- **`--template <TEMPLATE>`**

  Starts from a template: the template comes into the folder first, then the plan builds on it. A template is an npm `create` package such as `vite`, a folder such as `./my-shell`, a git address, or a name from `greenfield.templates` in the configuration. It goes before the request. See [Start a new app from a template](../tutorials/start-from-a-template.md).

- **`--start-over`**

  Plans anew, even when a build of this folder is in progress. Without it, CodeLead continues the build in progress.

- **`--build`**

  Not available yet. CodeLead refuses it with `/create-app --build is not available yet.`

`/create-app` alone, or with a request and no mode, is refused: `/create-app requires --plan in this version.`

## What CodeLead asks

| Question | CodeLead recommends | Enter gives |
|---|---|---|
| `Continue planning for this folder? [y/N]` (the folder holds a project) | No: "the folder holds an existing project". | No. |
| `Continue planning here? [y/N]` (a workspace or a root folder) | No: "the folder looks like a workspace or root folder". | No. |
| The planner's clarifying questions, each with a default | The default. | The default. |
| `Start the automated build from this plan? [Y/n]` (`--auto-after-plan`) | Yes. | Yes. |
| ``Run `npm create <package> .` from the registry? It is not on CodeLead's trusted list.`` (`--template`) | No: "it is not on CodeLead's trusted list". | No. |
| The questions of each build step | See [`/build-step`](build-step.md). | |

With `--auto`, CodeLead answers these questions by itself:

- A question with a recommended answer gets that answer, also when it is no. For each no, CodeLead writes a line: `Automated build: answered no, as CodeLead recommends (<why>).`
- A question with no recommended answer gets yes when it is a yes-or-no question, and its default otherwise.
- The template question is the exception. CodeLead always asks you.

So in a folder that holds a project, or in a workspace or root folder, `--auto` takes the recommended no. The command stops with the outcome `stopped`, and nothing is planned.

Two questions of a build step get special care with `--auto`:

- `Retry scaffold command?`: CodeLead first runs the failed scaffold command once more by itself. It says `Automated build: the scaffold command failed; retrying it once.` After a second failure, it takes the recommended no, and the model writes the files.
- `Mark manual validation as passed?`: nobody confirms the manual check. CodeLead takes the recommended no, and the step's gates run and decide the step. It says `Automated build: nobody confirmed the manual validation; the step's gates ran instead.` The step's record says `manualValidation: "unconfirmed"`.

## What you see

| Outcome | Headline | The way on |
|---|---|---|
| `done` | `Project Specification created.`, with the plan summary | `Review with /plan --show`, `Approve with /plan --approve`, `Revise with /plan --revise <instruction>` |
| `done` | `Automated build completed: all plan steps are done. Run npm run dev in the target folder to open the app.` | Open the app. |
| `stopped` | `Planning stopped: the folder holds an existing project` (a no to the folder question, yours or the recommended no of `--auto`). A workspace or root folder gives `Planning stopped: the folder looks like a workspace or root folder`. | `Open or create a dedicated folder for the app, then send the request again`, `Or send the request again and answer y to plan in this folder`. With `--auto`, the second line is `Or plan in this folder on your own word: /create-app --plan <request>, and answer y`. |
| `stopped` | `You did not confirm the package, so nothing was run (npm create <package>)` (`--template`) | `Name the template again (/create-app --template <template> <request>) and answer y to run the package`, `Or give a template on the trusted list (vite, next-app), a folder or a git address`, `Or build without a template: /create-app <request>` |
| `failed` | `Failed to create greenfield plan` | `Send the request again (Retry): the planner is asked again`, `Or a larger model for the planner: Settings → Model roles` |
| `failed` | `Automated build stopped: step <id> is blocked.` | `Continue the build: send the request again (Retry)`, `/plan --revise <what to change>`, `/build-step` |

## Examples

- Plan a new app and stop, to read the plan first:

  ```console
  /create-app --plan a task manager app with a list of tasks and a done checkbox
  ```

  <!-- described, not captured -->
  ```output
  Create App Plan
  Request: a task manager app with a list of tasks and a done checkbox

  Step 1/4: Analyzing request...
  Step 2/4: No clarifications required.
  Step 3/4: Project specification generated.
  Step 4/4: Execution plan generated.
  Project Specification created.
  Execution Plan created.

  Plan summary:
  1. …

  Next:
  - Review with /plan --show
  - Approve with /plan --approve
  - Revise with /plan --revise <instruction>
  ```

- Plan in a folder that already holds a project. Enter gives the recommended no:

  ```console
  /create-app --plan a task manager app
  ```

  ```output
  ⚠️  WARNING: Existing project folder detected

  Current folder:
  <your project>

  This folder appears to contain an existing project.

  Reasons:
  - Found a child project marker.

  Why this matters:
  Greenfield build steps may create files that conflict with files already in this folder.
  …
  Continue planning for this folder? [y/N]
  ■ Planning stopped: the folder holds an existing project; next: Open or create a dedicated folder for the app, then send the request again
  ```

- Build without stopping in the same folder. `--auto` takes the recommended no and stops:

  ```console
  /create-app --auto a task manager app
  ```

  ```output
  ⚠️  WARNING: Existing project folder detected
  …
  Continue planning for this folder? [y/N]
  Automated build: answered no, as CodeLead recommends (the folder holds an existing project).
  ■ Planning stopped: the folder holds an existing project; next: Open or create a dedicated folder for the app, then send the request again
  ```

  The result is `stopped`. The last line is the same in the terminal and in the chat.

- The model server does not run. This is a stop, and the problem gives the way on:

  ```output
  Create App Plan
  Request: a task manager app with a list of tasks and a done checkbox

  Step 1/4: Analyzing request...
  Failed to create greenfield plan
  Could not connect to lmstudio at http://localhost:1234/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ```

  In the chat, the problem card says `Send the request again (Retry): the planner is asked again`. Start the model server first.

- Start from a template and build without stopping:

  ```console
  /create-app --template vite --auto a task manager app with a list of tasks and a done checkbox
  ```

- A template option without its value is refused:

  ```console
  /create-app --template --auto a task manager
  ```

  ```output
  Missing value for --template: name the template, as in /create-app --template vite --auto <request>.
  ```

## Files

- Writes the plan and the build state to the session folder: `.codeleadsessions/<session>/greenfield/` (`state.json`, `spec.json`, `plan.json`, `build.json` and others). A later session continues from them.
- The build steps write the app's files in the folder, each through the patch check.
- When the build finishes in an initialized folder, CodeLead writes the [repository map](../concepts/repository-map.md) of the new app.

## See also

- [`/plan`](plan.md), [`/spec`](spec.md) and [`/build-step`](build-step.md)
- [`/build-incremental`](build-incremental.md)
- [`/greenfield`](greenfield.md)
- [Quickstart: build a new app](../get-started/quickstart-new-app.md)
- [Tutorial: build a new app, step by step](../tutorials/build-a-new-app.md)
- [Start a new app from a template](../tutorials/start-from-a-template.md)
- [Template manifest](../reference/template-manifest.md)
