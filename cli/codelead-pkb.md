---
title: codelead pkb
description: The commands of the PKB service, the shared project knowledge base. Today its store, its sync, its embeddings, its search, and Ask with its stored questions.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb

**Applies to:** CodeLead 0.1 and later

`codelead pkb` holds the commands of the **PKB service**. PKB means project knowledge base. The service keeps what a project knows, for a team and its coding agents, and every answer names its sources. Today the command has eight subcommands:

- [`store`](codelead-pkb-store.md) runs the service's database on this machine.
- [`sync`](codelead-pkb-sync.md) sends a project's knowledge into it.
- [`embed`](codelead-pkb-embed.md) makes the embeddings that a search by meaning compares.
- [`search`](codelead-pkb-search.md) finds what the store holds about a question, and says why each hit was found.
- [`ask`](codelead-pkb-ask.md) answers a question. Each sentence of the answer names its sources and its trust. The question and its answer are stored.
- [`asks`](codelead-pkb-asks.md) lists the stored questions, each with its answers and the marks that say the evidence changed.
- [`reask`](codelead-pkb-reask.md) asks a stored question again.
- [`promote`](codelead-pkb-promote.md) makes an answer an entry, by a person's act.

> [!IMPORTANT]
> The PKB service itself does not run as a service yet: these commands use the store directly. Only `ask` and `reask` call a chat model, and only when something the search found clears the floor. `embed`, the end of `sync`, `search`, `ask` and `reask` call an embeddings model.

## Name

`codelead pkb` - The commands of the PKB service. Today: its store (`codelead pkb store`), its sync (`codelead pkb sync`), its embeddings (`codelead pkb embed`), its search (`codelead pkb search`), and Ask with its stored questions (`codelead pkb ask`, `asks`, `reask` and `promote`).

## Synopsis

```console
codelead pkb store up [--dir <FOLDER>] [--json]
codelead pkb store down [--dir <FOLDER>] [--json]
codelead pkb store status [--dir <FOLDER>] [--json]
codelead pkb sync [--project <ID>] [--dir <FOLDER>] [--dossier <FOLDER>] [--audit-pack <FOLDER>] [--dry-run] [--by <NAME>] [--no-embed] [--json]
codelead pkb embed [--project <ID>] [--dir <FOLDER>] [--dry-run] [--confirm-remote] [--by <NAME>] [--json]
codelead pkb search "<QUESTION>" [--project <ID>] [--dir <FOLDER>] [--dry-run] [--confirm-remote] [--json]
codelead pkb ask "<QUESTION>" [--project <ID>] [--dir <FOLDER>] [--by <NAME>] [--dry-run] [--confirm-remote] [--json]
codelead pkb asks [<QUESTION_ID>] [--project <ID>] [--dir <FOLDER>] [--json]
codelead pkb reask <QUESTION_ID> [--dir <FOLDER>] [--by <NAME>] [--dry-run] [--confirm-remote] [--json]
codelead pkb promote <ANSWER_ID> --by <NAME> [--dir <FOLDER>] [--json]
codelead pkb -h|--help
```

## Description

Each project already keeps its own [knowledge base](../concepts/knowledge-base.md) in its `.codelead` folder: the encyclopedia and the [evidence records](../concepts/evidence-records.md). The PKB service is the shared tier above it. It will collect that knowledge from several projects and several people, in one place on the team's machines.

| Subcommand | What it does |
|---|---|
| [`store`](codelead-pkb-store.md) | Runs the service's database on this machine. `up` starts it, `down` stops it, and `status` says what runs. |
| [`sync`](codelead-pkb-sync.md) | Sends a project's knowledge into the store. It scans every text for secrets, paths of this machine and private names first, and it logs what left which folder. At its end it makes the new pieces' embeddings. |
| [`embed`](codelead-pkb-embed.md) | Makes the embeddings of the pieces that have none, from the embeddings server that `pkb.json` names. A text is embedded once. |
| [`search`](codelead-pkb-search.md) | Finds what the store holds about a question: by words, by meaning, and in the code for a where, which file or how do I question. Each hit says why. |
| [`ask`](codelead-pkb-ask.md) | Answers a question from what the search found. A chat model writes the answer, and the service keeps only the sentences that cite their sources. When nothing clears the floor, no model is called. |
| [`asks`](codelead-pkb-asks.md) | Lists the stored questions with their dated answers. An answer whose evidence changed carries a mark. |
| [`reask`](codelead-pkb-reask.md) | Asks a stored question again. The new answer is stored beside the earlier ones. |
| [`promote`](codelead-pkb-promote.md) | Makes an answer a confirmed entry, with the answer as its source and the person's name in its history. |

### What exists today

- The store: MongoDB Community with its search process, mongot. One compose file runs them, and the store listens on `127.0.0.1` only.
- The store's collections and their indexes. See [The collections](codelead-pkb-store.md#the-collections).
- The configuration file `pkb.json`, and the env file that holds the store's passwords.
- The sync of a project. It reads the encyclopedia, the evidence records, the dossier, the System Model and the walk and parity reports. It also reads the decided board cards, the answers of `/ask` and the selected events of the sessions.
- The embeddings of the pieces, from an embeddings server on your machines.
- The search, with its reasons and its rule for "not in the knowledge base".
- Answers with citations, from a chat model on your machines. The service drops every sentence that cites nothing.
- Stored questions with their dated answers, the marks when the evidence changed, the re-ask and the promotion of an answer to an entry.

### What does not exist yet

- The service and its address (`codelead pkb serve`), and its web page.
- Adding documents and exporting a project (`codelead pkb add`, `codelead pkb export`).

These commands are not in this release. When you type one, CodeLead answers `error: unknown command` and ends with exit code `2`.

