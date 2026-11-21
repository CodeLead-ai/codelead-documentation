---
title: /scan
description: Find the files that a request is likely about, with no model call, and see why each file was chosen.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /scan

**Applies to:** CodeLead 0.1 and later

## Name

`/scan` - Finds the files that a request is likely about, with no model call.

## Synopsis

```console
/scan <REQUEST>
/scan --from-clipboard
```

## Description

`/scan` reads the repository and ranks its files for a request. It matches the words of the request with file names, symbols and types, then adds the files that are related to the best matches. For each file it says why it was chosen. It calls no model.

You use `/scan` to see what CodeLead would read for a request, before [`/preflight`](preflight.md) or [`/implement`](implement.md) sends anything to a model. When the list misses the file that you mean, name that file in the request.

`/scan` never changes a file.

> [!TIP]
> **For experienced users:** the scan selects at most `context.maxFiles` files (the default is 12) and marks the files that a relationship added. The [repository map](../concepts/repository-map.md) gives the project type and the validation commands that the scan prints. The CLI command [`codelead scan`](../cli/codelead-scan.md) also runs the security scan.

## Arguments

- **`<REQUEST>`**

  The request, in plain words.

## Options

- **`--from-clipboard`**

  Takes the request from the clipboard text.

## What you see

| Outcome | Headline | The way on |
|---|---|---|
| `done` | `Scan complete.` | None. |
| `stopped` | `Nothing scanned: /scan needs a request`, after `Usage: /scan <request>` | `/scan <request>`, `/scan --from-clipboard` |
| `failed` | `Failed to run scan: <the error>` | Run it again after you fix the cause. |

## Examples

- Scan the Basic Store sample for a request about the order total:

  ```console
  /scan add a discount to the order total
  ```

  ```output
  Scan Results
  Finding likely relevant files without calling a model.
  CodeLead Scan

  Request:
  add a discount to the order total

  Repository Context
  - Project type: existing
  - Languages: C#, JavaScript
  - Tools: .NET
  - Validation: dotnet build
  - Files scanned: 89
  - Relevant files shown: 12

  Context Pack
  - Selected 12 files: 12 source, 0 test, 0 project/config anchors, 3 relationship-linked.
  …

  Likely Relevant Files

  1. WebApplication1/Models/OrderViewModels.cs
     Role: source (C#)
     Score: 1000
     Original score: 14
     Reason: lexical match; symbol/token match for order, total
     Reason types: lexical-match, symbol-match, type-declaration
  2. WebApplication1/Controllers/OrdersController.cs
     Role: source (C#)
     Score: 1000
     Original score: 11
     Reason: lexical match; symbol/token match for order
     Reason types: lexical-match, symbol-match, type-declaration
  …
  4. WebApplication1/DAL/StoreContext.cs
     Role: source (C#)
     Score: 860
     Original score: 3
     Reason: lexical match; symbol/token match for order; related source file for selected context
     Reason types: lexical-match, related-source, relationship, symbol-match
  …
  Scan complete.
  ```

  The file `StoreContext.cs` matched only one word, but it is related to the controller, so the scan added it.

- Scan without a request. The command stops, as `/preflight` does without a request:

  ```console
  /scan
  ```

  ```output
  Usage: /scan <request>
  ■ Nothing scanned: /scan needs a request; next: /scan --from-clipboard
  ```

- Scan the text that you copied, for example an error message:

  ```console
  /scan --from-clipboard
  ```

## Files

- Reads the repository's files and the ignore rules (`.gitignore`).
- Keeps the scan in the session (`/status` shows `Last scan result: yes`). It writes no file of the project.

## See also

- [`/preflight`](preflight.md)
- [`/lint`](lint.md)
- [`/map`](map.md)
- [The repository map](../concepts/repository-map.md)
- [`codelead scan`](../cli/codelead-scan.md)
