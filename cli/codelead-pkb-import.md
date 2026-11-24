---
title: codelead pkb import
description: Rebuild a project in a store from the folder that codelead pkb export wrote. Each file is checked and scanned first.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb import

**Applies to:** CodeLead 0.1 and later

`codelead pkb import` rebuilds a project from the folder of its export, in a store of the [PKB service](codelead-pkb.md). The store must hold nothing of the project yet. Every entry, question, answer, source, evidence record and piece of text comes back as it was stored. The marks of the answers and the history of each entry come back too.

## Name

`codelead pkb import` - Rebuilds a project from its export, into a store that holds nothing of it.

## Synopsis

```console
codelead pkb import <FOLDER> [--dir <FOLDER>] [--json]
codelead pkb import -h|--help
```

## Description

`import` reads the folder that [`codelead pkb export`](codelead-pkb-export.md) wrote. It reads the JSON files and the manifest only. The markdown files are for people to read.

Before it writes anything, `import` checks these:

1. Each file is the file that the export wrote: its SHA-256 is the one that the manifest gives.
2. `pkb.json` names the project. The project's repositories and their checkouts on this machine come from `pkb.json`, because the export names no checkout.
3. The scan with the project's own rules would change no text of the export. So a secret, a path of this machine or a private name never comes in.
4. The store holds nothing of the project.
5. Each document's id is the one that its own content gives. A later sync then finds the same documents.

When one check fails, nothing is written, and the words say which check failed.

The embeddings are not in an export. After an import, run [`codelead pkb embed`](codelead-pkb-embed.md) to make them.

> [!NOTE]
> **New to this?** An import is how you move a project to another store, or check an export. `codelead pkb serve --memory` runs a service over an empty store in memory. Import into it, export again, and compare the two folders.

`import` goes through the service when one is named. See [How the commands find the service](codelead-pkb-serve.md#how-the-commands-find-the-service).

## Options

- **`<FOLDER>`**

  The folder that `codelead pkb export` wrote.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The command then writes the store on this machine, not through the service. The default is `<home>/.codelead/pkb`.

- **`--json`**

  Prints the result as one JSON document: the project and the counts of what was written.

## Examples

These examples use the sample project `tally`, and a second service over an empty store in memory, on port `7426`.

- Import the export:

  ```console
  CODELEAD_PKB_URL=http://127.0.0.1:7426 codelead pkb import ./tally-export
  ```

  ```output
  Imported the project tally: 14 entries, 1 question with 1 answer, 12 sources, 2 evidence records and 30 chunks, as the export holds them.
  The chunks wait for their embeddings: codelead pkb embed makes them.
  ```

- Import it a second time. The store holds the project now, so the import is refused:

  ```console
  CODELEAD_PKB_URL=http://127.0.0.1:7426 codelead pkb import ./tally-export
  ```

  ```output
  The store holds the project tally already (14 entries, 12 sources, 2 evidence records, 30 chunks, 1 question): an import rebuilds a project into an empty store, and never merges.
  Nothing was written.
  ```

- Import a folder whose `entries.json` was changed by hand:

  <!-- described: the words that the command prints, as its tests check them -->
  ```output
  This is not a PKB export as it was written: entries.json is not the file the export wrote (its SHA-256 differs from the manifest's): it was changed since.
  Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The project was imported. |
| `1` | It started and failed. The words say why. |
| `2` | Refused: no folder, a folder that is not an export, a file changed since the export, a text the scan would change, a project that `pkb.json` does not name, a store that holds the project, or the service named does not answer. |

## See also

- [`codelead pkb export`](codelead-pkb-export.md): write the folder.
- [`codelead pkb embed`](codelead-pkb-embed.md): make the embeddings after an import.
- [`codelead pkb serve`](codelead-pkb-serve.md): `--memory` serves an empty store for a round trip.
- [PKB service API](../reference/pkb-service-api.md): `POST /api/projects/<id>/import`.
- [`codelead pkb`](codelead-pkb.md)
