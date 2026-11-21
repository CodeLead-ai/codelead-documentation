---
title: Validation file
description: The format of .codelead/validation.json, codelead.validation/1 - where the gate looks for it, every key of an entry, scopes and platforms, and how it replaces the detected commands.
kind: reference
area: reference
date: 2026-11-20
---

# Validation file

**Applies to:** CodeLead 0.1 and later

`.codelead/validation.json` declares the commands of the [gate](../concepts/validation-and-the-gate.md): the build and test commands that must pass before a change lands. Without it, CodeLead detects the commands from the project's files. With it, the commands that you declare replace the detected ones for that folder. Declare them when your build needs a tool that CodeLead cannot guess: Visual Studio's MSBuild, a test runner of your own, or a script.

## The file

| Fact | Value |
|---|---|
| Path | `.codelead/validation.json` at the repository root, or in a scope's own folder. |
| Schema | `codelead.validation/1`. A file without `schema` reads as version 1. |
| Written by | You. [`/modernize --pin`](../chat-commands/modernize.md#--pin) adds its gate entries after a landing, and [`codelead rebuild init`](../cli/codelead-rebuild-init.md) writes one for a rebuild. |
| Read by | [`/validate`](../chat-commands/validate.md), [`/implement`](../chat-commands/implement.md), every repair round, every build step of a new app, [`codelead doctor`](../cli/codelead-doctor.md) and [`codelead conventions`](../cli/codelead-conventions.md). |
| In git | Yes. Your team and the unattended runs use the same gate. |

## A file

The Basic Store sample builds with Visual Studio's MSBuild on Windows. Its tests run with `vstest.console`. On a Mac, the team runs the tests with Mono:

```json
{
  "schema": "codelead.validation/1",
  "commands": [
    {
      "label": "build",
      "command": "msbuild",
      "args": ["sampleapp.slnx", "/p:Configuration=Debug", "/nologo"],
      "platforms": ["win32"]
    },
    {
      "label": "tests",
      "command": "vstest.console",
      "args": ["WebApplication1.Tests/bin/Debug/WebApplication1.Tests.dll"],
      "platforms": ["win32"]
    },
    {
      "label": "tests on Mono",
      "command": "mono",
      "args": ["packages/xunit.runner.console/tools/net452/xunit.console.exe", "WebApplication1.Tests/bin/Debug/WebApplication1.Tests.dll"],
      "platforms": ["darwin"]
    }
  ]
}
```

On Windows the gate runs `build` and `tests`. On macOS it runs `tests on Mono` only.

## Keys

The top level holds one of two keys:

| Key | Type | What it holds |
|---|---|---|
| `schema` | string | `codelead.validation/1`. |
| `commands` | list of entries | The commands of the folder that holds `.codelead/`. |
| `scopes` | object | At the repository root only: the commands of each scope, keyed by the scope's path from the root, such as `"WebApplication1.Tests"`. The key `"."` is the root itself. |

Each entry:

| Key | Type | Required | What it does |
|---|---|---|---|
| `command` | string | Yes | One executable, on the `PATH` or as a path. Never a shell line. An entry without it is left out. |
| `args` | list of strings | No | The arguments, one word each. |
| `label` | string | No | The gate's name for the command. The result message, the session and the [evidence record](evidence-record-format.md) use it. The default is the command and its arguments. |
| `cwd` | string | No | Where the command runs, relative to the folder that holds `.codelead/`. The default is the scope's folder. |
| `platforms` | list of strings | No | Runs the entry only on these platforms: `win32`, `darwin`, `linux`. The default is every platform. |
| `pinsDatabase` | string | No | Written by the database pins: the folder of their test project. Before it runs, the gate checks that the tests use the pins' own database, never the application's. |

## Where the gate looks

For a scope folder, the gate reads, in this order:

1. The scope's own `.codelead/validation.json`: its `commands`, or else its `scopes["."]`.
2. The repository root's `.codelead/validation.json`: its `scopes["<the scope's path>"]`. The root is the nearest folder with `.git`.

The first file that gives at least one command for this platform wins. A file that CodeLead cannot read is passed over: it is not JSON, it has another schema, or its commands are not a list. The detected commands then run, and the validation result says so in one line:

```output
Warning: .codelead/validation.json was not read (it is not valid JSON: Expected property name or '}' in JSON at position 2 (line 1 column 3)), so the detected commands ran instead; correct the file, or remove it, to run the declared commands.
```

The declared commands replace both the detected commands and the project profile's suggestions for that scope. Nothing is added to them, except a dependency install. See [Detected commands](#detected-commands).

Example: a root file that declares two scopes:

```json
{
  "schema": "codelead.validation/1",
  "scopes": {
    ".": [
      { "label": "build", "command": "msbuild", "args": ["sampleapp.slnx", "/nologo"] }
    ],
    "WebApplication1.Tests": [
      { "label": "tests", "command": "vstest.console", "args": ["bin/Debug/WebApplication1.Tests.dll"] }
    ]
  }
}
```

## The scope of a validation

[`/validate`](../chat-commands/validate.md) chooses the scope from the files that the change touched, and shows the choice:

```output
Run validation commands?

Validation scope: repo root

Commands:
  1. build
  2. tests

Run these commands? [Y/n]
```

- Commands declared at the repository root outrank the guesses for each project. When a touched project has its own declared gate too, both run, the project's first.
- When scopes are combined, a command found in more than one of them runs once. Two commands are the same when they run the same program with the same arguments on the same files. The command keeps the declared entry's label, and the record lists every place it was found in `sources`.
- A dependency install of a combined scope runs before any build or test.
- `/validate --scope repo`, `--scope current` and `--scope <path>` choose the scope by hand.

## Detected commands

Without a declared command for the scope, CodeLead finds the commands in one of two ways.

**At the repository root**, CodeLead takes the commands of the project profile first. The profile is CodeLead's reading of the repository, the same one that [`codelead scan`](../cli/codelead-scan.md) prints after `Validation:`. It lists, in this order, each one that applies:

| The repository holds | Commands |
|---|---|
| `package.json` | `npm run typecheck`, `npm run test`, `npm run build`, for each script that exists. With `pnpm-lock.yaml` or `yarn.lock`, `pnpm` or `yarn` takes the place of `npm`. |
| .NET projects | `dotnet build`, and `dotnet test` when a test project is there. A non-SDK .NET Framework project builds with Visual Studio's MSBuild instead, when the machine has it. A `test/` folder counts as a test project only when it holds .NET files, so a Node.js repository never gets `dotnet test`. |
| pytest | `python -m pytest`. |
| `go.mod`, `Cargo.toml`, `pom.xml` or Gradle files | `go test ./...`, `cargo test`, `mvn test`, or the Gradle `test` task. |

For example, in the rebuild of the Basic Store sample, `codelead scan` prints `Validation: npm run test, npm run build, dotnet build, dotnet test`. That rebuild also declares its own gate, which `/validate` runs in their place. The profile runs no lint. **In another scope**, or when the profile lists nothing, CodeLead reads the scope's own files:

| The scope holds | Commands, in order |
|---|---|
| `package.json` | `npm run typecheck`, `npm test`, `npm run build`, `npm run lint`, for each script that exists. |
| a `.sln`, `.csproj` or `.fsproj` | `dotnet build`, and `dotnet test` when a test project is there. A non-SDK .NET Framework project builds with Visual Studio's MSBuild instead, when the machine has it. |
| `pyproject.toml`, `pytest.ini` or `requirements.txt`, with pytest | `python -m pytest`. |

When a change adds or changes dependencies, `npm install --no-audit --no-fund` runs first. `CODELEAD_INSTALL_TIMEOUT_MS` bounds it, 15 minutes by default.

## What success and failure look like

A file with a passing `build` and a failing `tests` gives this result in the session:

```output
Validation

Status: failed
Scope: repo root

Failed command:
  tests

Exit code: 1

Output preview:
  1 test failed

Next:
  Fix the issue manually, or paste the output with /paste --analyze.
✖ Validation failed: tests exited 1; next: /repair
```

The command's result names each declared entry by its label, in the [session protocol](session-io-protocol.md) too:

```json
{"outcome":"failed","headline":"Validation failed: tests exited 1","facts":["Failed command: tests (exit 1)","Scope: repo root","Ran: 1 of 2 commands passed before it"],"next":["/repair","Or fix it by hand, then /validate"],"detail":{"block":"validation"}}
```

A scope with no command at all ends with `No validation commands found`, and the way out is `/validate --override` after you check by hand.

## Rules

- **Never a shell line.** `"command": "npm run build"` is one executable named `npm run build`, which does not exist. Write `"command": "npm", "args": ["run", "build"]`.
- **On Windows**, `npm`, `npx` and a `.cmd` file run through `cmd.exe`. An argument with a space, a quote or `& | < > ^ %` can be read as shell syntax there. Keep such arguments out, or call the program directly.
- **Keep the pins' entries.** After `/modernize --pin` lands, the file holds the pins' entries, for example `pins in-memory: webapp.Characterization (dotnet test, net48)`. The gate runs them on every later change. See [Pins](../modernization/pins.md).
- **No key in the file.** The file is committed. A command reads its secrets from the environment.

## See also

- [Validation and the gate](../concepts/validation-and-the-gate.md)
- [Declare validation commands](../how-to/declare-validation-commands.md)
- [`/validate`](../chat-commands/validate.md) and [`/repair`](../chat-commands/repair.md)
- [Build .NET Framework projects on Windows](../how-to/build-on-windows.md)
- [Evidence record format](evidence-record-format.md)
- [Messages and what to do](messages.md)
