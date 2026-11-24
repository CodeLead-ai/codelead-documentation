---
title: codelead pkb export
description: Write a project's knowledge into a folder of files, a markdown file per entry and per question, the JSON files and a manifest.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb export

**Applies to:** CodeLead 0.1 and later

`codelead pkb export` writes what the store of the [PKB service](codelead-pkb.md) holds about a project into a folder. You can read the files, keep them, or commit them. [`codelead pkb import`](codelead-pkb-import.md) rebuilds the project from them, in another store.

## Name

`codelead pkb export` - Writes a project's entries, questions, sources, evidence records and chunks into a folder, with a manifest.

## Synopsis

```console
codelead pkb export --out <FOLDER> [--project <ID>] [--dir <FOLDER>] [--json]
codelead pkb export -h|--help
```

## Description

The folder holds these files:

| File | What it holds |
|---|---|
| `entries/<id>.md` | One entry, in words: its trust, scope, text, sources, conflicts and history. |
| `questions/<id>.md` | One stored question, with every answer it had and their marks. |
| `entries.json` | Every entry, as stored. |
| `questions.json` | Every question, each with its answers, as stored. |
| `sources.json` | Every source: where it came from, its hash and its version. |
| `evidence.json` | Every evidence record. |
| `chunks.json` | Every piece of text that a search reads, without its embedding. |
| `project.json` | The project: its name, and each repository by its name and branch. |
| `manifest.json` | Each other file with its SHA-256, so that an import can check it. |

The files hold what the store holds, and the store holds only what passed the scan at the sync. So the files hold no secret, no path of this machine and no private name. A repository is named, never its checkout.

The embeddings are not in the files, because they are long lists of numbers. After an import, [`codelead pkb embed`](codelead-pkb-embed.md) makes them again.

> [!NOTE]
> **For team leads:** the files are the product. Committing them to a repository is a person's act, and `export` never commits.

### The folder

`--out` must name an empty folder, a folder that does not exist yet, or the folder of an earlier export of the same project. In an earlier export, `export` removes the files that its manifest lists, and only those. Then it writes the new files. A folder that holds other files is refused, and nothing is written.

`export` goes through the service when one is named, and writes the files on this machine. See [How the commands find the service](codelead-pkb-serve.md#how-the-commands-find-the-service).

## Options

- **`--out <FOLDER>`**

  The folder to write into. Required.

- **`--project <ID>`**

  The project to export. You can leave it out when `pkb.json` names one project only, or when `pkb.project` names it.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The command then reads the store on this machine, not the service. The default is `<home>/.codelead/pkb`.

- **`--json`**

  Prints the result as one JSON document: the project, the folder, the number of files and the counts.

## Examples

These examples use the sample project `tally`, through a service on port `7425`.

- Export the project:

  ```console
  codelead pkb export --project tally --out ./tally-export
  ```

  ```output
  Exported the project tally into ./tally-export: 14 entries, 1 question with 1 answer, 12 sources, 2 evidence records and 30 chunks.
  22 files: a markdown file per entry and per question, the JSON files, and manifest.json with each file's SHA-256.
  Everything in it passed the scan when it came into the store. The vectors are not in it: after an import, codelead pkb embed makes them again.
  Committing these files is a person's act.
  ```

- Export into a folder that holds other files. It is refused:

  ```console
  codelead pkb export --project tally --out ./busy
  ```

  ```output
  The export cannot be written: --out names a folder that holds other files: give an empty folder, or the folder of an earlier export of the project.
  Nothing was written.
  ```

- Check an export by a round trip. Import it into a service over an empty store, export it again, and compare:

  ```console
  CODELEAD_PKB_URL=http://127.0.0.1:7426 codelead pkb import ./tally-export
  CODELEAD_PKB_URL=http://127.0.0.1:7426 codelead pkb export --project tally --out ./tally-again
  diff -r ./tally-export ./tally-again
  ```

  `diff` prints nothing: the two folders hold the same files. The service on port `7426` ran with `codelead pkb serve --memory`.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The files were written. |
| `1` | It started and failed. The words say why. |
| `2` | Refused: no `--out`, a folder of other files, no `pkb.json`, no such project, the store does not answer, or the service named does not answer. |

## See also

- [`codelead pkb import`](codelead-pkb-import.md): rebuild a project from its export.
- [PKB service API](../reference/pkb-service-api.md): `GET /api/projects/<id>/export`, with `format=md` or `format=json`.
- [`codelead pkb sync`](codelead-pkb-sync.md#what-never-leaves): the scan that everything passed.
- [`codelead pkb`](codelead-pkb.md)
