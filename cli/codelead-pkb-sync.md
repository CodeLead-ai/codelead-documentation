---
title: codelead pkb sync
description: Send a project's knowledge into the PKB store, read from its repositories, scanned for secrets, paths and private names, with a log of what left.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb sync

**Applies to:** CodeLead 0.1 and later

`codelead pkb sync` sends what a project knows into the store of the [PKB service](codelead-pkb.md). It reads the files that CodeLead keeps in each repository of the project. It scans every text for secrets, paths of this machine and private names before anything leaves. It logs what left which folder, as file names, hashes and counts.

> [!IMPORTANT]
> The store must run first: see [`codelead pkb store`](codelead-pkb-store.md). Nothing reads the knowledge back yet. Questions and answers (`codelead pkb ask`) come later.

## Name

`codelead pkb sync` - Sends a project's knowledge into the PKB store.

## Synopsis

```console
codelead pkb sync [--project <ID>] [--dir <FOLDER>] [--dossier <FOLDER>] [--audit-pack <FOLDER>] [--dry-run] [--by <NAME>] [--json]
codelead pkb sync -h|--help
```

## Description

Each repository keeps its own [knowledge base](../concepts/knowledge-base.md) in its `.codelead` folder and in `docs/codelead`. `sync` copies that knowledge into the shared store, one project at a time. A project is an entry of `projects` in `pkb.json`. It names the repositories it holds and where each one is checked out on this machine.

> [!NOTE]
> **New to this?** A sync copies. It never changes the files that it reads. You can run it as often as you like: what did not change is not sent twice.

### What it reads

In each repository, `sync` runs these readers, in this order. A reader that finds nothing says so in one line.

| Reader | What it reads | What it stores |
|---|---|---|
| `evidence` | `.codelead/evidence/ev-*.json` | Each [evidence record](../concepts/evidence-records.md) as it was written, and its text: request, outcome, files, validation. |
| `knowledge_store` | `.codelead/knowledge.json` | Each entry of the encyclopedia, with its trust level and history. |
| `dossier` | `docs/codelead/dossier/*.md` | Each section, in pieces by row and by paragraph. |
| `system_model` | `docs/codelead/rebuild/system-model.json` | A contract fact for each entry point and push message, with its lines in the code. |
| `walk` | `docs/codelead/parity/walk.json` | A fact for each journey of the walk, and one for each finding. |
| `parity` | `docs/codelead/parity/rebuild-parity.json` | A fact for each route and for each pinned feature. |
| `board` | `docs/codelead/board/items/*.md` | A decision for each card that a person decided. |
| `answers` | `docs/codelead/answers/*.json` | Each answer of `/ask`, as a question with one answer. |
| `sessions` | `.codeleadsessions/session-*/session.jsonl` | Only the validation results, the patch checks, the applies and the approvals. |

The readers also bring CodeLead's own [capability profiles](../concepts/knowledge-base.md#capability-profiles). They go into a project of their own, `codelead-shared`, because they are the same in every project.

A board card goes to the project that its `repo:` tag names. A card without a `repo:` tag belongs to the repository of its board.

### What never leaves

- **No code.** The service will read the code from its own checkout.
- **No prompt, no model reply, no command output.** Of a session, only the selected events leave, and only their facts.
- **No secret.** Passwords, keys and tokens become `<redacted>`, by the same rules as the audit pack. See [what the audit pack removes](../modernization/audit-pack.md). A secret in a shape that no rule knows can pass, so name your secret files in `sanitize.secret_files`.
- **No path of this machine.** The repository's folder becomes `<repo>`, your home folder `<home>`, the temporary folder `<tmp>`.
- **No private name.** Each name of the board's `private-names.txt` becomes `<name>`, and so does each name of `sanitize.private_names` in `pkb.json`.

This scan is the one that the [audit pack](../modernization/audit-pack.md) and the board use. It runs on every text before anything is sent. Then `sync` checks every text once more. When a text would still change, `sync` sends nothing.

### Trust levels

