---
title: codelead spec
description: Write specs in plain words, check them, draft them into journeys and walk them on the running application.
kind: reference
area: cli
date: 2026-11-20
---

# codelead spec

**Applies to:** CodeLead 0.1 and later

`codelead spec` works with **specs in plain words**: short files that say what a person does in the application, and what they then see. CodeLead checks a spec, drafts it into a journey, and walks it on the running application.

## Name

`codelead spec` - Specs in plain words: check, new, draft, walk, coverage and report.

## Synopsis

```console
codelead spec new <NAME>
codelead spec check [<PATHS>...] [--json]
codelead spec draft [<PATHS>...]
codelead spec walk [<PATHS>...] --base <URL> [--api <ORIGIN>] [--time-zone <ZONE>]
                   [--no-write] [--headed] [--screenshots] [--json]
codelead spec coverage [<PATHS>...] --base <URL> [--out <DIR>] [--json]
codelead spec coverage [<PATHS>...] --sitemap <FILE> --forms <FILE> [--json]
codelead spec report [--json]
codelead spec -h|--help
```

## Description

A spec is a markdown file under `docs/codelead/specs/<name>.md`. It has a title (what the person gets done), a `Who:` line, numbered **Steps**, and other ways it can go under **Also**. Each step says what the person does, then what they see: `I see "…"`, `I do not see "…"`, `I see a "…" link`, or `I am on /a-page`.

Here is the example that `spec new` writes, without its help lines:

```markdown
# Register and sign in

Who: a new user

## Steps

1. Open the first screen and choose "Register". I see "Create your account".
2. Fill in a new name and a new password, then choose "Create account". I see "Welcome".
3. Sign out, then sign in with the same name and password. I see "Signed in as" and my name.

## Also

- An empty form: open the first screen, choose "Register", then choose "Create account" without filling anything in. I see "Enter a name" and I do not see "Welcome".
```

> [!NOTE]
> **New to this?** A spec is a test that a tester or a product owner can write, with no code. The walk follows it on the running application, the way a person would.

The chat command [`/spec`](../chat-commands/spec.md) is something else: it shows a new application's project specification.

### The subcommands

| Subcommand | What it does | Model calls |
|---|---|---|
| `new <name>` | Writes `docs/codelead/specs/<name>.md` from the template: a filled example with a line of help under each part. Never over a file that is there. | None |
| `check [paths...]` | Checks each spec before it is drafted. A **gap** means that the spec cannot be used until it is fixed. A **note** means that the walk could judge more. Each one says how to fix it, with a line to write. | None |
| `draft [paths...]` | Drafts each spec into a journey that the walk runs, after the check passes: `<name>.journeys.json` and `<name>.journeys.md` beside it. | One per spec |
| `walk [paths...]` | Walks each drafted spec on the running application and reports every step as pass, fail or blocked: `<name>.walk.json` and `<name>.walk.md` beside it. | None |
| `coverage [paths...]` | Explores the running application with the [Analyst](../analyst/explore-an-application.md) (observe only: forms are never sent). It says which screens and forms the specs never name, and what they name that is not there. | None |
| `report` | Writes the spec library: `docs/codelead/specs/README.md` and the tester's page `.codelead/spec-report.html`. | None |

Without paths, each subcommand takes every spec in `docs/codelead/specs`.

> [!TIP]
> **For experienced users:** `draft` uses the architect role. When the privacy settings block its remote provider, the draft is refused with exit code 2. In a session, [`/implement --spec <file>`](../chat-commands/implement.md) walks a spec after the landing.

## Arguments

- **`<NAME>`**

  For `new`: the spec's name, in lower-case letters, digits and hyphens, for example `export-invoice`.

- **`<PATHS>...`**

  For `check`, `draft`, `walk` and `coverage`: spec files or folders. The default is `docs/codelead/specs`.

## Options

`codelead spec` itself takes only `-h|--help`. The subcommands take these options.

- **`--json`** (`check`, `walk`, `coverage`, `report`)

  Prints the result as JSON.

- **`--base <URL>`** (`walk`, `coverage`)

  The application's base URL, served **without** any development identity. For `coverage`, explore a copy: the explorer clicks the buttons outside forms.

- **`--api <ORIGIN>`** (`walk`)

  The API's origin, when it is on another host than the base.

- **`--time-zone <ZONE>`** (`walk`)

  The server's time zone (an IANA name such as `Europe/Berlin`, or `UTC`), which the walk reads the server's day in.

- **`--no-write`** (`walk`)

  Prints the reports without writing `<name>.walk.json` and `<name>.walk.md`.

- **`--headed`** (`walk`)

  Shows the browser.

- **`--screenshots`** (`walk`)

  Saves a screenshot after each step, in `<name>.screenshots` beside the report.

- **`--sitemap <FILE>`** (`coverage`)

  An exploration's `sitemap.json`, read in place of exploring. Give `--forms` with it.

- **`--forms <FILE>`** (`coverage`)

  An exploration's `forms.json`, read in place of exploring. Give `--sitemap` with it.

- **`--out <DIR>`** (`coverage`)

  Where the explorer writes its files. The default is a new scratch folder.

## Examples

