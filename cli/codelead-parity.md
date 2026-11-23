---
title: codelead parity
description: Compare the pages of a running application before and after a change, or check that a rebuild is finished.
kind: reference
area: cli
date: 2026-11-20
---

# codelead parity

**Applies to:** CodeLead 0.1 and later

`codelead parity` has two jobs. Over HTTP, it captures the pages of a running application as a baseline, then checks the application against it after a change. With `--rebuild`, it is the gate for calling a rebuild finished, read from the rebuild's files.

## Name

`codelead parity` - Page-level parity over HTTP, and the exit gate of a rebuild.

## Synopsis

```console
codelead parity --capture <BASE_URL> [--routes <LIST>] [--bundle <DIR>] [--baseline <DIR>] [--json]
codelead parity --check <BASE_URL> [--baseline <DIR>] [--out <DIR>] [--json]
codelead parity --rebuild [<ROOT>] [--json]
codelead parity -h|--help
```

## Description

*Parity* means that the new version does what the old version did. See [The walk and parity](../modernization/walk-and-parity.md).

### Page parity over HTTP

Use page parity when a change must not change what the pages show, for example a change of the user interface's framework.

1. **Capture** the baseline on the unchanged application: `codelead parity --capture <url>`. CodeLead fetches every GET route, cleans each page of the parts that change at every request, and stores it.
2. Make the change.
3. **Check** the changed application: `codelead parity --check <url>`. CodeLead fetches the same routes again, and compares each page with the baseline.

Each route is `same`, `changed`, `missing`, `new`, `error` or `waived`. The check fails on a changed, missing or errored route that is not waived. Declare the check in the [validation](../concepts/validation-and-the-gate.md) commands of a change, so that the gate runs it.

The routes come from the dossier's structural inventory: the GET actions with no parameters. When the dossier holds none, give them with `--routes`.

> [!NOTE]
> **New to this?** A *baseline* is a copy of how things are before a change. The check compares the new pages with it, line by line.

A difference that you expect is a **waiver**. Write it in `docs/codelead/parity-waivers.json`, with its reason. A waived route passes, and the report shows its reason.

### The exit gate of a rebuild

`codelead parity --rebuild` fetches nothing. It reads the rebuild's files and decides whether the rebuild is finished. It checks:

- every feature of the old application's map against the rebuild's API routes, SPA routes, contract tests and recorded decisions;
- the wiring and the reachability of the new code;
- the last walk: it must be a clean walk of the code as it is now, with the rebuild's own journeys.

It fails while a feature is missing or half landed. It fails while a pin has no contract test that runs, or while a wiring finding is open. It also fails while there is no clean walk of the current code; run [`codelead walk --base <url>`](codelead-walk.md) for that. The report names each blocker, and is written to `docs/codelead/parity/rebuild-parity.md`.

This gate is for the end of a rebuild, not for each part. It fails until the first clean walk.

> [!TIP]
> **For experienced users:** a feature is `landed`, `endpoint only`, `screen only`, `endpoint not read`, `screen not read`, `dropped by decision` or `missing`. "Not read" means that CodeLead could not read the API's inventory or the SPA's route table, so it cannot show that the part landed. A rebuild without a web target has nothing to wire or walk.

The command is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, it is refused before anything runs, with exit code 2. See [Licence and plans](../concepts/licence-and-plans.md).

## Options

- **`--capture <BASE_URL>`**

  Fetches the routes from this base URL, and stores the baseline.

- **`--check <BASE_URL>`**

  Fetches the routes again, and compares them with the baseline. Exit code 1 on a difference that is not waived.

- **`--rebuild [<ROOT>]`**

  Parity for a rebuild, and the gate for calling it finished. `<ROOT>` is the rebuild repository; the default is the current folder. Exit code 1 while a blocker is open. The report goes to `docs/codelead/parity/rebuild-parity.md`.

- **`--baseline <DIR>`**

  The baseline folder. The default is `docs/codelead/parity/baseline`.

- **`--out <DIR>`**

  With `--check`: where the new capture and the report go. The default is `docs/codelead/parity/latest`.

- **`--routes <LIST>`**

  The routes to use, separated by commas, in place of the GET routes of the structural inventory.

- **`--bundle <DIR>`**

  The dossier bundle that holds `functional-structure.json`. The default is `docs/codelead/dossier`.

- **`--json`**

  Prints the report as JSON.

## Examples

The first three examples ran against a small local site with three pages, on port 5791. It takes the place of a running application.

