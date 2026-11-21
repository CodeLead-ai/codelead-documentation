---
title: codelead analyst replay
description: Walk a journey specification against the running application with CodeLead's walk, as a new user would, and print every step as pass, fail or blocked.
kind: reference
area: analyst
date: 2026-11-20
---

# codelead analyst replay

**Applies to:** CodeLead 0.1 and later

`codelead analyst replay` walks a journey specification against the running application with CodeLead's walk. It uses the application as a new user would, and prints every step as pass, fail or blocked. It writes nothing, and it calls no model.

## Name

`codelead analyst replay` - Walks a journey specification against the running application.

## Synopsis

```console
codelead analyst replay <JOURNEYS> --base <URL> [--api <ORIGIN>] [--json] [--headed]
codelead analyst help replay
```

## Description

The replay runs the same walk as [`codelead walk --no-write`](../../cli/codelead-walk.md), in Playwright's Chromium:

- It starts every journey at the first page, and reaches every other page by clicking. It never jumps to an address.
- For new users, it registers its own users through the screens, with names and passwords that it makes up for each run.
- For given accounts, it signs in as the accounts that the file names. The passwords come from `CODELEAD_WALK_PASSWORD_<USER>`, or from the file that `CODELEAD_WALK_CREDENTIALS` names. See [Passwords](../passwords.md).
- For Windows sign-in, the browser answers the application's challenge as the account, with its password when one is given. Otherwise it uses this computer's session, and the replay prints which one signed in.

After each step, the walk checks what the step expects and marks it **pass** or **fail**. A step after a failed step is **blocked**. At the end, it prints the counts and the **findings**: what no step expected, such as a refused request or an error on a page.

The walk refuses a server that signs a visitor in by itself with a development identity. Serve the application without one. See [The walk and parity](../../modernization/walk-and-parity.md).

> [!WARNING]
> A replay sends the saves of the journeys again. A journey that creates a product creates one on each replay. Replay on a copy whose database you can reset.

## Arguments

- **`<JOURNEYS>`**

  The `journeys.json` to replay.

## Options

- **`--base <URL>`**

  Required. The application's address, served without any development identity.

- **`--api <ORIGIN>`**

  The origin of the application's server, when it is not the address's origin.

- **`--json`**

  Prints the walk's report as JSON, in place of the step lines. It is the same report that `codelead walk --json` prints.

- **`--headed`**

  Shows the browser while the walk runs.

## Examples

- Replay a journeys file of the Basic Store sample on the old application:

  ```console
  codelead analyst replay analyst-recordings/2026-11-20T09-30-00Z/journeys.json --base http://localhost:5080/
  ```

  <!-- described, not captured -->
  ```output
  PASS    create-a-product · New product on / — clicked "New product"
  PASS    create-a-product · Create product on /Products/Create — clicked "Create product"

  2 steps: 2 pass, 0 fail, 0 blocked. 0 findings.
  ```

- Replay while the application is not running. The first step fails, the next ones are blocked, and the exit code is 1:

  <!-- described, not captured -->
  ```output
  FAIL    create-a-product · New product on / — the page threw: page.goto: net::ERR_CONNECTION_REFUSED at http://localhost:5080/
  BLOCKED create-a-product · Create product on /Products/Create — waits on s01

  2 steps: 0 pass, 1 fail, 1 blocked. 0 findings.
  ```

- Replay a file of given accounts with no password set. Nothing is walked, and the exit code is 2:

  ```console
  codelead analyst replay analyst-recordings/2026-11-20T10-15-00Z/journeys.json --base http://localhost:5080/
  ```

  ```output
  Nothing was walked: the walk signs in as the accounts it is given, each with its password, and has none for clerk (store.clerk): set CODELEAD_WALK_PASSWORD_CLERK, or name with CODELEAD_WALK_CREDENTIALS a file outside the repository that maps each account to its password.
  ```

- Replay the journeys of an intranet application behind Windows authentication. The line before the counts says which account signed in:

  <!-- described, not captured -->
  ```output
  PASS    time-tracking · Open the timesheet — clicked "Timesheet"

  Sign-in: signed in as the account "CORP\store.clerk" with the machine's own session.
  1 steps: 1 pass, 0 fail, 0 blocked. 0 findings.
  ```

- Print the walk's report as JSON, for a script:

  ```console
  codelead analyst replay analyst-recordings/2026-11-20T09-30-00Z/journeys.json --base http://localhost:5080/ --json
  ```

  <!-- described, not captured -->
  ```output
  {
    "schema": "codelead.walk/1",
    "ranAt": "2026-11-20T09:52:08.017Z",
    "base": "http://localhost:5080/",
    "steps": [
      {
        "journey": "create-a-product",
        "id": "s01",
        "title": "New product on /",
        "user": "clerk",
        "status": "pass",
        "detail": "clicked \"New product\"",
  …
  ```

- Give a file that does not exist:

  ```console
  codelead analyst replay missing.json --base http://localhost:5080/
  ```

  ```output
  No journey specification at missing.json.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Every step passed, and there was no finding. |
| `1` | A step failed or was blocked, or there was a finding. Also `--base` missing, and a folder without the pro plan. |
| `2` | Nothing was walked: no file, a file that the walk refuses, a user key that breaks the rule (a letter first, then letters, digits or underscores), a password not set, Chromium not installed, or a server that lends an identity. |

## Files

| File | What `replay` does with it |
|---|---|
| `<JOURNEYS>` | Reads the journeys. The replay writes nothing. |
| The file that `CODELEAD_WALK_CREDENTIALS` names | Reads the passwords of given accounts, by account. It must be outside the repository. |
| `.env` | Reads `CODELEAD_WALK_PASSWORD_<USER>`, when the variable is not set in the terminal. |

## See also

- [From a recording to a specification](../from-recording-to-spec.md)
- [`codelead analyst check`](analyst-check.md): check the file first.
- [`codelead walk`](../../cli/codelead-walk.md): the same walk, with a report written for a rebuild.
- [The walk and parity](../../modernization/walk-and-parity.md)
- [Passwords](../passwords.md)
- [Limits](../limits.md)
