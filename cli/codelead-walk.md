---
title: codelead walk
description: Walk the running application as a new user would, through the screens, and report every journey step as pass, fail or blocked.
kind: reference
area: cli
date: 2026-11-25
---

# codelead walk

**Applies to:** CodeLead 0.1 and later

`codelead walk` uses the running application the way a new person does. It registers through the screens, or signs in as the accounts that a spec names. It clicks through the menus, and it reports every journey step as pass, fail or blocked.

## Name

`codelead walk` - Walks the running application as a new user would, and reports each journey step.

## Synopsis

```console
codelead walk --base <URL> [--rebuild [<ROOT>]] [--journeys <FILE>] [--api <ORIGIN>]
              [--identity-header <PREFIX>] [--time-zone <ZONE>] [--no-write] [--headed] [--json]
codelead walk -h|--help
```

## Description

A **journey** is a path that a person takes through the application, step by step, with what they should see after each step. The journeys of a rebuild are in `docs/codelead/rebuild/journeys.json`. See [Journeys format](../reference/journeys-format.md).

The walk follows each journey in a real browser (Playwright's Chromium):

- It starts at the first screen, and reaches every page by clicking. It never jumps to a URL.
- It prepares nothing by hand. A new user registers through the screens. For a spec with a given identity, the walk signs in as the accounts that the spec names. The browser answers the server's own challenge, or the journey's steps type the account into the sign-in form. See [What a given identity does](../reference/journeys-format.md#what-a-given-identity-does-and-what-it-does-not).
- After each step, it checks what the step expects, and marks the step **pass** or **fail**. A step after a failed step is **blocked**.

> [!NOTE]
> **New to this?** Tests check the code from the inside. The walk checks the application from the outside, as a person would. This is why a rebuild is finished only after a clean walk. See [The walk and parity](../modernization/walk-and-parity.md).

### An honest walk

The walk refuses to record a result that it cannot trust. It stops with exit code 2 when the server lends a development identity (a request header such as `x-dev-user`). It also stops when the spec is refused, when the root is not a rebuild, or when the browser is not installed. Serve the application **without** any development identity.

Before the first journey, the walk sends one request to `--base`. Any HTTP answer counts, even a 401 or a 404. When nothing answers, the walk stops with exit code 2. It walks no step, runs no reset and writes no report. A report from an earlier walk stays as it was.

The report records **what** was walked: the root's code stamp, the spec's path and its content hash. [`codelead parity --rebuild`](codelead-parity.md) trusts a walk only for the code and the journeys that it walked. When the code changed while the walk ran, the report says so.

### Passwords

The passwords of a spec's accounts come from `CODELEAD_WALK_PASSWORD_<USER>`, or from the file that `CODELEAD_WALK_CREDENTIALS` names. They never come from the spec, and the walk never prints them. See [Passwords](../analyst/passwords.md).

> [!TIP]
> **For experienced users:** the walk reads the server's day in the `--time-zone` zone. This day sets each step's server time, `{today}`, and the journeys bound to one day. Without it, the walk uses the run profile's zone or the spec's.

`--rebuild` is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, `codelead walk --rebuild` is refused before anything is walked, with exit code 2. The walk of a spec, without `--rebuild`, stays free. See [Licence and plans](../concepts/licence-and-plans.md).

## Options

- **`--base <URL>`**

  Required. The application's base URL, served **without** any development identity.

- **`--rebuild [<ROOT>]`**

  The rebuild repository. The default is the current folder. The report is written under its `docs/codelead/parity`, so the root must hold `docs/codelead/rebuild/rebuild.json`.

- **`--journeys <FILE>`**

  The journey spec. The default is `docs/codelead/rebuild/journeys.json` under the root.

- **`--api <ORIGIN>`**

  The API's origin, when it is on another host than the base. Without it, the walk watches what the pages ask of the base's host, on any port.

- **`--identity-header <PREFIX>`**

  How the names of more request headers that lend a development identity start. The walk stops when it sees one of them, as it always does on `x-dev-`.

- **`--time-zone <ZONE>`**

  The server's time zone: an IANA name such as `Europe/Berlin`, or `UTC`. The walk reads the server's day in it. The default is the run profile's zone or the spec's.

- **`--no-write`**

  Prints the report without writing `walk.md` and `walk.json`. Any application can be walked this way, a rebuild or not.

- **`--headed`**

  Shows the browser while the walk runs.

- **`--json`**

  Prints the report as JSON.

## Examples

- Walk the rebuild of the Basic Store sample while its servers are not running. Nothing answers at the base, so the walk stops before the first journey, and the exit code is 2:

  ```console
  codelead walk --base http://127.0.0.1:5791 --rebuild
  ```

  ```output
  Nothing answers at http://127.0.0.1:5791 (connect ECONNREFUSED 127.0.0.1:5791): nothing was walked, the run profile's reset was not run and no walk report was written. Start the application first (for a rebuild: the API and the web application its run profile names, run.api and run.web in docs/codelead/rebuild/rebuild.json), then walk again.
  ```

  Start the rebuild's API and web application first, then walk again. The walk then prints each step as `PASS`, `FAIL` or `BLOCKED`, writes `docs/codelead/parity/walk.md`, and ends with the counts. When every step passes, the last line counts only passes, and the exit code is 0.

- Walk a rebuild whose only failing step also fails on the old application. The journeys' proof, `docs/codelead/rebuild/journeys-proof.json`, says so. So the step is a defect of the drafted journey, not of the rebuild. The walk says it apart, after its counts, and the exit code is 0:

  ```console
  codelead walk --base http://localhost:3000 --rebuild
  ```

  <!-- described, not captured -->
  ```output
  …
  FAIL    orders.cancel · Click "Cancel this order": … — nothing to click named "Cancel this order" on /orders/details?id=3
  …
  39 steps: 38 pass, 1 fail, 0 blocked. 0 findings.
  Not the rebuild's: orders.cancel-5 fails on the old application as well (the proof of 2026-11-25): a defect of the drafted journey, not of the rebuild. Every other step passes.
  ```

  A finding at another step still counts. Then the line ends without `Every other step passes.`, and the exit code is 1. See [The old application's journeys](../modernization/walk-and-parity.md#the-old-applications-journeys).

- Walk a spec whose user key does not start with a letter. The walk refuses the spec before it sends anything, and the exit code is 2:

  ```console
  codelead walk --base http://127.0.0.1:5173 --journeys bad-key.json --no-write
  ```

  ```output
  The journey spec is refused and nothing was walked: The user key "1clerk" is not one a placeholder can name: a user's key starts with a letter (A to Z), then letters, digits or underscores, such as alpha or clerk_2, and "1clerk" does not start with a letter. Rename it in the spec's users and in its steps.
  ```

- Walk any application with a journey spec of your own, and write nothing:

  ```console
  codelead walk --base http://localhost:5173 --journeys specs/journeys.json --no-write
  ```

  <!-- described, not captured -->
  ```output
  PASS    add-product · Open the application at its root: …
  PASS    add-product · Click "New product": …
  PASS    add-product · Click "Create product": … I see "Product 6"
  …
  3 steps: 3 pass, 0 fail, 0 blocked. 0 findings.
  ```

- Walk outside a rebuild without `--no-write`. The walk refuses, and says the two ways on:

  ```console
  codelead walk --base http://localhost:5791
  ```

  ```output
  <your project> is not a rebuild (no docs/codelead/rebuild/rebuild.json), so no walk report is written there. Pass --rebuild <root>, or --no-write to walk without recording.
  ```

- Give an address without `http://`:

  ```console
  codelead walk --base localhost:5183
  ```

  ```output
  --base is not an http address: localhost:5183 (for example http://localhost:5183).
  ```

- Give a time zone that the machine does not know:

  ```console
  codelead walk --base http://localhost:5791 --time-zone Mars/Base
  ```

  ```output
  --time-zone is not a time zone this machine knows: Mars/Base (an IANA name such as Europe/Berlin, or UTC).
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Every step passed, and there was no finding. Or every step that did not pass fails on the old application as well, by the journeys' proof, or waits on such a step. Then no finding is at another step, and the last line starts with `Not the rebuild's:`. |
| `1` | A step of the rebuild failed or was blocked, there was a finding at a step of the rebuild, or no step was walked. A journey whose clock was not put back is `1` too. |
| `2` | Nothing honest could be recorded: a refused spec, nothing that answers at `--base`, a server that lends an identity, a root that is not a rebuild, a browser that is not installed, an option that is not valid, `--rebuild` in a folder without the Pro plan. `--base` missing is also `2`. |

## Files

| File | What `walk` does with it |
|---|---|
| `docs/codelead/rebuild/journeys.json`, or `--journeys` | Reads the journeys. |
| `docs/codelead/rebuild/rebuild.json` | Reads the rebuild's manifest, its run profile and its day reset. |
| `docs/codelead/parity/walk.md`, `walk.json` | Writes the report. Not with `--no-write`, and not when nothing answers at `--base`. |
| `docs/codelead/rebuild/journeys-proof.json` | Reads the journeys' proof on the old application, when it is there. A step that fails there is said apart. With `--json`, that line goes to the standard error. |
| `CODELEAD_WALK_CREDENTIALS` | Reads the passwords from the file that it names, when it is set. |

## See also

- [The walk and parity](../modernization/walk-and-parity.md)
- [Journeys format](../reference/journeys-format.md)
- [`codelead parity`](codelead-parity.md): `--rebuild` needs a clean walk.
- [`codelead spec`](codelead-spec.md): walk specs written in plain words.
- [`codelead analyst`](codelead-analyst.md): `replay` walks a recorded journey.
- [Passwords](../analyst/passwords.md)
