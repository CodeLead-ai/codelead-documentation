---
title: The project library
description: Learn what the project library is, the index of files, modules, symbols and relationships that CodeLead builds from the code, and what uses it.
kind: concept
area: concepts
date: 2026-11-20
---

# The project library

The project library is the index that CodeLead builds from a repository's code. It holds the files, the modules, the symbols that the files declare, and the relationships between the files. CodeLead builds it with no model call. The same code always gives the same library. CodeLead uses it to find the files that a request is about, and to see which files must change together.

**In short**
- The library lists each file with its language and its role (such as source, test or project), the modules, the symbols and the relationships.
- CodeLead builds it again for each request. The [repository map](repository-map.md) is its kept summary.
- The scan (the search for the files of a request), the preflight and the change surface (the files that must change together) read it.
- [`codelead library`](../cli/codelead-library.md) prints it, so that you can see why a file was or was not chosen.

## Why it exists

A model reads only what CodeLead gives it. When CodeLead gives the wrong files, the model writes a wrong patch, or a patch that misses a file that had to change too. The library is how CodeLead chooses with reasons, not with guesses. A symbol, such as a class name in your request, leads to the file that declares it. A relationship, such as a `using` line, leads to the files that depend on it.

## How it works

CodeLead reads the files of the repository and follows its ignore rules. For each file, it detects the language and the role, and it reads the symbols that the file declares: classes, interfaces, functions. Then it links the files: one file imports another, uses it, references its project, or tests it. Each link has a confidence and a reason.

A real library of the Basic Store sample, trimmed:

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
…
Relationships:
- WebApplication1/App_Start/IdentityConfig.cs -> WebApplication1/Models/AccountViewModels.cs
  Type: imports
  Confidence: 0.8
  Reason: Import/using "WebApplication1.Models" matches symbol models in webapplication1
…
```

### What reads it

- **The scan**. [`codelead scan`](../cli/codelead-scan.md) and [`/scan`](../chat-commands/scan.md) score each file against the words and the symbols of a request. They show the reasons: a word match, a symbol match, a related file.
- **The preflight**. The files that the [preflight](requests-plans-and-patches.md) shows to the model come from the same scoring.
- **The change surface**. When a patch changes a file, the change surface says which other files it must change too. An example is a class that implements a changed interface. The [patch check](the-patch-check.md) writes a note when such a file is missing, or when a patch changes a file outside the surface of the request.
- **The repository map**. The map keeps a summary of the library and its relationships. When the files have not changed, CodeLead takes the relationships from the map and saves time.

> [!NOTE]
> **New to this?** A *symbol* is a name that the code declares, such as the class `OrdersController`. A *relationship* says that one file depends on another, such as through a `using` line.

## What it is not

- **Not a compiler**. The library reads the code as text, with patterns for each language. It can miss a link that only a compiler sees, such as one made at run time.
- **Not kept as a file**. CodeLead builds it again for each request. Only the repository map keeps a summary.
- **Not the repair's file choice**. A [repair round](repair.md) reads the names in the failure. It finds the files that declare, implement or build those names. It does not rely on the library's scores.

## Details

- **Contents**. `rootPath`, `projectProfile` (the type, the languages, the tools), `files`, `modules`, `symbols` and `relationships`. `codelead library --json` prints all of it. The text output shows the first 20 symbols.
- **File roles**. `source`, `test`, `config`, `docs`, `project`, `build`, `data` and `unknown`.
- **Relationship types**. `imports`, `uses`, `project-reference`, `test-target`, `same-name`, `same-directory` and `mentions`, each with `confidence` (0 to 1) and `reason`.
- **Type edges**. The change surface also reads which types implement or derive from which, from the declarations. Those edges are its own, not relationships of the library.
- **Ignored files**. Dependency, build, cache and session folders, and sensitive files such as `.env`, private keys and database files, are never indexed.
- **No model call, no file written**. `codelead library` writes nothing.

> [!TIP]
> **For experienced users:** when a scan misses a file that you expected, run `codelead library --json` and search for its symbols. A file with no symbol and no relationship can only be found by its words. Name it in the request.

## Related content

- [`codelead library`](../cli/codelead-library.md)
- [`codelead scan`](../cli/codelead-scan.md) and [`/scan`](../chat-commands/scan.md)
- [`codelead map`](../cli/codelead-map.md)
- [The repository map](repository-map.md)
- [Requests, plans and patches](requests-plans-and-patches.md)
- [The patch check](the-patch-check.md)
- [Glossary](glossary.md)
