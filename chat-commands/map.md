---
title: /map
description: Read the repository into its map, with no model call, and know when CodeLead reads it again.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /map

**Applies to:** CodeLead 0.1 and later

## Name

`/map` - Reads the repository into its map: the stack, the validation commands, the modules.

## Synopsis

```console
/map
/map --refresh
```

## Description

The repository map is a short description of the repository that CodeLead keeps in two files: `docs/codelead/repository-map.json` for programs and `docs/codelead/repository-map.md` for people. It names the stack, the entry points, the modules and the validation commands. A stamp says which commit it was read from. See [The repository map](../concepts/repository-map.md).

`/map` writes the map when there is none, or when the stamp is old. A map that is fresh is used as it is. `--refresh` reads the repository again in every case. `/map` calls no model.

A map that is written also seeds the encyclopedia (the knowledge that CodeLead keeps about the project) and writes one evidence record. See [The knowledge base](../concepts/knowledge-base.md).

You rarely need `/map`: **Initialize** in the extension and [`codelead init`](../cli/codelead-init.md) write the map, and a build that finishes writes it again. Use `/map --refresh` after a large change that you made by hand.

> [!NOTE]
> **New to this?** The map is not a copy of your code. It is a summary: what kind of project this is, where it starts, and how to build and test it.

## Options

- **`--refresh`**

  Reads the repository again and writes the map, even when the map is fresh.

`/map refresh` without dashes is refused with `Use: /map --refresh`. Any other word after `/map` is refused too.

## What you see

| Outcome | Headline |
|---|---|
| `done` | `Repository map written: 1 module, 89 files; docs/codelead/repository-map.md` |
| `done` | `Repository map is fresh: 1 module, 89 files; docs/codelead/repository-map.md` |
| `done` | `Repository map read again (--refresh): 1 module, 89 files; docs/codelead/repository-map.md` |
| `failed` | `The repository map could not be written`, with the reason. The way on is `Try again: /map --refresh` and `Check the folder is readable`. |

## Examples

- Write the map of the Basic Store sample for the first time:

  ```console
  /map
  ```

  ```output
  Repository map written: 1 module, 89 files; docs/codelead/repository-map.md
  Entry points: WebApplication1/webapp.csproj
  Stamp: git bd30c0ac3c5f (uncommitted changes), 89 files scanned, written 2026-11-20T04:29:40.312Z
  Encyclopedia: 4 entries from the map (4 new, 0 updated, 0 downgraded); evidence record ev-20261120042940-1uggx4.
  ```

- Run it again immediately. The map is fresh, so CodeLead uses it as it is:

  ```console
  /map
  ```

  ```output
  Repository map is fresh: 1 module, 89 files; docs/codelead/repository-map.md
  Entry points: WebApplication1/webapp.csproj
  Stamp: git bd30c0ac3c5f (uncommitted changes), 89 files scanned, written 2026-11-20T04:29:40.312Z
  ```

- Read it again on purpose:

  ```console
  /map --refresh
  ```

  ```output
  Repository map read again (--refresh): 1 module, 89 files; docs/codelead/repository-map.md
  Entry points: WebApplication1/webapp.csproj
  Stamp: git bd30c0ac3c5f (uncommitted changes), 89 files scanned, written 2026-11-20T04:29:40.997Z
  Encyclopedia: 4 entries from the map (0 new, 4 updated, 0 downgraded); evidence record ev-20261120042941-dchvdd.
  ```

- The form without dashes is refused:

  ```console
  /map refresh
  ```

  ```output
  Unsupported command form: /map refresh

  Use:
    /map --refresh
  ```

## Files

- Writes `docs/codelead/repository-map.json` and `docs/codelead/repository-map.md`.
- When it writes the map, it also updates the encyclopedia in `.codelead/knowledge.json` and writes an evidence record to `.codelead/evidence/<id>.json`.

## See also

- [`/scan`](scan.md)
- [The repository map](../concepts/repository-map.md)
- [The knowledge base](../concepts/knowledge-base.md)
- [`codelead map`](../cli/codelead-map.md)
- [`codelead init`](../cli/codelead-init.md)