- Write the first spec from the template:

  ```console
  codelead spec new add-product
  ```

  ```output
  Written: docs/codelead/specs/add-product.md
  Edit it, then: codelead spec check docs/codelead/specs/add-product.md
  ```

- Check every spec. One spec is usable. The other has a gap, and the check says how to fix it:

  ```console
  codelead spec check
  ```

  ```output
  docs/codelead/specs/add-product.md: usable (0 notes)
  docs/codelead/specs/change-price.md:7: note: step 1 says nothing the walk can check. Fix: add what the person sees after it, so the walk can tell the step worked. Example: 1. Open the first screen and choose "Products". I see "…".
  docs/codelead/specs/change-price.md:8: step 2, the last step, says nothing the walk can check. Fix: end it with what the person sees: I see "…", I do not see "…", I see a "…" link, or I am on /path. Example: 2. Choose "Edit" beside "Product 1", fill in 12.50 as the price, then choose "Save". I see "…".
  docs/codelead/specs/change-price.md: 1 gap, 1 note; fix the gap, then draft it
  ```

- Draft a spec that has a gap. The draft is refused before any model call:

  ```console
  codelead spec draft docs/codelead/specs/change-price.md
  ```

  ```output
  docs/codelead/specs/change-price.md:8: step 2, the last step, says nothing the walk can check. Fix: end it with what the person sees: I see "…", I do not see "…", I see a "…" link, or I am on /path. Example: 2. Choose "Edit" beside "Product 1", fill in 12.50 as the price, then choose "Save". I see "…".
  docs/codelead/specs/change-price.md: not drafted: the spec has a gap the walk cannot judge past. Fix it, then: codelead spec check docs/codelead/specs/change-price.md
  ```

- Walk the specs before they are drafted. Nothing is walked, and each spec says what to do:

  ```console
  codelead spec walk --base http://localhost:5791
  ```

  ```output
  docs/codelead/specs/add-product.md: draft it first (codelead spec draft docs/codelead/specs/add-product.md)
  docs/codelead/specs/change-price.md: draft it first (codelead spec draft docs/codelead/specs/change-price.md)

  Walked 0 of 2 specs; 2 to draft first.
  ```

  Nothing was walked, so the spec library stays as it was.

- Write the spec library:

  ```console
  codelead spec report
  ```

  ```output
  Written: docs/codelead/specs/README.md and .codelead/spec-report.html
  Spec library: 2 specs, 1 usable, 0 walked, coverage: not checked yet.
  ```

  The table in `docs/codelead/specs/README.md`:

  ```output
  | Spec | Title | Who | Steps | Check | Last walk | When | Report |
  | --- | --- | --- | --- | --- | --- | --- | --- |
  | [add-product](add-product.md) | Register and sign in | a new user | 3 + 1 variation | usable | never walked |  |  |
  | [change-price](change-price.md) | Change a product price | a clerk | 2 | 1 gap | never walked |  |  |
  ```

- Draft a usable spec, then walk it with screenshots. The draft makes one model call:

  ```console
  codelead spec draft docs/codelead/specs/add-product.md
  codelead spec walk docs/codelead/specs/add-product.md --base http://localhost:5173 --screenshots
  ```

  <!-- described, not captured -->
  ```output
  docs/codelead/specs/add-product.md: drafted: docs/codelead/specs/add-product.journeys.json and .journeys.md
  …
  PASS    … · Open the first screen and choose "Register": …
  …
  Walked 1 of 1 spec …
  ```

## Exit codes

| Subcommand | `0` | `1` | `2` |
|---|---|---|---|
| `new` | Written. | Refused: the file is there, or the name is not valid. | |
| `check` | Every spec is usable. | A spec has a gap. | A path is missing, a folder holds no spec, or a file cannot be read. |
| `draft` | Every spec was drafted. | A spec was refused or failed, or a path is missing. | No spec was found, or a remote provider is blocked. |
| `walk` | Every step passed. | A step failed or was blocked, or there was a finding. | Nothing was walked, a journey was refused, or a path is missing. |
| `coverage` | Written. | A file cannot be read. The coverage of the others is still written. | Coverage is not available, or a flag is wrong. |
| `report` | Written. | | The specs folder is not there, or holds no spec. |

## Files

| File | Written by |
|---|---|
| `docs/codelead/specs/<name>.md` | `new`. You edit it. |
| `docs/codelead/specs/<name>.journeys.json`, `.journeys.md` | `draft`. |
| `docs/codelead/specs/<name>.walk.json`, `.walk.md`, `<name>.screenshots/` | `walk`. |
| `docs/codelead/specs/coverage.json`, `coverage.md` | `coverage`. |
| `docs/codelead/specs/README.md`, `.codelead/spec-report.html` | `report`, and again after `walk` and `coverage`. |

## See also

- [`codelead walk`](codelead-walk.md): the walk that `spec walk` uses.
- [Journeys format](../reference/journeys-format.md): what `spec draft` writes.
- [`/implement`](../chat-commands/implement.md): `--spec` walks a spec after a landing.
- [Explore an application](../analyst/explore-an-application.md): what `spec coverage` runs.
- [The walk and parity](../modernization/walk-and-parity.md)
- [Validation and the gate](../concepts/validation-and-the-gate.md)
