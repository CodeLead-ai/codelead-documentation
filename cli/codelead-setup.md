---
title: codelead setup
description: Install what CodeLead can install for itself, write a starter .env.example, and learn what to install by hand.
kind: reference
area: cli
date: 2026-11-20
---

# codelead setup

**Applies to:** CodeLead 0.1 and later

`codelead setup` installs the parts that CodeLead can install for itself, after it asks you once. It also writes a starter `.env.example` file and lists what you must install by hand.

## Name

`codelead setup` - Installs what CodeLead can install for itself and says what to install by hand.

## Synopsis

```console
codelead setup [--yes] [--dry-run]
codelead setup -h|--help
```

## Description

Run `codelead setup` once on a new machine, after [`codelead doctor`](codelead-doctor.md) shows what is missing.

`setup` makes a plan with two kinds of steps:

- **What CodeLead installs for itself**: Playwright's Chromium, the browser that [the walk](../modernization/walk-and-parity.md) and the [Analyst](../analyst/index.md) use. When it is already installed, the line says `in place`.
- **What CodeLead writes**: a starter `.env.example` in the current folder. Every line in it is a comment. It names the model settings, the Anthropic key and the consent to send code to a remote model.

Then `setup` asks one question: "Do the step marked "will do"?". Press Enter or type `y` to do it.

`setup` never installs the .NET SDK, Docker or SQL Server's tools. It says whether each one is installed, and the command that installs it on your system.

> [!NOTE]
> **New to this?** `.env.example` is a template. Copy it to `.env`, then remove the `#` in front of the lines that you use. CodeLead reads `.env`. Keep `.env` out of git, because it holds keys.

### Without a terminal

When nobody can answer the question (a script, a pipe), the answer is no. `setup` does nothing and says so. Use `--yes` for an unattended setup.

`setup` never writes over an existing `.env.example`.

> [!TIP]
> **For experienced users:** Chromium is installed with `npx playwright install chromium`. When that fails, `setup` says so and gives the command to run by hand.

## Options

- **`--yes`**

  Does the steps without asking. Use it in scripts and on build machines.

- **`--dry-run`**

  Says what would be done, and does nothing.

## Examples

- See what `setup` would do, without changing anything:

  ```console
  codelead setup --dry-run
  ```

  ```output
  codelead setup: what it installs for CodeLead, and what it leaves to you.

    in place  Playwright's Chromium: already installed for playwright 1.61.1
    will do   A starter .env.example: write <your project>/.env.example (the model, the Anthropic key, the remote-models consent; every line commented)

  Dry run: nothing was installed and nothing was written.

  Not installed by setup (install by hand when the client's code needs it):
    .NET SDK: SDK 10.0.302, installed
    Docker Desktop: Docker version 29.8.1, build 4a63305d74, daemon 29.9.1, installed and running
    SQL Server tools (sqlcmd): not installed; install: brew install sqlcmd (or https://learn.microsoft.com/sql/tools/sqlcmd/sqlcmd-download-install)

  Next: codelead doctor checks the whole machine (codelead doctor --probe-model also asks the model to answer).
  ```

- Do the steps without a question:

  ```console
  codelead setup --yes
  ```

  ```output
  codelead setup: what it installs for CodeLead, and what it leaves to you.

    in place  Playwright's Chromium: already installed for playwright 1.61.1
    will do   A starter .env.example: write <your project>/.env.example (the model, the Anthropic key, the remote-models consent; every line commented)

    done: wrote <your project>/.env.example; copy it to .env and fill in what you use
  …
  ```

- Run it where nobody can answer. Nothing is done, and the line says what to do next:

  ```console
  codelead setup < /dev/null
  ```

  ```output
  …
  Do the step marked "will do"? [Y/n] (no terminal to answer at: run codelead setup --yes for an unattended setup)

  Nothing was installed and nothing was written (codelead setup --yes does it without asking).
  …
  ```

- The `.env.example` that `setup` writes:

  ```output
  # CodeLead: copy this file to .env in the folder you run CodeLead from, and uncomment what you use.
  # .env holds keys: keep it out of git (add .env to .gitignore). codelead doctor says whether a key is
  # set, never its value.

  # The model. Nothing set: a local LM Studio server at http://localhost:1234/v1.
  # CODELEAD_BASE_URL=http://localhost:1234/v1
  # CODELEAD_MODEL=

  # Anthropic's API instead of a local model (codelead doctor --probe-model checks that the key answers):
  # CODELEAD_PROVIDER=anthropic
  # ANTHROPIC_API_KEY=

  # Consent to send the repository's context to a remote provider (Anthropic, OpenRouter) in unattended
  # runs. Set it only once the code's owner has agreed that the code may be sent to that provider.
  # CODELEAD_ALLOW_REMOTE_MODELS=1
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The steps were done, or nothing was asked, or the answer was no. |
| `1` | A step failed, for example the Chromium install. The line says what to run by hand. |

## Files

| File | What `setup` does with it |
|---|---|
| `.env.example` | Writes it in the current folder, only when it is not there. |
| Playwright's browser cache | Installs Chromium there when it is missing. |

## See also

- [`codelead doctor`](codelead-doctor.md): check the whole machine.
- [`codelead init`](codelead-init.md): prepare a project folder.
- [Install CodeLead](../get-started/install.md)
- [Environment variables](../reference/environment-variables.md)
- [Set up remote models](../how-to/set-up-remote-models.md)
