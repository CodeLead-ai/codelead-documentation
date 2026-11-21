---
title: codelead rebuild-report
description: Write a rebuild's progress report as one HTML page, from its files alone, and print the path of the page.
kind: reference
area: cli
date: 2026-11-20
---

# codelead rebuild-report

**Applies to:** CodeLead 0.1 and later

`codelead rebuild-report` writes the modernization progress report of a rebuild as one HTML page. It prints the path of the page. It calls no model and starts no server.

## Name

`codelead rebuild-report` - Renders a rebuild's progress report and prints the path written.

## Synopsis

```console
codelead rebuild-report [--root <REBUILD>] [--out <FILE>]
codelead rebuild-report -h|--help
```

## Description

The report shows where a [rebuild](../modernization/rebuild.md) stands:

- the plan's parts and their states (planned, accepted, parked, dropped);
- the walk over time: how many journey steps passed, failed or were blocked at each walk;
- every [evidence record](../concepts/evidence-records.md), grouped by landing chain (a change and its repairs);
- what fails now.

CodeLead reads it all from the rebuild's files: the manifest, the plan, the evidence records, the walk reports and the journeys. Open the page in a browser.

By default the page is `.codelead/rebuild-report.html` in the rebuild. An interactive session in the rebuild refreshes the same file. With `--out`, the page is written to the path that you give, and nothing is written under the root.

> [!NOTE]
> **For team leads:** the report is one file with no outside resources. You can attach it to a mail or put it in a review, and it opens without CodeLead.

> [!TIP]
> **For experienced users:** use `--out` to write the report outside the repository, for example into a folder that a build server publishes.

## Options

- **`--root <REBUILD>`**

  The rebuild repository. The default is the current folder.

- **`--out <FILE>`**

  Writes the page to this path, in place of `<root>/.codelead/rebuild-report.html`. Nothing is then written under the root.

## Examples

- Write the report of the rebuild in the current folder:

  ```console
  codelead rebuild-report
  ```

  ```output
  <your project>/.codelead/rebuild-report.html
  ```

- Write the report of a rebuild in another folder, to a path outside it:

  ```console
  codelead rebuild-report --root ../basic-store-rebuild --out ../reports/report.html
  ```

  ```output
  <the folder above>/reports/report.html
  ```

- Write the report to a file of your choice, from inside the rebuild:

  ```console
  codelead rebuild-report --out ../report.html
  ```

  ```output
  <the folder above>/report.html
  ```

- Run it in a folder that is not a rebuild. CodeLead writes nothing, and the exit code is 2:

  ```console
  codelead rebuild-report
  ```

  ```output
  Refused: this folder is not a rebuild (no docs/codelead/rebuild/rebuild.json), so there is no report to write. Run it in the rebuild's repository, or name it with --root <rebuild>; codelead rebuild init makes one.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The page was written. Its path is printed. |
| `1` | The page was not written. The line says why. |
| `2` | Refused before anything was written: the folder is not a rebuild, because it has no `docs/codelead/rebuild/rebuild.json`. |

## Files

| File | What `rebuild-report` does with it |
|---|---|
| `docs/codelead/rebuild/rebuild.json`, `rebuild-plan.json` | Reads the manifest and the plan. |
| `.codelead/evidence/ev-*.json` | Reads the evidence records. |
| `docs/codelead/parity/walk.json` and the stored walks | Reads the walks. |
| `docs/codelead/rebuild/journeys.json` | Reads the journeys. |
| `.codelead/rebuild-report.html`, or the `--out` path | Writes the page. |

## See also

- [The rebuild](../modernization/rebuild.md)
- [`codelead rebuild run`](codelead-rebuild-run.md): the run that the report follows.
- [`codelead walk`](codelead-walk.md): the walks that the report shows over time.
- [`codelead parity`](codelead-parity.md): the gate for calling a rebuild finished.
- [Evidence records](../concepts/evidence-records.md)
- [The audit pack](../modernization/audit-pack.md)
