---
title: codelead analyst draft
description: Turn a recording into a journey specification (codelead.journeys/1), with a list of what to confirm, and check what was drafted.
kind: reference
area: analyst
date: 2026-11-20
---

# codelead analyst draft

**Applies to:** CodeLead 0.1 and later

`codelead analyst draft` turns a recording into a journey specification in CodeLead's format, `codelead.journeys/1`. It writes `journeys.json`, the specification, and `journeys.md`, the list of what to confirm. Then it checks what it drafted, as [`codelead analyst check`](analyst-check.md) does. The command calls no model, and opens no browser.

## Name

`codelead analyst draft` - Turns a recording into a journey specification and a list of what to confirm.

## Synopsis

```console
codelead analyst draft <RECORDING> [--out <FOLDER>]
codelead analyst help draft
```

## Description

The draft makes one journey for each journey that the recording names, in the order recorded. Steps marked by hand, outside any journey, make one journey of their own. A journey's id comes from its name: `create a product` gives `create-a-product`.

Each recorded action becomes an action of the walk:

- A click becomes `{ "click": ["Create product"], "role": "button" }`, by the control's name and role.
- A value typed becomes `{ "fill": { "label": "SKU" }, "value": "ACC-1001" }`, by the field's label, else its placeholder, else its place among the text boxes.
- An option chosen becomes `{ "select": { "label": "Status" }, "value": "Cancelled" }`, with the option's text.
- An address typed becomes `{ "open": "/" }`. The walk opens only the first page.

Each step expects, in this order:

1. The address that it ended on, as a pattern in which each id is general: `^/Products/Details/[^/]+$`.
2. The writes of the step, accepted or refused: `{ "responses": { "method": "POST", "path": "^/Products/Create$" } }`.
3. The headings and alerts that the page showed, as text outcomes.
4. What the person marked with **Expect this**, as text outcomes.

A password is never drafted. The user's own password becomes `{clerk.password}`. A password that was not the account's becomes `{clerk.wrongPassword}`, which the walk builds to be different.

`journeys.md` lists what to confirm in four groups. They are **Refused by design?**, **Values that look like secrets**, **Nothing to assert: confirm by eye**, and **Other things to confirm**. Then it shows each step in words. See [From a recording to a specification](../from-recording-to-spec.md).

The draft reads a person's recording and an Explorer's recording in the same way. The Explorer runs the draft for each role by itself, unless you give `--no-derive`.

## Arguments

- **`<RECORDING>`**

  The `recording.json` to draft from.

## Options

- **`--out <FOLDER>`**

  Where `journeys.json` and `journeys.md` are written. The default is beside the recording. The modernization program looks for the drafts beside the recordings, so keep the default there.

## Examples

- Draft a recording of the Basic Store sample:

  ```console
  codelead analyst draft analyst-recordings/2026-11-20T09-30-00Z/recording.json
  ```

  <!-- described, not captured -->
  ```output
  Drafted 9 steps in 4 journeys into analyst-recordings/2026-11-20T09-30-00Z/journeys.json and analyst-recordings/2026-11-20T09-30-00Z/journeys.md.
  To confirm (journeys.md): 0 refused, 0 secret-looking, 0 with nothing to assert, 1 other.
  analyst-recordings/2026-11-20T09-30-00Z/journeys.json: no problems; CodeLead can walk it.
  ```

- Draft a recording whose first journey types a wrong password on the sign-in page:

  ```console
  codelead analyst draft analyst-recordings/2026-11-20T10-15-00Z/recording.json
  ```

  <!-- described, not captured -->
  ```output
  Drafted 3 steps into analyst-recordings/2026-11-20T10-15-00Z/journeys.json and analyst-recordings/2026-11-20T10-15-00Z/journeys.md.
  To confirm (journeys.md): 1 refused, 1 secret-looking, 0 with nothing to assert, 0 other.
  analyst-recordings/2026-11-20T10-15-00Z/journeys.json: no problems; CodeLead can walk it.
  ```

  `journeys.md` asks you to confirm that the refused sign-in is on purpose, and that the step types `{clerk.wrongPassword}`.

- Write the draft into another folder:

  ```console
  codelead analyst draft analyst-recordings/2026-11-20T09-30-00Z/recording.json --out drafts/store
  ```

- Draft a file that is not a recording, such as a journeys file. Nothing is written, and the exit code is 2:

  ```console
  codelead analyst draft analyst-recordings/2026-11-20T09-30-00Z/journeys.json
  ```

  <!-- described, not captured -->
  ```output
  analyst-recordings/2026-11-20T09-30-00Z/journeys.json: schema: expected "codelead.recording/1", found "codelead.journeys/1".
  ```

- Draft a recording that the walk could not read as drafted. The draft is written, and the exit code is 1:

  <!-- described, not captured -->
  ```output
  Drafted 4 steps into … and ….
  To confirm (journeys.md): 0 refused, 0 secret-looking, 0 with nothing to assert, 1 other.
  …/journeys.json: 1 problem
    journeys[0].steps[0].do[0].open: opens "/Products/Create": a walk starts at "/" and reaches every other page by clicking (no address typed).
  CodeLead's walk would refuse the draft as it is: fix the problems above (journeys.md says how).
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The draft was written, and the walk can read it. |
| `1` | The draft was written, and the check found problems. Fix them as `journeys.md` says. Also an option that the command does not know, and a folder without the pro plan. |
| `2` | Nothing was written: the recording cannot be read, is not JSON, or is not a `codelead.recording/1` recording. |

## Files

| File | What `draft` does with it |
|---|---|
| `<RECORDING>` | Reads the recording. |
| `journeys.json` beside the recording, or in `--out` | Writes the specification, `codelead.journeys/1`. It replaces an earlier draft. |
| `journeys.md` beside it | Writes the list of what to confirm, and each step in words. |

See [Files the Analyst writes](../files.md) and [Journeys format](../../reference/journeys-format.md).

## See also

- [From a recording to a specification](../from-recording-to-spec.md)
- [`codelead analyst record`](analyst-record.md): make the recording.
- [`codelead analyst check`](analyst-check.md) and [`codelead analyst replay`](analyst-replay.md): the next commands.
- [Roles, journeys and steps](../concepts.md)
- [Journeys format](../../reference/journeys-format.md)
- [The Analyst view](../../extension/analyst-view.md)
