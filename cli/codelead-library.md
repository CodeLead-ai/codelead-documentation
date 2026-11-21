---
title: codelead library
description: Print the project library of the current repository, the index of files, modules, symbols and relationships that CodeLead builds from the code.
kind: reference
area: cli
date: 2026-11-20
---

# codelead library

**Applies to:** CodeLead 0.1 and later

`codelead library` prints the project library of the current repository. The project library is the index that CodeLead builds from the code: the files, the modules, the symbols and the relationships between files.

## Name

`codelead library` - Inspects the deterministic project library of the current repository.

## Synopsis

```console
codelead library [--json]
codelead library -h|--help
```

## Description

Before CodeLead chooses the files for a request, it indexes the repository. The index is the **project library**. It is deterministic: the same code gives the same library. No model is called.

The library holds:

- **Inventory**: how many files, modules, relationships and symbols.
- **Languages** and **Roles**: the files of each language, and of each role (source, test, data, project).
- **Modules**: the folders that group the files, with a summary.
- **Symbols**: the classes, interfaces and functions that the files declare, with their files.
- **Relationships**: how one file is linked to another, with a confidence and a reason. The types are `imports`, `uses`, `project-reference`, `test-target`, `same-name`, `same-directory` and `mentions`.

The [scan](codelead-scan.md) scores these files against a request, and the [repository map](codelead-map.md) is a summary of the library. Use `codelead library` when a scan misses a file and you want to know why.

> [!NOTE]
> **New to this?** A *symbol* is a name that the code declares, for example the class `ProductsController`. A *relationship* says that one file depends on another, for example through a `using` line.

> [!TIP]
> **For experienced users:** `--json` prints the whole library, with `rootPath`, `projectProfile`, `files`, `modules`, `symbols` and `relationships`. The text output shows the first 20 symbols and a count of the rest.

## Options

- **`--json`**

  Prints the library as JSON. The default is a summary in words.

## Examples

- Print the library of the Basic Store sample:

  ```console
  codelead library
  ```

  ```output
  Project Library

  Root:
  <your project>

  Project:
  - Type: existing
  - Summary: Detected an existing project using C#, JavaScript; tools include .NET.

  Inventory:
  - Files indexed: 89
  - Modules: 1
  - Relationships: 299
  - Symbols: 244

  Languages:
  - C#: 26 files
  - CSS: 7 files
  - JavaScript: 5 files
  - JSON: 2 files
  - XML: 1 file
  …
  ```

- Read the symbols and the relationships, further down the same output:

  ```output
  Notable Symbols:
  - BundleConfig - class - WebApplication1/App_Start/BundleConfig.cs
  - FilterConfig - class - WebApplication1/App_Start/FilterConfig.cs
  …
  - ProductsController - class - WebApplication1/Controllers/ProductsController.cs
  - StoreContext - class - WebApplication1/DAL/StoreContext.cs
  …
  ...and 224 more symbols.

  Relationships:
  - WebApplication1/App_Start/IdentityConfig.cs -> WebApplication1/Models/AccountViewModels.cs
    Type: imports
    Confidence: 0.8
    Reason: Import/using "WebApplication1.Models" matches symbol models in webapplication1
  …
  ```

- Print the library as JSON, for a script:

  ```console
  codelead library --json
  ```

  ```output
  {
    "rootPath": "<your project>",
    "projectProfile": {
      "rootPath": "<your project>",
      "projectType": "existing",
      "isGreenfield": false,
      "detectedLanguages": [
        {
          "name": "C#",
          "confidence": 0.95,
  …
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The library was printed. |
| `1` | The repository could not be indexed. |

## Files

`codelead library` writes nothing. It reads the files of the repository, and follows the repository's ignore rules.

## See also

- [`codelead scan`](codelead-scan.md): the files that a request is about.
- [`codelead map`](codelead-map.md): the summary that CodeLead keeps.
- [The repository map](../concepts/repository-map.md)
- [How CodeLead works](../concepts/how-codelead-works.md)
