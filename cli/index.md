---
title: Command-line reference
description: Every codelead command, what it is for, the options all commands share and what the exit codes mean.
kind: overview
area: cli
date: 2026-11-20
---

# Command-line reference

The `codelead` command runs CodeLead in a terminal. Each command has its own page with the same headings. A page holds the synopsis, the options, real examples, the exit codes and the files. This page lists every command by purpose.

**In short**
- You know when to use the command line and when to use the chat or the extension.
- You can check that CodeLead is installed and find the command for a task.
- You know the options every command takes and what an exit code means.

## When to use the command line

CodeLead has three ways in. They run the same engine.

| Way in | Best for |
|---|---|
| [CodeLead for VS Code](../extension/index.md) | Most people. Projects, the chat, the board and the Modernize wizard in the editor, with buttons. |
| [The chat](../extension/chat.md), or a terminal session with [`codelead interactive`](codelead-interactive.md) | Work in steps: you type a request or a chat command such as [`/implement`](../chat-commands/implement.md), and you answer questions as they come. |
| The command line (this reference) | One task with a clear end: check the machine, read a repository, run a gate in a script, run a long job without a person. |

> [!NOTE]
> **New to this?** A command-line program is one that you start by typing its name and its options in a terminal. CodeLead prints what it did and ends with an exit code (a number that says whether it worked).

Prefer the command line when you write scripts, when you run CodeLead in continuous integration, or when you want the exact facts. Prefer the chat when a model writes code, because the chat asks you before it changes files.

## Install and check

1. Install CodeLead. See [Install CodeLead](../get-started/install.md).
2. Open a terminal and check the version:

   ```console
   codelead --version
   ```

   ```output
   0.1.0 (commit 02022e35)
   ```

3. Check the machine with [`codelead doctor`](codelead-doctor.md). It prints one line for each thing that CodeLead needs, and how to install what is missing.

To see the help of any command, add `--help`: `codelead doctor --help`.

## The commands

### Set up and check

- [`codelead init`](codelead-init.md): creates the CodeLead folder in a project and reads the repository into its map.
- [`codelead doctor`](codelead-doctor.md): checks this machine and this folder for what CodeLead needs.
- [`codelead setup`](codelead-setup.md): installs what CodeLead can install for itself and writes a starter `.env.example`.
- [`codelead config`](codelead-config.md): prints the configuration that CodeLead uses, and where each value came from.
- [`codelead models`](codelead-models.md): lists the models that a provider offers.
- [`codelead licence`](codelead-licence.md): shows this folder's licence and its plan, with no network.

### Work in a session

- [`codelead interactive`](codelead-interactive.md): starts a session in the terminal, or a session over JSON lines for an editor.
- [`codelead chat`](codelead-chat.md): sends one message to a model and prints the answer.
- [`codelead sessions`](codelead-sessions.md): lists and exports the session logs.

### Change code

- [`codelead preflight`](codelead-preflight.md): analyzes a coding request with a model and writes a prompt for a coding agent.
- [`codelead scan`](codelead-scan.md): finds the files that a request is likely about, with no model. It also runs the security scan.
- [`codelead lint`](codelead-lint.md): checks a request that a person wrote, before a run, with no model.
- [`codelead implement`](codelead-implement.md): works the board's cards without a person, one session per card.
- [`codelead pr`](codelead-pr.md): opens a pull request for the last landed change, with its evidence record as the body.

### Read a repository

- [`codelead map`](codelead-map.md): reads the repository into its map: the stack, the validation commands, the modules.
- [`codelead library`](codelead-library.md): prints the project library: files, modules, symbols and relationships.
- [`codelead conventions`](codelead-conventions.md): extracts the repository's conventions with their evidence.

### Modernize

- [`codelead modernize`](codelead-modernize.md): the modernization program, one stage after another. It has eight subcommands.
- [`codelead model`](codelead-model.md): reads an old application from its sources into its System Model, with no model call.
- [`codelead register`](codelead-register.md): writes the behaviour register: every rule and behaviour found, with its check.
- [`codelead seed`](codelead-seed.md): writes a T-SQL script of sample rows for an empty database.

### Rebuild and verify

