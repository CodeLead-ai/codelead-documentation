---
title: codelead pkb
description: The commands of the PKB service, the shared project knowledge base. Today its store, run with codelead pkb store, and its sync.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb

**Applies to:** CodeLead 0.1 and later

`codelead pkb` holds the commands of the **PKB service**. PKB means project knowledge base. The service will keep what a project knows, for a team and its coding agents, and every answer will name its sources. Today the command has two subcommands. [`store`](codelead-pkb-store.md) runs the service's database on this machine. [`sync`](codelead-pkb-sync.md) sends a project's knowledge into it.

> [!IMPORTANT]
> The PKB service itself does not exist yet. The store holds what `sync` sends, and nothing reads it back yet. No model is called.

## Name

`codelead pkb` - The commands of the PKB service. Today: its store (`codelead pkb store`) and its sync (`codelead pkb sync`).

## Synopsis

```console
codelead pkb store up [--dir <FOLDER>] [--json]
codelead pkb store down [--dir <FOLDER>] [--json]
codelead pkb store status [--dir <FOLDER>] [--json]
codelead pkb sync [--project <ID>] [--dir <FOLDER>] [--dossier <FOLDER>] [--audit-pack <FOLDER>] [--dry-run] [--by <NAME>] [--json]
codelead pkb -h|--help
```

## Description

Each project already keeps its own [knowledge base](../concepts/knowledge-base.md) in its `.codelead` folder: the encyclopedia and the [evidence records](../concepts/evidence-records.md). The PKB service is the shared tier above it. It will collect that knowledge from several projects and several people, in one place on the team's machines.

| Subcommand | What it does |
|---|---|
| [`store`](codelead-pkb-store.md) | Runs the service's database on this machine. `up` starts it, `down` stops it, and `status` says what runs. |
| [`sync`](codelead-pkb-sync.md) | Sends a project's knowledge into the store. It scans every text for secrets, paths of this machine and private names first, and it logs what left which folder. |

### What exists today

- The store: MongoDB Community with its search process, mongot. One compose file runs them, and the store listens on `127.0.0.1` only.
- The store's collections and their indexes. See [The collections](codelead-pkb-store.md#the-collections).
- The configuration file `pkb.json`, and the env file that holds the store's passwords.
- The sync of a project. It reads the encyclopedia, the evidence records, the dossier, the System Model and the walk and parity reports. It also reads the decided board cards, the answers of `/ask` and the selected events of the sessions.

### What does not exist yet

- The service and its address (`codelead pkb serve`).
- Questions and answers with citations (`codelead pkb ask`).
- Adding documents and exporting a project (`codelead pkb add`, `codelead pkb export`).

These commands are not in this release. When you type one, CodeLead answers `error: unknown command` and ends with exit code `2`.

> [!NOTE]
> **New to this?** A store is a database program that keeps documents on disk. You start it once, and it runs in the background until you stop it. CodeLead runs it in Docker, so you install no database yourself.

## Options

- **`-h|--help`**

  Prints the help and ends.

The subcommands of `store` take `--dir <FOLDER>` and `--json`. See [`codelead pkb store`](codelead-pkb-store.md#options). The options of `sync` are on [its page](codelead-pkb-sync.md#options).

## Examples

- Print the commands that exist today:

  ```console
  codelead pkb --help
  ```

  ```output
  Usage: codelead pkb [options] [command]

  The project knowledge base (PKB) service: its store (codelead pkb store up,
  down, status) and its sync (codelead pkb sync); the service and ask come later

  Options:
    -h, --help      display help for command

  Commands:
    store           The PKB's store: MongoDB Community with its search process
                    (mongot), from one compose file in ~/.codelead/pkb/ (or
                    --dir, or CODELEAD_PKB_HOME), on 127.0.0.1 only
    sync [options]  Send a project's knowledge into the PKB store: read from the
                    repositories pkb.json names, scanned for secrets, paths of
                    this machine and private names, with a log of what left which
                    folder
    help [command]  display help for command
  ```

- Start the store on this machine. Docker must run first:

  ```console
  codelead pkb store up
  ```

  The first line says where the store listens. See [`codelead pkb store`](codelead-pkb-store.md#examples) for the whole result.

- See what a sync of a project would send, without writing anything:

  ```console
  codelead pkb sync --project basic-store --dry-run
  ```

  See [`codelead pkb sync`](codelead-pkb-sync.md#examples) for the result, and for how to add a project to `pkb.json`.

- Check what runs:

  ```console
  codelead pkb store status
  ```

  ```output
  No store is set up in <home>/.codelead/pkb: codelead pkb store up sets one up.
  ```

## Exit codes

The codes of [`codelead pkb store`](codelead-pkb-store.md#exit-codes) and of [`codelead pkb sync`](codelead-pkb-sync.md#exit-codes). `codelead pkb` without a subcommand prints its help and ends with `2`.

## Files

`codelead pkb store` writes its files in one folder, `<home>/.codelead/pkb/` by default. See [Files](codelead-pkb-store.md#files). `codelead pkb sync` reads `pkb.json` there, and adds a line to `.codelead/disclosures.jsonl` in each repository that it reads. See [Files](codelead-pkb-sync.md#files).

## See also

- [`codelead pkb store`](codelead-pkb-store.md): start, stop and check the store.
- [`codelead pkb sync`](codelead-pkb-sync.md): send a project's knowledge into the store.
- [The knowledge base](../concepts/knowledge-base.md): what each project knows today, in its own folder.
- [Evidence records](../concepts/evidence-records.md)
- [`codelead board`](codelead-board.md): the board is a separate tool. The PKB holds what a project knows, the board what the team is doing.
- [Command-line reference](index.md)
