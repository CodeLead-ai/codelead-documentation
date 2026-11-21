---
title: codelead modernize answer
description: Record the answers that the stages of the modernization program need, or accept the pins as they stand, without running anything.
kind: reference
area: cli
date: 2026-11-20
---

# codelead modernize answer

**Applies to:** CodeLead 0.1 and later

`codelead modernize answer` records the answers that the stages of the [modernization program](../modernization/the-program.md) need. They say where to make the rebuild, from which template and for which framework. They also say what to do with the accounts, the database, the pins' own database and some security findings. The command runs nothing.

## Name

`codelead modernize answer` - Records answers for the program; nothing is run.

## Synopsis

```console
codelead modernize answer <KEY=VALUE>...
codelead modernize answer -h|--help
```

## Description

Some decisions only a person can make. A stage that needs one waits, and its problem names the answer to give. You give it with `answer`, then run the line that the problem names, often [`codelead modernize next`](codelead-modernize-next.md).

The program takes eight keys, from one table:

| Key | Kind | Values | Default | When it is needed |
|---|---|---|---|---|
| `rebuildFolder` | stored | A folder | `../<folder>-rebuild` | Always. A new or empty folder beside the old application. |
| `template` | stored | A template name or path | `dotnet-next-shadcn` | Always. |
| `framework` | stored | `net10.0` or `net8.0` | The template's | Optional. |
| `identity` | stored | `keep` or `drop` | None | When the old application signs users in. |
| `database` | stored | A connection string's name, or `sample-seed` | None | When the old application reaches a database. |
| `pinsDatabase` | stored | `done`, `env:<NAME>`, or the path of a copy of the database file | None | When the database pins need a database on a server. |
| `security` | stored | `fix-none`, `fix-all` or `decided` | None | When security findings wait for a person in the Upgrade in place stage. |
| `pins` | act | `enough` | | Only while the Pin stage waits or failed. |

- **A stored answer** is kept under `answers` in `.codelead/modernization.json`. You can give it at any time, whatever stage waits. A stage reads it when it runs.
- **An act** acts on its stage, and is never kept. `pins=enough` marks the Pin stage `done` with the coverage as it stands, and you go on. At any other time, it is refused.

What the values mean:

- `identity=keep`: the new API keeps the accounts, with ASP.NET Core Identity over the old user tables. `identity=drop`: the rebuild has no sign-in, and the account features are dropped.
- `database=<name>`: the name of the connection string that the new API reads. The string itself stays out of the repository. `database=sample-seed`: the new API starts on a sample database, reset before every journey.
- `pinsDatabase=done`: you wrote the connection in the pins' git-ignored connections file. `env:<NAME>`: an environment variable holds it. See [Pins](../modernization/pins.md#the-pins-own-database).
- `security=decided`: you waived or fixed each finding yourself. `fix-all`: the stage fixes them all. `fix-none`: they stay open. See [Waive a security finding](../how-to/waive-a-security-finding.md).

> [!NOTE]
> **New to this?** `identity` and `database` are the same decisions that [`codelead rebuild init`](codelead-rebuild-init.md) asks for with `--identity` and `--database`. The program keeps them in its state file, so that you give them once.

> [!TIP]
> **For experienced users:** a wrong key or a wrong value is refused, and nothing is recorded. The refusal of an unknown key lists all eight. A connection string is never an answer, because the shell keeps the command line in its history.

## Arguments

- **`<KEY=VALUE>...`**

  One or more `key=value` pairs, separated by spaces. The keys are `rebuildFolder`, `template`, `framework`, `identity`, `database`, `pinsDatabase`, `security` and `pins`.

## Examples

- Keep the accounts and start the rebuild on a sample database:

  ```console
  codelead modernize answer identity=keep database=sample-seed
  ```

  ```output
  Answers recorded: identity=keep, database=sample-seed.
  ```

- Record that you decided each security finding yourself. When a stage waits, the last line names how it goes on:

  ```console
  codelead modernize answer security=decided
  ```

  <!-- described, not captured -->
  ```output
  Answers recorded: security=decided.
  Then: codelead modernize run in-place (Transform in place waits).
  ```

- Accept the pins before the Pin stage ran. CodeLead refuses, records nothing, and the exit code is 2:

  ```console
  codelead modernize answer pins=enough
  ```

  ```output
  Refused: pins=enough: Pin the contracts (pins) has not run (it is pending): pins=enough is taken only while the stage waits for you or failed; run it first (codelead modernize run pins), then answer again if you choose. Nothing was recorded.
  ```

- Give a value that the key does not take:

  ```console
  codelead modernize answer security=maybe
  ```

  ```output
  Refused: security=maybe: fix-none (the findings that change who may do what stay open), fix-all (the in-place stage fixes them too) or decided (you waived or fixed each yourself). Nothing was recorded.
  ```

- Give a key that the program does not take. The refusal lists every pair that it takes:

  ```console
  codelead modernize answer nonsense=1
  ```

  ```output
  Refused: nonsense: not an answer the program takes (rebuildFolder=<folder>, template=<name|path>, framework=net10.0|net8.0, identity=keep|drop, database=<connection name>|sample-seed, pinsDatabase=done|env:<NAME>|<copy's path>, security=fix-none|fix-all|decided, pins=enough). Nothing was recorded.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The answers were recorded, or the pins were accepted. |
| `2` | A key or a value was refused, or `pins=enough` came while the Pin stage neither waits nor failed. Nothing was recorded. |

## Files

`answer` writes the stored answers into `.codelead/modernization.json`. `pins=enough` writes the Pin stage's status and the time of the acceptance. See [Modernization state file](../reference/modernization-state-file.md#answers).

## See also

- [`codelead modernize`](codelead-modernize.md)
- [`codelead modernize status`](codelead-modernize-status.md): see the answers.
- [`codelead rebuild init`](codelead-rebuild-init.md): the stage that uses the rebuild's answers.
- [Waive a security finding](../how-to/waive-a-security-finding.md): the `security` answer.
- [The modernization program](../modernization/the-program.md)
- [Rebuild manifest](../reference/rebuild-manifest.md): where the answers end up.