- [`codelead rebuild`](codelead-rebuild.md): prints a rebuild's plan. Its subcommands [`init`](codelead-rebuild-init.md) and [`run`](codelead-rebuild-run.md) make the rebuild and run it. [`journeys`](codelead-rebuild-journeys.md) compares the rebuild's journeys with the System Model's draft, and refreshes them.
- [`codelead rebuild-report`](codelead-rebuild-report.md): writes a rebuild's progress report as a page.
- [`codelead walk`](codelead-walk.md): walks the running application as a new user would, and reports each step.
- [`codelead parity`](codelead-parity.md): compares pages before and after a change, or checks that a rebuild is finished.
- [`codelead spec`](codelead-spec.md): specs in plain words: check, write, draft, walk, coverage and the report.

### The board and the Analyst

- [`codelead board`](codelead-board.md): Pinboard, the board between you and your coding agents.
- [`codelead analyst`](codelead-analyst.md): the CodeLead Analyst, which records and explores a running web application.

### The shared knowledge base

- [`codelead pkb`](codelead-pkb.md): the commands of the PKB service, the shared project knowledge base. [`codelead pkb store`](codelead-pkb-store.md) starts, stops and checks its store. [`codelead pkb sync`](codelead-pkb-sync.md) sends a project's knowledge into it. [`codelead pkb embed`](codelead-pkb-embed.md) makes the embeddings of its pieces. [`codelead pkb search`](codelead-pkb-search.md) searches it for a question, and says why each hit was found. [`codelead pkb ask`](codelead-pkb-ask.md) answers a question, each sentence with its sources, and stores it. [`codelead pkb asks`](codelead-pkb-asks.md) lists the stored questions and their marks. [`codelead pkb reask`](codelead-pkb-reask.md) asks one again. [`codelead pkb promote`](codelead-pkb-promote.md) makes an answer an entry. [`codelead pkb serve`](codelead-pkb-serve.md) runs the service, and [`codelead pkb status`](codelead-pkb-status.md) says what runs. [`codelead pkb entries`](codelead-pkb-entries.md) lists the entries with their trust. [`codelead pkb confirm`](codelead-pkb-confirm.md) and [`codelead pkb downgrade`](codelead-pkb-downgrade.md) change an entry's trust, by a person's act. [`codelead pkb export`](codelead-pkb-export.md) writes a project into files, and [`codelead pkb import`](codelead-pkb-import.md) rebuilds it from them.

## Options that every command shares

- **`-h|--help`**

  Prints the command's help and ends. Every command and every subcommand takes it. `codelead help <command>` prints the same.

- **`-V|--version`**

  Prints the version and the commit, then ends. Only the program itself takes it: `codelead --version`.

  When CodeLead runs from a source checkout, and the build is older than the checkout's commit, the line says so, with the way out:

  ```output
  0.1.0 (commit 5a8d11d7 with changes not committed); the build is older than the source: run npm run build (the build: commit f5ae05a3; the source: commit 5a8d11d7)
  ```

Most commands read their settings from `.codelead/config.json` in the current folder. Environment variables can override some of them. See [Configuration file](../reference/configuration.md) and [Environment variables](../reference/environment-variables.md).

> [!TIP]
> **For experienced users:** a packaged release asks once for a typed `yes` to its evaluation terms before the first command. Set `CODELEAD_ACCEPT_TERMS=yes` for a script or a scheduled job. `doctor`, `licence`, `--help` and `--version` never ask.

## Exit codes

Every command ends with an exit code. A script reads it to know what happened.

| Code | Meaning |
|---|---|
| `0` | The command did what it was asked. A check found nothing to fix. |
| `1` | The command ran and found a failure: a check failed, a gate did not pass, a step failed. An error that is not a refusal is also `1`, and so is a wrong option for most commands. |
| `2` | The command refused before it did anything: a file is missing, an answer or a configuration is refused. For [`lint`](codelead-lint.md), [`walk`](codelead-walk.md), [`model`](codelead-model.md), [`modernize`](codelead-modernize.md) and [`pkb`](codelead-pkb.md), a wrong or missing option is `2` too. The words say why and what to do. |
| `130` | You pressed Ctrl+C during a long run ([`implement`](codelead-implement.md), [`rebuild run`](codelead-rebuild-run.md)). |

Each command page lists its own codes, because some commands use them differently. See [Exit codes](../reference/exit-codes.md) for the full table and [Messages and what to do](../reference/messages.md) for the words that CodeLead prints.

## See also

- [How CodeLead works](../concepts/how-codelead-works.md)
- [Chat command reference](../chat-commands/index.md)
- [Files and folders](../reference/files-and-folders.md)
- [Quickstart: change existing code](../get-started/quickstart-existing-code.md)
