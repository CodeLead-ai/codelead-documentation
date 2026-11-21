---
title: codelead scan
description: Find the files that a coding request is likely about, with no model, or run the security exposure scan as a gate.
kind: reference
area: cli
date: 2026-11-20
---

# codelead scan

**Applies to:** CodeLead 0.1 and later

`codelead scan` shows which files of the repository a coding request is likely about, and why. With `--security`, it runs the security exposure scan instead. Neither calls a model.

## Name

`codelead scan` - Shows the repository files that are relevant to a coding request, with no model.

## Synopsis

```console
codelead scan [<REQUEST>...] [--max-files <N>] [--json] [--from-clipboard]
codelead scan --security [--expect-absent <KEY>]... [--out <DIR>] [--json]
codelead scan -h|--help
```

## Description

### The relevance scan

`codelead scan "<request>"` reads the repository and scores each file against the words of the request. It is deterministic: the same repository and the same request give the same list. The list is the context that a [preflight](codelead-preflight.md) sends to the model, so the scan shows you what the model will see.

For each file, the scan prints its role (source, test, project), its language, its score and the reasons. A reason can be a word of the request, a symbol that the file declares, or a relationship to another selected file.

> [!NOTE]
> **New to this?** Run a scan before a change, to check that CodeLead finds the right files. When an important file is missing, name it in the request.

### The security scan

`codelead scan --security` reads the source and the configuration files for known weaknesses. Examples are debug compilation left on, or a write action without `[Authorize]`. Each finding has a class, a file and line, its evidence, a confidence (`proven` or `suspected`), a remediation and the proof that would close it. This is section D15 of the [dossier](../modernization/dossier.md).

With `--expect-absent`, the scan is a gate: it fails when a finding with that key is still present. A security transform declares this gate in its [validation](../concepts/validation-and-the-gate.md) commands, to prove that a weakness is gone.

> [!TIP]
> **For experienced users:** a key for `--expect-absent` has three forms. A class: `debug-configuration`. A class and a file: `debug-configuration:WebApplication1/Web.config`. An id from the previous report: `S-001`. An id is read from `security-exposure.json` in the dossier folder, because an id is stable only inside one report.

## Arguments

- **`<REQUEST>...`**

  The coding request, in words. Without a request and without `--from-clipboard`, CodeLead asks you to type it. The security scan takes no request.

## Options

- **`--max-files <N>`**

  The most relevant files to show. A whole number of at least 1. The default is `context.maxFiles` in the configuration (12).

- **`--json`**

  Prints the result as JSON. For the security scan, the JSON is the whole report with `expectAbsent` and `present`.

- **`--from-clipboard`**

  Uses the text on the clipboard as the request.

- **`--security`**

  Runs the security exposure scan (dossier D15) on the repository, in place of the relevance scan.

- **`--expect-absent <KEY>`**

  With `--security`: fails when a finding with this class, `class:file` or id is present. Give it again for more keys.

- **`--out <DIR>`**

  With `--security`: the dossier bundle folder, where the scan reads the dependency report (D3) and the readiness inventory (D13). The default is `docs/codelead/dossier`.

## Examples

- Find the files that a request is about, on the Basic Store sample:

  ```console
  codelead scan "add a discount field to products"
  ```

  ```output
  CodeLead Scan

  Request:
  add a discount field to products

  Repository Context
  - Project type: existing
  - Languages: C#, JavaScript
  - Tools: .NET
  - Validation: dotnet build
  - Files scanned: 89
  - Relevant files shown: 12
  …
  Likely Relevant Files

  1. WebApplication1/Controllers/ProductsController.cs
     Role: source (C#)
     Score: 1000
     Original score: 14
     Reason: lexical match; symbol/token match for product, field
     Reason types: lexical-match, symbol-match, type-declaration
  2. WebApplication1/Models/OrderViewModels.cs
  …
  4. WebApplication1/DAL/StoreContext.cs
     Role: source (C#)
     Score: 860
     Original score: 3
     Reason: lexical match; symbol/token match for product; related source file for selected context
     Reason types: lexical-match, related-source, relationship, symbol-match
  …
  ```

- Show only the three best files, as JSON:

  ```console
  codelead scan "add a discount field to products" --max-files 3 --json
  ```

  ```output
  {
    "request": "add a discount field to products",
    "scannedFileCount": 89,
    "projectProfile": {
      "projectType": "existing",
      "isGreenfield": false,
      "detectedLanguages": [
        {
          "name": "C#",
          "confidence": 0.95,
          "evidence": [
            "Found 26 .cs files",
            "Found WebApplication1/webapp.csproj"
          ]
        },
  …
  ```

- Run the security scan:

  ```console
  codelead scan --security
  ```

  ```output
  Security exposure: 10 finding(s) — 1 proven, 9 suspected — over 75 source and 2 configuration files.
    unauthorized-write-action: 9
    debug-configuration: 1
    note: Package advisories not included: the D3 dependency report (dependencies.json) was not available to this run.
    note: End-of-life runtimes not included: the D13 inventory (modernization-readiness.json) was not available to this run.
  ```

- Use the security scan as a gate. The finding is still there, so the gate fails with exit code 1:

  ```console
  codelead scan --security --expect-absent debug-configuration
  ```

  ```output
  Security exposure: 10 finding(s) — 1 proven, 9 suspected — over 75 source and 2 configuration files.
  …
  Expected absent but still present: debug-configuration — FAIL.
  ```

  A key that has no finding passes with exit code 0. This is what you see after a fix:

  ```console
  codelead scan --security --expect-absent sql-injection
  ```

  ```output
  …
  Expected absent: 1 key(s), none present — pass.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The scan ran. With `--expect-absent`, no expected-absent key is present. |
| `1` | With `--expect-absent`, a key is still present. Also an error: an option that is not valid, a repository that cannot be read. |

## Files

`codelead scan` writes nothing.

- The relevance scan reads the repository's files and `.codelead/config.json` (the `context` settings).
- The security scan reads the source and the configuration files. From the dossier folder it reads `dependencies.json`, `modernization-readiness.json` and the previous `security-exposure.json`, when they are there.

## See also

- [`codelead preflight`](codelead-preflight.md): the scan, then the model's analysis.
- [`codelead library`](codelead-library.md): the index that the scan scores.
- [`/scan`](../chat-commands/scan.md): the same scan inside a session.
- [The dossier](../modernization/dossier.md): the security section D15.
- [Transforms](../modernization/transforms.md): the security transform that uses the gate.
- [Validation and the gate](../concepts/validation-and-the-gate.md)