- Capture a baseline of three pages of the running application:

  ```console
  codelead parity --capture http://localhost:5791 --routes /,/Products,/Home/About
  ```

  ```output
  Parity baseline: 3 route(s) captured from http://localhost:5791 into docs/codelead/parity/baseline. Declare `codelead parity --check http://localhost:5791` in the validation gate before the UI changes.
  ```

- Check the application when nothing changed:

  ```console
  codelead parity --check http://localhost:5791
  ```

  ```output
  # Page parity — http://localhost:5791

  Checked 2026-11-20T04:17:18.934Z: 3 route(s) — 3 same, 0 changed, 0 missing, 0 new, 0 error, 0 waived. **PASS** (a changed, missing or errored route fails; a waived one carries its reason).

  | Route | State | Detail |
  |---|---|---|
  | `/` | same |  |
  | `/Products` | same |  |
  | `/Home/About` | same |  |
  ```

- Check it after a change of the About page. The check fails with exit code 1, and says what changed:

  ```console
  codelead parity --check http://localhost:5791
  ```

  ```output
  # Page parity — http://localhost:5791

  Checked 2026-11-20T04:17:48.341Z: 3 route(s) — 2 same, 1 changed, 0 missing, 0 new, 0 error, 0 waived. **FAIL** (a changed, missing or errored route fails; a waived one carries its reason).

  | Route | State | Detail |
  |---|---|---|
  | `/` | same |  |
  | `/Products` | same |  |
  | `/Home/About` | changed | line 1: before "<html><body><h1>About</h1><p>Your application description page.</p></body></html", after "<html><body><h1>About us</h1><p>A new text.</p></body></html>" |
  ```

- Check whether the rebuild of the Basic Store sample is finished. It is not, and the report names each blocker:

  ```console
  codelead parity --rebuild
  ```

  ```output
  # Parity for a rebuild
  …
  **Routes:** 63 features — 0 landed, 0 endpoint only, 0 screen only, 0 endpoint not read, 26 screen not read, 0 dropped by decision, 37 missing.
  **Pins:** 0 features carry characterization tests — 0 with contract tests that run on the new endpoint, 0 without. Engine pins: the source repository has no characterization project.

  **Wiring:** 1 finding to resolve (1 uncalled-endpoint); reachability not checked, the route table could not be read (no React Router route table (<Routes>, createBrowserRouter or useRoutes) was found in the web target).
  **Walk:** none — the rebuild has never been walked (`codelead walk --base <url>`): nothing shows that a new user can do what the old application allowed.
  **Finished:** no — 5 blockers:
  …
  Written: docs/codelead/parity/rebuild-parity.md
  ```

  Under the title, the report starts with `Generated <time> for the rebuild in this repository`. The report is committed with the rebuild, so neither the page nor its JSON record holds a path of this machine.

- Check before any baseline was captured. CodeLead says what to run first:

  ```console
  codelead parity --check http://127.0.0.1:5791/
  ```

  ```output
  No baseline at docs/codelead/parity/baseline: run `codelead parity --capture http://127.0.0.1:5791/` on the unchanged application first.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The capture fetched every route; the check passed; the rebuild is finished. |
| `1` | A route could not be fetched at capture; the check found a difference that is not waived; the rebuild has a blocker. |
| `2` | Nothing to compare or read: no `--capture`, `--check` or `--rebuild`; no routes; no baseline; no rebuild manifest; a folder without the Pro plan. |

## Files

| File | What `parity` does with it |
|---|---|
| `docs/codelead/parity/baseline/` | `--capture` writes one file for each route, and `manifest.json`. `--check` reads them. |
| `docs/codelead/parity/latest/` | `--check` writes the new capture, `parity-report.json` and `parity-report.md`. |
| `docs/codelead/parity-waivers.json` | `--check` reads the waivers. |
| `docs/codelead/dossier/functional-structure.json` | `--capture` reads the routes. |
| `docs/codelead/parity/rebuild-parity.md` | `--rebuild` writes the report. |
| `docs/codelead/rebuild/rebuild.json`, `docs/codelead/parity/walk.json` | `--rebuild` reads the manifest and the last walk. |

## See also

- [The walk and parity](../modernization/walk-and-parity.md)
- [`codelead walk`](codelead-walk.md): the walk that the rebuild gate needs.
- [`codelead rebuild-report`](codelead-rebuild-report.md): the rebuild's progress.
- [The dossier](../modernization/dossier.md): the structural inventory of the routes.
- [Declare validation commands](../how-to/declare-validation-commands.md)
- [Transforms](../modernization/transforms.md)
