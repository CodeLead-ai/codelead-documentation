---
title: Validation and the gate
description: How CodeLead decides that a change is done, by running your project's own build and tests, and where those commands come from.
kind: concept
area: concepts
date: 2026-11-20
---

# Validation and the gate

**Validation** is CodeLead running your project's own checks: the build, the tests, the type check, the linter. The **gate** is the set of these commands that decides whether a change is done. A change that passes the gate is a **landing**. A change that fails it is not done, however good it looks.

**In short**
- The gate is made of your project's commands, not of CodeLead's opinion and not of a model's opinion.
- CodeLead reads the commands from `.codelead/validation.json` when the file exists. Otherwise it detects them from the project's files.
- A **scope** is a part of the repository with its own build and tests. CodeLead validates the scope of the files that changed.
- When the gate fails on the code, the way on is a [repair round](repair.md). When it fails on a tool, the way on is to fix the tool or declare the commands.

## Why it exists

A model can say "done" when nothing works. The patch check can say that a patch fits the files. It cannot say that the code compiles, or that it does what you asked. Only running the code can say that.

Your project already has checks that your team trusts. CodeLead runs them after every apply and reads the result for you. The rule is simple: no pass, no landing. The [evidence record](evidence-records.md) keeps each command with its exit code, so a reviewer sees the verdict and not a claim.

> [!NOTE]
> **New to this?** A failed validation is not a problem of CodeLead. It is your project telling you what is wrong. Read the first error line: it usually names the file and the line.

## How it works

### Where the commands come from

1. **Declared.** When `.codelead/validation.json` exists, its commands are the gate for that scope. Declared commands replace what CodeLead would detect. A file that CodeLead cannot read gives one `Warning:` line in the result, which names the file and the reason. The detected commands then run.
2. **Detected.** Otherwise CodeLead reads the project's files:
   - the `typecheck`, `test`, `build` and `lint` scripts of `package.json`;
   - `dotnet build` and `dotnet test` for a .NET SDK project. A `test/` folder alone, as in many Node.js projects, never adds `dotnet test`;
   - Visual Studio's MSBuild, then `dotnet test`, for a .NET Framework project that the .NET SDK cannot build;
   - `python -m pytest` and `go test ./...`.

A small declared gate for a Node.js project looks like this:

```json
{
  "schema": "codelead.validation/1",
  "commands": [
    { "label": "npm test", "command": "npm", "args": ["test"] },
    { "label": "npm run build", "command": "npm", "args": ["run", "build"] }
  ]
}
```

See [Declare validation commands](../how-to/declare-validation-commands.md) and the [Validation file](../reference/validation-file.md) reference.

### When it runs

- [`/implement`](../chat-commands/implement.md) asks `Run validation now? [Y/n]` after the apply. Enter gives yes.
- [`/validate`](../chat-commands/validate.md) shows the commands, asks `Run these commands? [Y/n]`, then runs them. You can run it at any time.
- Every build step of a new app, and every increment, runs its own validation before the next one starts.
- Every [repair round](repair.md) ends with validation.

### A real stop at the gate

Here is `/validate` on the Basic Store sample, on a Mac. The sample is an ASP.NET MVC 5 application on .NET Framework 4.5. Its project needs Visual Studio's MSBuild, which runs on Windows only. CodeLead finds this from the project's own files, before it runs anything:

```output
Run validation commands?

Validation scope: repo root

Commands:
  1. MSBuild sampleapp.slnx (Visual Studio's MSBuild: not found)

Run these commands? [Y/n]
…
Status: failed
Scope: repo root

Failed command:
  MSBuild sampleapp.slnx (Visual Studio's MSBuild: not found)

Exit code: (not run)

Output preview:
  Visual Studio's MSBuild is needed and was not found.
  …
Next:
  - Run it on Windows with Visual Studio or its Build Tools (the web development build tools workload): …
  - Or declare this scope's commands in its own .codelead/validation.json: …
✖ Validation failed: MSBuild could not run: Visual Studio's MSBuild is needed and was not found; next: Run it on Windows with Visual Studio or its Build Tools …
```

This failure is about a tool, not about the code. A repair cannot fix it. The way on is in the result: build on Windows, or declare the commands that do build the project. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).

When the code itself fails, for example a test, the way on is [`/repair`](../chat-commands/repair.md):

```output
✖ Validation failed: npm test exited 1; next: /repair
```

### Scopes

A repository can hold several projects, each with its own build. CodeLead finds the scope from the files that changed in this session. When more than one scope fits, it asks:

<!-- described, not captured -->
```output
Choose validation scope [1-3], or press Enter for 1:
```

The first scope is the widest, and it is the recommended answer. [`/validate --scope repo`](../chat-commands/validate.md) validates the whole repository.

## What it is not

- **Not the patch check.** The [patch check](the-patch-check.md) reads text before the apply. The gate runs your commands after the apply.
- **Not a judgement of quality.** The gate says that your checks pass. When your project has no tests, it can only say that the project builds. Add tests to make the gate stronger.
- **Not something CodeLead marks passed by itself.** `/validate --override` records a pass that a person checked by hand. CodeLead recommends no, because it did not see the pass, and the evidence record says that the pass was manual.

> [!NOTE]
> **For team leads:** the declared gate in `.codelead/validation.json` is the team's definition of done. Commit it with the code. Every evidence record lists the commands that ran, their labels and their exit codes.

## Details

- **The declared file.** Schema `codelead.validation/1`. Either `commands` (the folder that holds the file is the scope) or `scopes` keyed by folder, at the repository root. Each entry has `command`, `args`, and optional `label`, `cwd` (relative to the file's folder) and `platforms` (`darwin`, `win32`, `linux`).
- **One command, run once.** The same program with the same arguments in the same folder runs once, even when two scopes name it. The result names both sources.
- **A pin or an in-place transform** validates what it changed plus the declared gate, and asks no scope question.
- **A timeout** is inconclusive, not a code error: run `/validate` again.
- **Recovery before asking**. Sometimes `dotnet build` fails with MSB4019 on Visual Studio's targets. When Visual Studio's MSBuild is found, CodeLead runs the build once more with it, and says so.
- **Environment failures.** A failure that a patch cannot fix, such as a database that does not answer, ends `stopped on the environment`. No repair round runs for it.
- **In the evidence record.** `validation.status` and `validation.commands[]`, each with `command`, `status`, `exitCode`, and for a declared command its `label`.
- **Knowledge.** Each command that passes at a landing becomes an entry of the [knowledge base](knowledge-base.md): "Validation command passes: …".

## Related content

- [`/validate`](../chat-commands/validate.md), [`/implement`](../chat-commands/implement.md), [`/repair`](../chat-commands/repair.md), [`/build-step`](../chat-commands/build-step.md)
- [`codelead map`](../cli/codelead-map.md): the validation commands that the map found, the declared ones first
- [`codelead conventions --agents`](../cli/codelead-conventions.md): writes the gate's commands into `AGENTS.md` and `CLAUDE.md`
- [Declare validation commands](../how-to/declare-validation-commands.md)
- [Validation file](../reference/validation-file.md)
- [Build .NET Framework projects on Windows](../how-to/build-on-windows.md)
- [Repair](repair.md)
- [Results and stops](results-and-stops.md)
- [Fix a stop](../how-to/fix-a-stop.md)
