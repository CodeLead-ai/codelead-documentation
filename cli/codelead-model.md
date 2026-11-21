---
title: codelead model
description: Read an old application from its sources into its System Model, with no model call, and evaluate the extraction gate.
kind: reference
area: cli
date: 2026-11-20
---

# codelead model

**Applies to:** CodeLead 0.1 and later

`codelead model` reads an old application from its source code into its **System Model**. The System Model lists every entry point, screen, script and database table, and the edges between them, each with its file and line. The command calls no model. It then checks the result with the extraction gate.

## Name

`codelead model` - Reads an old application from its sources into its System Model.

## Synopsis

```console
codelead model --source <REPO> [--project <NAME>] [--artifact <FOLDER>] [--project-root <FOLDER>]
               [--schema-dump <PATH>]... [--schema-from <KEY> [--consent]]
               [--no-write] [--json]
codelead model -h|--help
```

## Description

The [System Model](../modernization/system-model.md) is the base of a rebuild. CodeLead reads the old application's code with a recipe for its stack, for example `aspnet-mvc` and `jquery-browser`. It records:

- the entry points (routes and actions), the screens (views), the client code and the triggers (links, forms, redirects);
- the edges between them, each with its `file:line` evidence;
- what could not be resolved, listed as **unknown**;
- the database, as the code reads and writes it: the tables, the columns, the keys.

The name says "model", but no AI model is called. The words "System Model" mean this map of the application.

> [!NOTE]
> **New to this?** Before CodeLead rebuilds an old application, it must know everything that the application does. The System Model is that list, read from the code, with the place in the code for each item.

### The extraction gate

The **extraction gate** checks the model. For example, an **orphan** is an entry point that nothing live reaches and that nobody has explained. Each finding of the gate has a key. You explain a finding by writing a reason under its key in `docs/codelead/rebuild/extraction-dispositions.json`, then you run `codelead model` again.

A model whose gate does not pass is written all the same. It is the evidence that you read the findings against.

### What it writes

`codelead model` writes its files under `docs/codelead/rebuild/` of the old repository. It writes the model, the gate's report, two drafts (the journeys and the truth file), the feature map and the schema. It registers them in the repository's CodeLead catalog. It writes nothing anywhere else.

Everything is made in memory first. Files are written only when all of them were made. An extraction that read nothing of the application writes nothing, and a model stored before stays as it was.

### The database's own definition

When you have the database's definition, give it to CodeLead:

- `--schema-dump <file.sql>` reads a T-SQL DDL script, or every `*.sql` file in a folder.
- `--schema-from <key>` reads the catalog of a live copy of the database, read-only. The connection string is kept under `<key>` in a git-ignored `connections.local.config` or `connections.local.json`. It needs the owner's consent: `--consent`.

CodeLead then reconciles the schema read from the code with the definition. Each table and column that the code names is confirmed or contradicted. A column of a table that the definition does not hold is neither: it is "not in the dump" (the verdict `absent`). A contradiction is a warning of the gate, never a failure. No password is printed or written.

> [!TIP]
> **For experienced users:** when several projects build the assembly that a reference names, CodeLead reads the one under `--project-root`. Without it, CodeLead reads the one that a solution lists, then the one the web project references by its project file. The summary says which one, and why.

## Options

- **`--source <REPO>`**

  Required. The old application's repository. CodeLead reads it, and writes only under its `docs/codelead/`.

- **`--project <NAME>`**

  The web project, when the repository holds several: its whole name, or a part that only it has.

- **`--artifact <FOLDER>`**

  The deployment artifact: a folder of the repository that holds the site as deployed (`Web.config`, `Views`, `Global.asax`, no project file). CodeLead reads the web project's views, scripts and configuration from it. Use it when the repository holds several artifacts, or when CodeLead finds none. When there is no web project, the artifact is read alone.

- **`--project-root <FOLDER>`**

  Reads the projects beside the web project (its libraries) only from under this folder of the repository. This is the way to say which of several projects that build one assembly is read.

- **`--schema-dump <PATH>`**

  A T-SQL DDL script of the database, or a folder of them (every `*.sql` under it, in path order). Give it again for more files; they are read in order as one dump.

- **`--schema-from <KEY>`**

  Reconciles with the catalog of a live copy, read-only, through the connection string kept under `<KEY>` in a git-ignored `connections.local.config` or `connections.local.json`.

- **`--consent`**

  The owner of the live copy consents to reading its catalog. `--schema-from` needs it, unless the local configuration records the consent.

