---
title: Declare validation commands
description: See which build and test commands the gate runs, declare your own in .codelead/validation.json, and check the result with /validate.
kind: how-to
area: how-to
date: 2026-11-20
---

# Declare validation commands

This guide declares the commands of [the gate](../concepts/validation-and-the-gate.md): your project's own build and tests, which decide that a change is done. CodeLead finds commands by itself from the project's files. Declare them when its choice does not fit, or when the team wants one written definition of done.

**In short**
- [`/validate`](../chat-commands/validate.md) lists the commands of the gate before it runs them.
- You declare the commands in `.codelead/validation.json`. Declared commands replace what CodeLead finds by itself.
- `/validate` shows your labels, runs the commands, and ends with the verdict.

## Before you begin

- You ran [`codelead init`](../cli/codelead-init.md) in the repository.
- You know the commands that build and test your project, and they work in a terminal.

> [!NOTE]
> **New to this?** A *scope* is a part of the repository with its own build and tests. A small project has one scope: the repository root. CodeLead validates the scope of the files that a change touched.

## See what the gate runs today

1. Start a session with [`codelead interactive`](../cli/codelead-interactive.md), or open the chat in the extension.
2. Type `/validate --scope repo`. CodeLead lists the commands and asks before it runs them. In a small Node.js project with a `test` and a `build` script:

   ```output
   Run validation commands?

   Validation scope: repo root

   Commands:
     1. npm run test
     2. npm run build

   Run these commands? [Y/n]
   ```

3. Press **Enter** to run them, or type `n` to stop here. A pass ends with `✔ Validation passed: 2 commands`.

When the list is right, you need no file. When a command is missing, or wrong for your project, declare the commands.

## Declare the commands

1. Create the file `.codelead/validation.json` at the repository root.
2. List each command as one program and its arguments. A command is never a shell line. Give each one a `label`, which the results and the evidence records use:

   ```json
   {
     "schema": "codelead.validation/1",
     "commands": [
       { "label": "unit tests", "command": "npm", "args": ["test"] },
       { "label": "type check", "command": "npx", "args": ["tsc", "--noEmit"] }
     ]
   }
   ```

3. Commit the file. It is the team's definition of done, and every machine reads the same one.

A repository with several projects declares each scope under `scopes`, keyed by the folder from the root. The key `"."` is the root:

```json
{
  "schema": "codelead.validation/1",
  "scopes": {
    ".": [
      { "label": "build sampleapp.slnx (MSBuild)", "command": "msbuild", "args": ["sampleapp.slnx", "-restore", "-nologo", "-v:minimal"] }
    ],
    "WebApplication1.Tests": [
      { "label": "tests", "command": "dotnet", "args": ["test"], "platforms": ["win32"] }
    ]
  }
}
```

`platforms` limits an entry to `win32`, `darwin` or `linux`. `cwd` runs an entry in another folder, relative to the folder that holds `.codelead`. See [Validation file](../reference/validation-file.md) for every key.

> [!IMPORTANT]
> The pins and the transforms of a modernization add their own entries to this file. Keep those entries when you edit it.

## Check the result with /validate

1. Type `/validate --scope repo` again. The list now shows your labels:

   ```output
   Commands:
     1. unit tests
     2. type check
   ```

2. Press **Enter**. Every command runs in order, and the result names the one that failed. Here the type check exits with code 2:

   ```output
   Validation

   Status: failed
   Scope: repo root

   Failed command:
     type check

   Exit code: 2
   …
   ✖ Validation failed: type check exited 2; next: /repair
   ```

3. Fix the cause, or run [`/repair`](../chat-commands/repair.md), then run `/validate` again until it passes.

## Check that it worked

- `/validate` lists your labels, not the commands that CodeLead found by itself.
- When `/validate` still lists the found commands, CodeLead did not read your file. The validation result then has one line that names the file and the reason, for example `Warning: .codelead/validation.json was not read (it is not valid JSON: …), so the detected commands ran instead; …`. Check the commas and the `schema` line.
- After a change, the [evidence record](../concepts/evidence-records.md) lists each command with its label and its exit code, under `validation.commands`.

## Details

- **What CodeLead finds by itself.** It uses the scripts `typecheck`, `test` and `build` of `package.json`. It uses `dotnet build` and `dotnet test`, or Visual Studio's MSBuild for an old .NET Framework project. It also knows `python -m pytest` and `go test ./...`. See [Validation and the gate](../concepts/validation-and-the-gate.md#where-the-commands-come-from).
- **Where the gate looks.** First the scope's own `.codelead/validation.json`, then the `scopes` of the root's file. The first file that gives a command for this platform wins.
- **A command found twice** runs once, even when two scopes name it.
- **Other places that show commands.** The [repository map](../concepts/repository-map.md) and the block of `codelead conventions --agents` list commands too. The block names only your declared commands. The map lists them first, each marked `(declared in .codelead/validation.json)`. It marks each command that it found by itself `(detected; the gate runs the declared commands in its place)`.
- **A timeout** is not a failure of the code. Run `/validate` again.
- **An old .NET Framework project** builds with Visual Studio's MSBuild. See [Build .NET Framework projects on Windows](build-on-windows.md).

## See also

- [Validation and the gate](../concepts/validation-and-the-gate.md)
- [Validation file](../reference/validation-file.md)
- [`/validate`](../chat-commands/validate.md) and [`/repair`](../chat-commands/repair.md)
- [Build .NET Framework projects on Windows](build-on-windows.md)
- [Fix a stop](fix-a-stop.md)
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
