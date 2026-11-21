---
title: /validate
description: Run the project's build and test commands in a CodeLead session, read the verdict, and know what to do when it fails.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /validate

**Applies to:** CodeLead 0.1 and later

## Name

`/validate` - Runs the project's validation commands and records the verdict.

## Synopsis

```console
/validate
/validate --scope repo
/validate --scope current
/validate --scope <PATH>
/validate --override
/validate --mark-passed
/validate --repair
```

## Description

`/validate` runs the validation commands of a scope (a part of the repository with its own build and tests). The commands are the gate: a change is done when they pass. See [Validation and the gate](../concepts/validation-and-the-gate.md).

You use `/validate` after [`/edit`](edit.md) applies a patch, after a repair round, or at any time to see whether the project builds and its tests pass. When it passes, the change is ready to commit, or to send for review with [`/pr`](pr.md). When it fails, run [`/repair`](repair.md), or fix the code by hand and run `/validate` again.

`/validate` never changes a file. It shows the commands first and asks before it runs them.

> [!NOTE]
> **New to this?** Validation means the checks that a project already has: its build, its tests, its type check. CodeLead runs them and reads the result for you. A failed validation is not a problem of CodeLead: it is the project telling you what is wrong.

### Where the commands come from

1. The commands declared in `.codelead/validation.json`, when the file exists. Declared commands replace everything that CodeLead would detect for that scope.
2. Otherwise, the commands that CodeLead detects. For Node.js, these are the `typecheck`, `test`, `build` and `lint` scripts of `package.json`. For .NET, they are `dotnet build` and `dotnet test`, or the MSBuild of Visual Studio for a .NET Framework project. For Python and Go, they are `python -m pytest` and `go test ./...`.

### Which scope

Without `--scope`, CodeLead finds the scope from the files that changed in this session. When more than one scope fits, it asks which one, and it recommends the widest. A [`/modernize --pin`](modernize.md) or an in-place transform validates what it changed and the declared gate, and asks no scope question.

When the session knows the task's chain of evidence records (an `/implement` or a pin before it), the result names the repair round: `Validation failed after repair round 1: dotnet test exited 1`. Each round's validation writes its own evidence record.

> [!TIP]
> **For experienced users:** declare the gate in `.codelead/validation.json` with the schema `codelead.validation/1`: a `commands` list, or `scopes` keyed by folder. Each entry has `command`, `args`, and optional `label`, `cwd` and `platforms`. See [Validation file](../reference/validation-file.md).

## Options

- **`--scope repo|current|<PATH>`**

  The scope to validate. `repo` is the repository root. `current` is the scope of the files that changed in this session. A `<PATH>` is a folder inside the repository: CodeLead uses the scope that holds it. A path that leaves the repository is refused. The default is the scope of the changed files.

- **`--override`**

  Records a manual pass: you checked the change yourself, outside CodeLead. CodeLead asks what you verified, then asks for your yes. Use it only when the automated validation cannot run or does not test the change.

- **`--mark-passed`**

  The same as `--override`.

- **`--repair`**

  The same as [`/repair`](repair.md): asks a model for a patch that fixes the last failed validation.

## What CodeLead asks

| Question | CodeLead recommends | Enter gives |
|---|---|---|
| `Run these commands? [Y/n]` | Yes. | Yes. |
| `Choose validation scope [1-3], or press Enter for 1:` | 1, the widest scope. | 1. |
| `What did you verify?` (with `--override`) | Nothing: you type what you checked. | Your words are required. |
| `Mark validation as manually passed? [y/N]` (with `--override`) | No: "CodeLead did not see it pass; say yes once you checked it yourself". | No. |

## What you see

The result says the verdict. In the terminal, the last line is the closing line. In the chat, the same words are the result message, with **Show details** for the full validation block.

| Outcome | Headline | Facts | The way on |
|---|---|---|---|
| `passed` | `Validation passed: 2 commands` | `Passed: npm test`, `Passed: npm run build`, `Scope: repo root` | None. Commit, or run `/pr`. |
| `failed` | `Validation failed: npm test exited 1` | `Failed command: npm test (exit 1)`, `First error: …`, `Scope: repo root` | `/repair`, or `Or fix it by hand, then /validate`. |
| `failed` | `Validation failed: MSBuild could not run: Visual Studio's MSBuild is needed and was not found` | The command and the reason. | The tool's own way out, then `/validate`. |
| `failed` | `Validation failed: dotnet test timed out` | The command. | `/validate (a timeout is inconclusive, not a code error)`. |
| `stopped` | `Validation not run: the answer was no` | `Scope: repo root` | `/validate` |
| `nothing` | `No validation commands found for docs` | None. | `Run the validation by hand, then /validate --override` |
| `passed` | `Validation marked passed by hand: <your words>` | None. | None. |
| `stopped` | `Manual validation override canceled.` | None. | `/validate --override` |