> [!NOTE]
> **New to this?** A store is a database program that keeps documents on disk. You start it once, and it runs in the background until you stop it. CodeLead runs it in Docker, so you install no database yourself.

## Options

- **`-h|--help`**

  Prints the help and ends.

The subcommands of `store` take `--dir <FOLDER>` and `--json`. See [`codelead pkb store`](codelead-pkb-store.md#options). The options of the others are on their pages: [`sync`](codelead-pkb-sync.md#options), [`embed`](codelead-pkb-embed.md#options), [`search`](codelead-pkb-search.md#options), [`ask`](codelead-pkb-ask.md#options), [`asks`](codelead-pkb-asks.md#options), [`reask`](codelead-pkb-reask.md#options) and [`promote`](codelead-pkb-promote.md#options).

## Examples

- Print the commands that exist today:

  ```console
  codelead pkb --help
  ```

  ```output
  Usage: codelead pkb [options] [command]

  The project knowledge base (PKB) service: its store (codelead pkb store up,
  down, status), its sync, its embeddings and its search (codelead pkb sync,
  embed, search), and Ask with its stored questions (codelead pkb ask, asks,
  reask, promote); the service comes later

  Options:
    -h, --help                     display help for command

  Commands:
    store                          The PKB's store: MongoDB Community with its
                                   search process (mongot), from one compose file
                                   in ~/.codelead/pkb/ (or --dir, or
                                   CODELEAD_PKB_HOME), on 127.0.0.1 only
    sync [options]                 Send a project's knowledge into the PKB store:
                                   read from the repositories pkb.json names,
                                   scanned for secrets, paths of this machine and
                                   private names, with a log of what left which
                                   folder
    embed [options]                Make the embeddings of a project's chunks that
                                   have none by the embeddings model pkb.json
                                   names, from its embeddings endpoint; a text
                                   embedded already is not embedded again
    search [options] [question]    Search a project's knowledge for a question:
                                   words, meaning and, for a where, which file or
                                   how do I question or a named identifier, the
                                   code; each hit with its reasons. No chat model
                                   is called
    ask [options] [question]       Ask a project's knowledge a question: each
                                   sentence of the answer cites its sources, with
                                   its trust; the chat model of pkb.json writes
                                   it, and the service drops what cites nothing.
                                   Nothing found is "not in the knowledge base",
                                   with no model call. The question and its
                                   answer are stored
    asks [options] [question id]   List a project's stored questions, each with
                                   its dated answers and their marks (the
                                   evidence an answer rested on changed); with a
                                   question's id, that question with every answer
                                   in full
    reask [options] [question id]  Ask a stored question again: what the
                                   knowledge base holds now, and the code as it
                                   is now; the new answer is stored beside the
                                   earlier ones, which stay as they were
    promote [options] [answer id]  Make a stored answer an entry of the project's
                                   knowledge, by a person's act: confirmed, the
                                   answer as its source, the person's name in its
                                   history
    help [command]                 display help for command
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

- Search a project for a question, without a store and without a model:

  ```console
  codelead pkb search "Why do invoice totals round half up?" --dry-run
  ```

  See [`codelead pkb search`](codelead-pkb-search.md#examples) for the result.

- See the prompt that an answer would be written from, without a model:

  ```console
  codelead pkb ask "How do I add another limit to the cold room store?" --dry-run
  ```

  See [`codelead pkb ask`](codelead-pkb-ask.md#examples) for the result, and for an answer.

- Check what runs:

  ```console
  codelead pkb store status
  ```

  ```output
  No store is set up in <home>/.codelead/pkb: codelead pkb store up sets one up.
  ```

## Exit codes

The codes of [`store`](codelead-pkb-store.md#exit-codes), [`sync`](codelead-pkb-sync.md#exit-codes), [`embed`](codelead-pkb-embed.md#exit-codes), [`search`](codelead-pkb-search.md#exit-codes), [`ask`](codelead-pkb-ask.md#exit-codes), [`asks`](codelead-pkb-asks.md#exit-codes), [`reask`](codelead-pkb-reask.md#exit-codes) and [`promote`](codelead-pkb-promote.md#exit-codes). `codelead pkb` without a subcommand prints its help and ends with `2`.

## Files

`codelead pkb store` writes its files in one folder, `<home>/.codelead/pkb/` by default. See [Files](codelead-pkb-store.md#files). The other subcommands read `pkb.json` there. `codelead pkb sync` adds a line to `.codelead/disclosures.jsonl` in each repository that it reads, when something left. See [Files](codelead-pkb-sync.md#files). `embed`, `search` and `ask` add one for each send to a remote model.

## See also

- [`codelead pkb store`](codelead-pkb-store.md): start, stop and check the store.
- [`codelead pkb sync`](codelead-pkb-sync.md): send a project's knowledge into the store.
- [`codelead pkb embed`](codelead-pkb-embed.md): make the embeddings of a project's pieces.
- [`codelead pkb search`](codelead-pkb-search.md): search a project for a question, each hit with its reasons.
- [`codelead pkb ask`](codelead-pkb-ask.md): answer a question, each sentence with its sources.
- [`codelead pkb asks`](codelead-pkb-asks.md), [`codelead pkb reask`](codelead-pkb-reask.md) and [`codelead pkb promote`](codelead-pkb-promote.md): the stored questions, asking one again, and making an answer an entry.
- [The knowledge base](../concepts/knowledge-base.md): what each project knows today, in its own folder.
- [Evidence records](../concepts/evidence-records.md)
- [`codelead board`](codelead-board.md): the board is a separate tool. The PKB holds what a project knows, the board what the team is doing.
- [Command-line reference](index.md)
