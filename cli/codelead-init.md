---
title: codelead init
description: Create the CodeLead folder in a project and read the repository into its map.
kind: reference
area: cli
date: 2026-11-20
---

# codelead init

**Applies to:** CodeLead 0.1 and later

`codelead init` prepares a folder for CodeLead. Run it once in each project, before the first session.

## Name

`codelead init` - Creates the local CodeLead configuration folder and reads the repository into its map.

## Synopsis

```console
codelead init [--force] [--no-map]
codelead init -h|--help
```

## Description

`codelead init` does three things in the current folder:

1. It writes the configuration file `.codelead/config.json` with the default settings. See [Configuration file](../reference/configuration.md).
2. It makes sure that git ignores `.env`, `.env.local` and `.env.*.local`. CodeLead reads your keys from `.env`, and the keys must never be committed. When there is no `.gitignore`, CodeLead creates one.
3. It reads the repository into its [repository map](../concepts/repository-map.md) (`docs/codelead/repository-map.md` and `.json`), the same way as [`codelead map`](codelead-map.md). Then it seeds the [knowledge base](../concepts/knowledge-base.md) from the map and writes one [evidence record](../concepts/evidence-records.md) of the read.

`codelead init` calls no model and uses no network.

> [!NOTE]
> **New to this?** CodeLead keeps its settings in a folder named `.codelead` inside your project. `codelead init` creates that folder. The extension's **Initialize** button runs the same command.

### An empty folder

In an empty folder there is nothing to read. CodeLead writes the configuration and says that the map is written later, when the first build finishes. This is the usual case for a new application.

### A folder that already has a configuration

`codelead init` never replaces an existing `.codelead/config.json`. It stops with exit code 1 and tells you to add `--force`. With `--force`, the file is written again with the defaults, and your changes to it are lost.

### When the map cannot be made

A map that cannot be made never stops `init`. CodeLead prints one warning line, and the configuration stays. Run [`codelead map`](codelead-map.md) later to try again.

> [!TIP]
> **For experienced users:** the knowledge base and the evidence records go to `.codelead/` by default. `CODELEAD_KNOWLEDGE_DIR` moves them. See [Environment variables](../reference/environment-variables.md).

## Options

- **`--force`**

  Writes `.codelead/config.json` again even when it exists. The file gets the default settings. Your changes to it are lost.

- **`--no-map`**

  Does not read the repository into its map, and does not seed the knowledge base. Only the configuration and the `.gitignore` lines are written.

## Examples

- Prepare an empty folder for a new application:

  ```console
  mkdir task-manager
  cd task-manager
  codelead init
  ```

  ```output
  ✔ Created <your project>/.codelead/config.json
  <your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
  Repository map: nothing to read yet (no source files); it is written when the first build finishes.
  ```

- Prepare an existing repository, the Basic Store sample:

  ```console
  codelead init
  ```

  ```output
  ✔ Created <your project>/.codelead/config.json
  <your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
  Repository map written: 1 module, 89 files; docs/codelead/repository-map.md
  Encyclopedia: 4 entries from the map (4 new, 0 updated, 0 downgraded); evidence record ev-20261120041050-b1slza.
  ```

- Run it again in the same folder. CodeLead refuses and says what to do next:

  ```console
  codelead init
  ```

  ```output
  Error:
  Config already exists at <your project>/.codelead/config.json.
  Run codelead init --force to overwrite it.
  ```

- Write the default configuration again, and do not read the repository:

  ```console
  codelead init --force --no-map
  ```

  ```output
  ✔ Created <your project>/.codelead/config.json
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The configuration was written. A map that could not be made is a warning, not a failure. |
| `1` | The configuration exists and `--force` was not given, or the file could not be written. |

## Files

| File | What `init` does with it |
|---|---|
| `.codelead/config.json` | Writes it. With `--force`, writes it again. |
| `.gitignore` | Adds `.env`, `.env.local` and `.env.*.local` when they are not there. Creates the file when there is none. |
| `docs/codelead/repository-map.md`, `docs/codelead/repository-map.json` | Writes the map, unless `--no-map` is given or the folder is empty. |
| `docs/codelead/CATALOG.md`, `docs/codelead/catalog.json` | Registers the map in the project's catalog of documents. |
| `.codelead/knowledge.json` | Adds the entries that the map seeds. |
| `.codelead/evidence/ev-*.json` | Writes one evidence record of the read. |

See [Files and folders](../reference/files-and-folders.md) for every file.

## See also

- [`codelead doctor`](codelead-doctor.md): check the machine before the first session.
- [`codelead map`](codelead-map.md): read the repository again.
- [`codelead config`](codelead-config.md): see the configuration that `init` wrote.
- [The repository map](../concepts/repository-map.md)
- [Quickstart: build a new app](../get-started/quickstart-new-app.md)
- [The Projects view](../extension/projects-view.md): the **Initialize** button.