When a command fails because it does not fit the project, the way on is `Declare the scope's build and test commands in .codelead/validation.json, then /validate`. A failure of the pins' own database says `stopped on the environment`: no repair round runs for it.

## Examples

- Run the validation of the project, and press Enter to accept the recommended yes:

  ```console
  /validate
  ```

  ```output
  Run validation commands?

  Validation scope: repo root

  Commands:
    1. npm test
    2. npm run build

  Run these commands? [Y/n]
  Validation
  Validation scope: repo root
  Running 1/2: npm test
  Running 2/2: npm run build
  Validation

  Status: passed
  Scope: repo root

  Commands:
    pass npm test - 287.875ms
    pass npm run build - 164.78995900000018ms

  Next:
    Review git status and commit your changes.
  ✔ Validation passed: 2 commands
  ```

- A test fails after a patch. This is a stop, and the closing line gives the way on:

  ```console
  /validate
  ```

  ```output
  Validation

  Status: failed
  Scope: repo root

  Failed command:
    npm test

  Exit code: 1

  Output preview:
    > task-manager@1.0.0 test
    > node --test

    TAP version 13
    # Subtest: clearCompleted removes the done tasks
    not ok 3 - clearCompleted removes the done tasks
    …
    # tests 3
    # pass 2
    # fail 1

  Next:
    Fix the issue manually, or paste the output with /paste --analyze.
  ✖ Validation failed: npm test exited 1; next: /repair
  ```

  Run [`/repair`](repair.md) to ask for a fix, or fix the code by hand and run `/validate` again.

- The project's `.codelead/validation.json` cannot be read. The detected commands run, and the result says so in one line:

  ```output
  Validation

  Status: passed
  Scope: repo root

  Commands:
    pass npm run test - 190.5892920000001ms

  Warning: .codelead/validation.json was not read (it is not valid JSON: Expected property name or '}' in JSON at position 2 (line 1 column 3)), so the detected commands ran instead; correct the file, or remove it, to run the declared commands.
  …
  ✔ Validation passed: 1 command
  ```

  The way out is to correct the file. After this file, `/validate` runs only `npm test` and `npm run build`:

  ```json
  {
    "schema": "codelead.validation/1",
    "commands": [
      { "label": "npm test", "command": "npm", "args": ["test"] },
      { "label": "npm run build", "command": "npm", "args": ["run", "build"] }
    ]
  }
  ```

- Record a manual pass after you checked the change yourself:

  ```console
  /validate --override
  ```

  ```output
  Manual validation override

  Use this only if you manually verified the change outside CodeLead or determined that automated validation is unavailable, unrelated, or not meaningful.
  …
  What did you verify? I ran the tests in the terminal

  Mark validation as manually passed? [y/N] y
  Validation marked as manually passed.

  Notes:
  I ran the tests in the terminal
  ✔ Validation marked passed by hand: I ran the tests in the terminal
  ```

- A scope outside the repository is refused before anything runs:

  ```console
  /validate --scope ../other
  ```

  ```output
  Invalid validation scope: ../other

  Usage:
    /validate
    /validate --scope repo
    /validate --scope current
    /validate --scope <path>
    /validate --override
    /validate --mark-passed
    /validate --repair
  ```

## Files

- Reads `.codelead/validation.json` at the repository root or in the scope's folder, and the project files that detection reads (`package.json`, `*.csproj`, `*.sln`, `*.slnx`).
- Writes the verdict to the session log (`.codeleadsessions/<session>/session.jsonl`).
- In a repair chain, writes the round's evidence record to `.codelead/evidence/<id>.json`.

## See also

- [`/edit`](edit.md) and [`/repair`](repair.md)
- [`/implement`](implement.md)
- [Validation and the gate](../concepts/validation-and-the-gate.md)
- [Declare validation commands](../how-to/declare-validation-commands.md)
- [Validation file](../reference/validation-file.md)
- [Fix a stop](../how-to/fix-a-stop.md)
- [Messages and what to do](../reference/messages.md)
