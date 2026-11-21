---
title: codelead analyst facts
description: List what one or more recordings show of the application - its screens, its server's surface, its users and its journeys - as facts for its dossier, each with its evidence.
kind: reference
area: analyst
date: 2026-11-20
---

# codelead analyst facts

**Applies to:** CodeLead 0.1 and later

`codelead analyst facts` reads one or more recordings and lists what they show of the application, as facts for its dossier. It writes `facts.observed.json` and `facts.observed.md`. Each fact is marked `observed`, with the steps that show it. The command reads files only: it calls no model and opens no browser.

## Name

`codelead analyst facts` - Lists the screens, the server's surface, the users and the journeys that recordings show.

## Synopsis

```console
codelead analyst facts <RECORDINGS>... [--out <FOLDER>]
codelead analyst help facts
```

## Description

The facts come in four kinds:

| Kind | Id | What it holds |
|---|---|---|
| Screens | `screen:/Products/Details/{id}` | Each path pattern, with the page titles and headings seen there, the controls used on it, and the users who reached it. Also the users that it was refused to, and the users who could not open it. |
| The server's surface | `request:POST /Products/Create` | Each method and path pattern, read or write, every status seen (0 for no answer), and the screens that called it. |
| Users | `user:clerk` | Each user key, with its account under given accounts, the screens that it reached, and the screens and requests that it was refused. Also the screens that it could not open. |
| Journeys | `journey:recording.json` | For each recording, the steps' titles in order, with their users. |

The facts also say how the users came to exist: registered through the screens, given accounts, or mixed. The recordings name users, not roles. A 403 says that a user lacks a role.

A screen is refused to a user when its opening was answered 401 or 403, or sent the user to the sign-in page. A refused screen is not one that the user reached. A screen that a user could not open (another status from 400, or no answer) is no refusal. The facts list it apart, with the words "could not open".

`facts.observed.md` reads like the System Model's `model.md`, with "observed" where the model says "read". Every entry and every control under a screen cites the steps that it was seen in, as `recording:step`. See [The System Model](../../modernization/system-model.md).

`facts` and [`rules`](analyst-rules.md) share the same reading of the recordings. A fact holds only as far as the steps that it cites. See [Limits](../limits.md#what-rules-and-facts-cannot-say).

## Arguments

- **`<RECORDINGS>...`**

  One or more `recording.json` files to read. A person's recordings and the Explorer's can be read together.

## Options

- **`--out <FOLDER>`**

  Where the files are written. The default is beside the first recording.

## Examples

- List the facts of one recording of the Basic Store sample:

  ```console
  codelead analyst facts analyst-recordings/2026-11-20T09-30-00Z/recording.json
  ```

  <!-- described, not captured -->
  ```output
  Observed in 1 recording: 6 screens, 4 requests of the server's surface, 1 user, 1 journey; written into analyst-recordings/2026-11-20T09-30-00Z/facts.observed.json and analyst-recordings/2026-11-20T09-30-00Z/facts.observed.md.
  ```

- List the facts of two recordings together:

  ```console
  codelead analyst facts analyst-recordings/2026-11-20T09-30-00Z/recording.json analyst-recordings/2026-11-20T10-15-00Z/recording.json
  ```

  <!-- described, not captured -->
  ```output
  Observed in 2 recordings: 6 screens, 10 requests of the server's surface, 2 users, 2 journeys; written into analyst-recordings/2026-11-20T09-30-00Z/facts.observed.json and analyst-recordings/2026-11-20T09-30-00Z/facts.observed.md.
  ```

- Read one screen in `facts.observed.md`:

  <!-- described, not captured -->
  ```markdown
  - `/Products/Create` (observed): title "New product - Basic Store"; headings "New product"; reached by clerk (`recording.json:s01`, `recording.json:s02`)
    - the field "SKU" (filled in) (`recording.json:s02`)
    - the field "Name" (filled in) (`recording.json:s02`)
    - the button "Create product" (`recording.json:s02`)
  ```

- Read a screen that was refused to one role, and that role's line under **Users**. In this application, only an administrator may open the page **Manage**:

  <!-- described, not captured -->
  ```markdown
  - `/Manage` (observed): title "Manage"; headings "Manage"; reached by admin; refused to clerk (answered 403, the page showing "Not allowed"; `explore-clerk/recording.json:s08`) (`explore-admin/recording.json:s08`, `explore-clerk/recording.json:s08`)
  - **clerk** (account `store.clerk`) (observed): reached `/`, `/Clients`, `/Products`, `/Orders`; refused the screen `/Manage` (answered 403; `explore-clerk/recording.json:s08`) (9 steps: …)
  ```

  A screen that the user could not open reads `could not be opened by clerk (answered 404; …)` under the screen, and `could not open /Products/Details/99 (answered 404; …)` under the user.

- Write the facts of every role of an exploration into the exploration's folder:

  ```console
  codelead analyst facts analyst-recordings/2026-11-20T09-15-00Z/explore-clerk/recording.json analyst-recordings/2026-11-20T09-15-00Z/explore-admin/recording.json --out analyst-recordings/2026-11-20T09-15-00Z
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The facts were written. |
| `1` | No recording was given, or an option is not known. Also a folder without the pro plan. |
| `2` | A recording cannot be read, is not JSON, or is not a `codelead.recording/1` recording. Nothing was written. |

## Files

| File | What `facts` does with it |
|---|---|
| `<RECORDINGS>` | Reads them. |
| `facts.observed.json` | Writes the facts, `codelead.facts.observed/1`, with `recordings`, `identity`, `screens`, `findings`, `requests`, `users` and `journeys`. |
| `facts.observed.md` | Writes the same facts, to read. |

See [Files the Analyst writes](../files.md#factsobservedjson-and-factsobservedmd).

## See also

- [Roles, journeys and steps](../concepts.md#rules-and-facts)
- [`codelead analyst rules`](analyst-rules.md): what the same recordings show the application enforcing.
- [`codelead analyst record`](analyst-record.md) and [`codelead analyst explore`](analyst-explore.md): make the recordings.
- [The dossier](../../modernization/dossier.md)
- [The System Model](../../modernization/system-model.md)
- [Limits](../limits.md)