- **`--no-write`**

  Extracts and prints the summary, and writes nothing.

- **`--json`**

  Prints the System Model as JSON.

## Examples

- Read the Basic Store sample without writing anything:

  ```console
  codelead model --source . --no-write
  ```

  ```output
  System Model of webapp (basic-store/WebApplication1, inventory: project-file): 289 nodes, 189 edges; 63 entry points, 154 caller records; 10 not resolved, 0 leading nowhere.
  Recipes: aspnet-mvc, jquery-browser. Passes: server, surfaces, client, channel ran.
  Gate: not passed: 36 findings to resolve (36 unexplained-orphan).
  Schema read from the code (schema.md): 4 tables, 28 columns (0 confirmed, 28 inferred), 30 keys, 0 procedures called by name; 4 data accesses not read whole (0 schema-unread warnings).
  ```

- Read it and write the files. The gate does not pass, and the files are written all the same:

  ```console
  codelead model --source .
  ```

  ```output
  System Model of webapp (basic-store/WebApplication1, inventory: project-file): 289 nodes, 189 edges; 63 entry points, 154 caller records; 10 not resolved, 0 leading nowhere.
  …
  Written all the same: the model is the evidence the findings are read against (extraction-gate.md names each, with the key to record a reason under).
  Written: docs/codelead/rebuild/system-model.json
  Written: docs/codelead/rebuild/extraction-gate.md
  Written: docs/codelead/rebuild/journeys.draft.json
  Written: docs/codelead/rebuild/truth.draft.json
  Written: docs/codelead/rebuild/model.md
  Written: docs/codelead/rebuild/schema.json
  Written: docs/codelead/rebuild/schema.md
  Written: docs/codelead/rebuild/feature-map.json
  Written: docs/codelead/rebuild/feature-map.md
  ```

- Reconcile the schema with a DDL script of the database:

  ```console
  codelead model --source . --no-write --schema-dump db/store.sql
  ```

  ```output
  …
  Gate: not passed: 36 findings to resolve (36 unexplained-orphan); 3 warnings.
  Schema read from the code (schema.md): 4 tables, 28 columns (as the code names them: 0 confirmed, 28 inferred), 30 keys, 0 procedures called by name; 4 data accesses not read whole (0 schema-unread warnings).
  Reconciled with the dump db/store.sql (sha256 32b625096b31…): of the 28 columns the code names, 7 confirmed by the dump, 0 contradicted, 21 not in the dump (their tables are not in it); 0 tables only the dump holds; 2 rules the database states; 3 schema-contradicted warnings (schema.md).
  ```

  The `Schema read` line counts what the code says of its own columns. The `Reconciled` line counts what the dump says of them. Here 21 columns belong to tables that the dump does not define, so the dump says nothing of them.

- Name a web project that does not exist. Nothing is extracted, and the exit code is 2:

  ```console
  codelead model --source . --project nothing --no-write
  ```

  ```output
  The System Model could not be extracted: No project named "nothing" under <your project> (found: webapp).
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The gate passed (a schema-only model included). |
| `1` | The gate did not pass. The report names each finding and the key to record a reason under. |
| `2` | Nothing could be extracted or written, the extraction read nothing, or a definition of the database was refused. `--source` missing, or a wrong option, is also `2`. |

## Files

All files are under `docs/codelead/rebuild/` of the old repository.

| File | What it holds |
|---|---|
| `system-model.json` | The System Model. |
| `extraction-gate.md` | The gate's report: each finding and its key. |
| `extraction-dispositions.json` | Read, not written: your reasons for the gate's findings. |
| `journeys.draft.json`, `truth.draft.json` | Drafts of the journeys and of the truth file. |
| `model.md` | The model, for people. |
| `feature-map.json`, `feature-map.md` | The feature map: the features that the rebuild will carry. |
| `schema.json`, `schema.md` | The database as the code reads it, reconciled with a definition when one was given. |

The command also registers the gate report, the map and the schema in `docs/codelead/CATALOG.md` and `catalog.json`.

## See also

- [The System Model](../modernization/system-model.md)
- [`codelead rebuild init`](codelead-rebuild-init.md): the next step, after the model.
- [`codelead modernize run`](codelead-modernize-run.md): the program's `model` stage runs this command.
- [`codelead register`](codelead-register.md) and [`codelead seed`](codelead-seed.md): read the model's files.
- [Rebuild manifest](../reference/rebuild-manifest.md)
- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)
