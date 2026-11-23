---
title: codelead modernize run
description: Run one stage of the modernization program, or run it again.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize run

**Applies to:** CodeLead 0.1 and later

`codelead modernize run <stage>` runs one stage of the [modernization program](codelead-modernize.md), and only that stage. Use it to run a stage again after you fixed the cause of its failure.

## Name

`codelead modernize run` - Runs one stage, again if it ran before.

## Synopsis

```console
codelead modernize run <STAGE> [--no-install]
codelead modernize run -h|--help
```

## Description

Each stage's problem ends with the line to run after the fix, for example `Then: codelead modernize run pins`. That line is this command.

`run` runs the stage that you name, whatever its status: pending, waiting, failed or done. It does not run the stages after it. Run [`codelead modernize next`](codelead-modernize-next.md) for those.

> [!NOTE]
> **New to this?** A stage that ran before is run again from its start. The files that it wrote before are written again. The Pin stage is different: the pins that landed stay, and it pins the facts that remain. Over an applied increment that its gate refused, it runs that gate first and composes nothing again.

> [!TIP]
> **For experienced users:** the chat runs the same stage with `/modernize --run <stage>`. See [`/modernize`](../chat-commands/modernize.md).

The command is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, it is refused before anything runs, with exit code 2. See [Licence and plans](../concepts/licence-and-plans.md).

## Arguments

- **`<STAGE>`**

  The stage's id: `read`, `recordings`, `pins`, `in-place`, `model`, `rebuild-init`, `rebuild` or `audit`. See [The eight stages](../modernization/the-eight-stages.md).

## Options

- **`--no-install`**

  The rebuild's initialization does not run `npm install`. It applies to the `rebuild-init` stage.

## Examples

- Run the `model` stage by hand. When every finding of the gate is an ASP.NET Identity page that nothing reaches, the stage records the reasons and reads the model again:

  ```console
  codelead modernize run model
  ```

  ```output
  ▸ Extract the model (codelead modernize run model)
  System Model of webapp (store-modernize/WebApplication1, inventory: project-file): 289 nodes, 189 edges; 63 entry points, 154 caller records; 10 not resolved, 0 leading nowhere.
  …
  Gate: not passed: 36 findings to resolve (36 unexplained-orphan).
  …
  The extraction gate: 36 findings, every one the ASP.NET Identity scaffold that nothing live reaches (Account/ConfirmEmail, Account/ForgotPassword, …
  …
  Gate: passed.
  …
  Rebuild map: webapp
  63 features — 22 screens, 0 fragments, 16 commands, 5 content screens, 20 authentication actions; 0 views carry jQuery; 0 facts (0 pinned), 0 anchored tests.
  …
    Done: The System Model of webapp: the gate passed (36 Identity scaffold findings recorded as intended); the feature map written.
  ```

- Run the `pins` stage again after a fix. Here the fix is not made yet, so the stage fails again with the same way out:

  ```console
  codelead modernize run pins
  ```

  ```output
  ▸ Pin the contracts (codelead modernize run pins)
    Failed: The pins of webapp need Visual Studio's MSBuild, and it is not on this machine.
  …
    Then: codelead modernize run pins
  ```

- Name a stage that does not exist:

  ```console
  codelead modernize run nothing
  ```

  ```output
  Refused: nothing is not a stage of the program: read, recordings, pins, in-place, model, rebuild-init, rebuild, audit.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The stage is done, or it waits for a person. |
| `1` | The stage failed. |
| `2` | The command was refused: a folder without the Pro plan, an unknown stage, a program that runs in another process, a state file that cannot be read. |

## Files

`run` reads and writes `.codelead/modernization.json`, and the stage writes its own files. See [`codelead modernize`](codelead-modernize.md#files).

## See also

- [`codelead modernize`](codelead-modernize.md)
- [`codelead modernize status`](codelead-modernize-status.md): read the stage's problem.
- [`codelead modernize next`](codelead-modernize-next.md): run the stages after it.
- [The eight stages](../modernization/the-eight-stages.md)
- [Fix a stop](../how-to/fix-a-stop.md)