Each entry keeps the trust level of its source. The levels are those of the [encyclopedia](../concepts/knowledge-base.md#the-encyclopedia).

- An entry that the encyclopedia records as evidence-backed is evidence-backed in the store only when an evidence record names it. Without one, the store keeps it as reported. A later sync raises it when a record names it.
- A downgrade in the encyclopedia is made in the store too. A trust change that a person made in the store later is kept.
- A walk and a parity report are evidence-backed: they say what was observed.
- A System Model fact is evidence-backed when the model states it confirmed and its file is committed. Otherwise it is reported.
- A dossier section is evidence-backed when the dossier made it automatically. Otherwise it is reported.
- A decided card is confirmed, because a person decided.

### The second sync

The store keeps each file by its hash. A second sync of files that did not change changes nothing. Its last line says `Nothing changed`.

When a file changed, the store keeps its new version and marks the pieces of the old one as superseded. A search never finds a superseded piece. It stays in the store, so that an answer that cited it can still show it.

### One sync at a time

Only one sync of a project runs at a time. A second sync of the same project stops before it sends anything, and names the sync that runs. A sync that died gives the project back after one hour.

### The disclosure log

Each sync writes one line in the store's disclosure log for each project it wrote into. The line says who ran the sync and when. For each reader, it gives the repository, the folder, each file read with its SHA-256 hash, and the counts. It never holds the text of a file.

`sync` also adds the same line to `.codelead/disclosures.jsonl` in each repository that it read. A sync that fails part-way still writes its line, with the count `partial`, because some files left already.

## Options

- **`--project <ID>`**

  The project of `pkb.json` to sync. You can leave it out when `pkb.json` names one project only.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--dossier <FOLDER>`**

  A dossier folder to read as well. The folder must be in one of the project's repositories. For a folder in another place, add it to `folders` in `pkb.json`.

- **`--audit-pack <FOLDER>`**

  An audit pack's folder to read as well, in one of the project's repositories. The pack's `manifest.json` vouches for each file: a file with another hash is not read. The pack's sessions are never read.

- **`--dry-run`**

  Reads, scans and counts, and writes nothing. It needs no store: it sends to a store in memory, so its counts are those of a first sync.

- **`--by <NAME>`**

  Who runs the sync. The history of each entry and the disclosure log name it. The default is `codelead pkb sync`.

- **`--json`**

  Prints the result as one JSON document. The default is text.

## Examples

- See what a first sync would send, without a store and without writing anything:

  ```console
  codelead pkb sync --dry-run
  ```

  ```output
  Dry run of the project basic-store: read, scanned and checked; sent to a store in memory, so everything counts as new. Nothing was written.
  - evidence (webapp, .codelead/evidence): 2 files holding 2 records; sources 2 new, 0 updated, 0 unchanged; evidence 2 new, 0 updated, 0 unchanged; chunks 2 new, 0 updated, 0 unchanged.
  - knowledge_store (webapp, .codelead): 1 file holding 5 entries, 1 answer of /ask; sources 1 new, 0 updated, 0 unchanged; entries 4 new, 0 updated, 0 unchanged; chunks 4 new, 0 updated, 0 unchanged.
    Note: 1 answer of /ask, recorded as a trace entry: the answers reader brings it as a question, not as an entry.
  - dossier (webapp, docs/codelead/dossier): 4 files holding 3 sections, 10 chunks; sources 3 new, 0 updated, 0 unchanged; chunks 10 new, 0 updated, 0 unchanged.
  …
  Evidence join: 1 entry named by 1 evidence record of their workspace; 1 entry recorded evidence-backed that no evidence record names, stored reported until one does.
  Scan: 6 secrets, 4 paths of this machine and 2 private names replaced before anything was sent.
  The project basic-store in the store: 13 entries, 11 sources, 2 evidence records, 29 chunks, 1 question.
  ```

  Each line names a reader, the repository and the folder. It says how many files it read, what they hold, and what the store did with each kind.

- Sync the project into the store. The store must run:

  ```console
  codelead pkb sync --project basic-store
  ```

  <!-- described, not captured -->
  ```output
  Synced the project basic-store into the PKB store.
  - evidence (webapp, .codelead/evidence): 2 files holding 2 records; sources 2 new, 0 updated, 0 unchanged; evidence 2 new, 0 updated, 0 unchanged; chunks 2 new, 0 updated, 0 unchanged.
  …
  Changed: the counts above say what is new, updated or superseded.
  Disclosure: d-af41f913fe9ac9f5042d, d-ff1780ca8039c0fa82f6, in the store and in each repository read (.codelead/disclosures.jsonl).
  ```

- Run the same sync again. Nothing changed in the files, so nothing changes in the store:

  ```console
  codelead pkb sync --project basic-store
  ```

  <!-- described, not captured -->
  ```output
  Synced the project basic-store into the PKB store.
  - evidence (webapp, .codelead/evidence): 2 files holding 2 records; sources 0 new, 0 updated, 2 unchanged; evidence 0 new, 0 updated, 2 unchanged; chunks 0 new, 0 updated, 2 unchanged.
  …
  Nothing changed: every file read was synced already, as it is.
  ```

- Run it when the store does not run. Nothing is read or sent, and the exit code is `2`:

  ```console
  codelead pkb sync
  ```

  ```output
  The PKB store does not answer (connect ECONNREFUSED 127.0.0.1:27415). Start it with codelead pkb store up, then run this again.
  Nothing was read or sent.
  ```

- Name a project that `pkb.json` does not hold:

  ```console
  codelead pkb sync --project orders
  ```

  ```output
  pkb.json names no project orders: add it to "projects" (its id, its repositories with their checkouts), then run this again. It names basic-store.
  Nothing was read or sent.
  ```

- Add a project to `pkb.json`. This one reads a dossier that is kept in another repository, and keeps two more names private:

  ```json
  {
    "projects": [
      {
        "id": "basic-store",
        "name": "Basic Store",
        "repositories": [
          { "name": "webapp", "checkout": "<your checkout of the sample>", "branch": "main" }
        ],
        "folders": [
          { "reader": "dossier", "repository": "samples", "checkout": "<your checkout of the samples>", "path": "dossiers/basic-store" }
        ],
        "sanitize": { "private_names": ["<a name to keep private>"] }
      }
    ]
  }
  ```

  Then check what the sync would send:

  ```console
  codelead pkb sync --project basic-store --dry-run
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Done. The store took everything that was sent. |
| `1` | The sync ran, and the store refused part of what was sent. The lines that start with `Refused:` say what and why. |
| `2` | Refused before anything was read or sent: no `pkb.json`, no such project, a checkout that does not exist, the store does not answer or is not set up, another sync of the project runs, or an option is wrong. |

## Files

| File | What `sync` does with it |
|---|---|
| `<home>/.codelead/pkb/pkb.json` | Reads the project: its repositories, its folders and its private names. |
| `<home>/.codelead/pkb/.env` | Reads the store's password. |
| `.codelead/disclosures.jsonl`, in each repository read | Adds one line for each sync. |
| `docs/codelead/board/private-names.txt`, in each repository | Reads the names to keep private. Git ignores this file. |

### The project in pkb.json

| Key | What it holds |
|---|---|
| `id` | The project's short name: lowercase letters, digits, dot, dash or underscore. |
| `name` | The project's name for people. |
| `private` | `true` by default. A private project shares nothing with other projects. |
| `repositories` | Each repository: `name`, `checkout` (its folder on this machine) and `branch`. `readers` limits the readers that run in it. |
| `folders` | More folders to read: `reader` (for example `dossier` or `audit_pack`), `repository`, `path` in that repository, and `checkout` when the repository is not one of the project's. |
| `sanitize.private_names` | More names to keep private, besides those of the board. |
| `sanitize.secret_files` | Files whose contents are secrets. Their contents become `<redacted>` wherever they appear. |

## Details

- The store keeps a source for each file, by its SHA-256 hash. A changed file is a new version, linked to the one before.
- The text of each piece holds the names in the code twice: whole, and split into words. For example, `ITableDataSource` is also stored as `i table data source`.
- Each piece cites where it came from: the repository, the file, its lines, and the commit.
- A piece holds about 300 words at most. A piece never crosses a heading of a document.
- The store keeps each repository's checkout, so that the service can read the code there. An export of the project leaves the checkouts out.
- One sync sends each record and each entry once. When an audit pack holds a copy of the workspace's record, the copy is not sent again.

## See also

- [`codelead pkb`](codelead-pkb.md): the commands of the PKB service.
- [`codelead pkb store`](codelead-pkb-store.md): start, stop and check the store.
- [The knowledge base](../concepts/knowledge-base.md)
- [Evidence records](../concepts/evidence-records.md)
- [The audit pack](../modernization/audit-pack.md)
- [`codelead board`](codelead-board.md): the board, whose decided cards a sync reads.
