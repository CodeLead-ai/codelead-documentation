---
title: Parity waivers
description: The format of docs/codelead/parity-waivers.json - the page differences that you accept on purpose, how codelead parity --check reads them, and what a waived route looks like in the report.
kind: reference
area: reference
date: 2026-11-20
---

# Parity waivers

**Applies to:** CodeLead 0.1 and later

**Page parity** compares the pages of a running application with a baseline that CodeLead captured before a change. A route whose page changed fails the check. When the change is intended, you record a **waiver**: the route, the reason, who decided, and when. A waived route passes, and the report carries its reason. An intended change is never a silent pass.

> [!NOTE]
> **New to this?** A *route* is a page's address, such as `/Products`. [`codelead parity --capture`](../cli/codelead-parity.md) saves each route's page as it is. `codelead parity --check` fetches them again and compares.

## The file

| Fact | Value |
|---|---|
| Path | `docs/codelead/parity-waivers.json` at the repository root. |
| Schema | none. |
| Written by | You. |
| Read by | `codelead parity --check`. Not by `codelead parity --rebuild`: a rebuild records its decisions in its [manifest](rebuild-manifest.md#decisions). |
| In git | Yes. A waiver is a decision that a reviewer must see. |

A file that is missing or cannot be read counts as no waiver.

## Format

The file is an object with a `waivers` list, or the list alone:

```json
{
  "waivers": [
    {
      "route": "/Products",
      "reason": "The product list now shows the stock, on purpose.",
      "by": "sam",
      "date": "2026-11-20"
    }
  ]
}
```

| Key | Type | What it holds |
|---|---|---|
| `route` | string | The route, exactly as the report writes it, such as `/Products` or `/` for the default page. |
| `reason` | string | Why the difference is intended. |
| `by` | string | Who decided. |
| `date` | string | When, as `YYYY-MM-DD`. |

All four are shown in the report. Write each one: a waiver without a reason or a name tells a reviewer nothing.

## How the check uses a waiver

For each route of the baseline, the check gives one state:

| State | Meaning | Fails the check |
|---|---|---|
| `same` | The page is the same as in the baseline. | No |
| `changed` | The page differs. | Yes |
| `missing` | The route no longer answers. | Yes |
| `new` | A route that the baseline does not have. | No |
| `error` | The route could not be fetched. | Yes |
| `waived` | The page differs, or is missing, and a waiver names the route. | No |

A waiver covers `changed` and `missing`. It never covers `error`.

## Example

The Basic Store sample, after a change that adds the stock to the product list. First the baseline, before the change:

```console
codelead parity --capture http://localhost:5080 --routes /,/Products,/Orders
```

```output
Parity baseline: 3 route(s) captured from http://localhost:5080 into docs/codelead/parity/baseline. Declare `codelead parity --check http://localhost:5080` in the validation gate before the UI changes.
```

The check after the change, without a waiver, ends with exit code 1:

```output
# Page parity — http://localhost:5080

Checked 2026-11-20T07:45:00.740Z: 3 route(s) — 2 same, 1 changed, 0 missing, 0 new, 0 error, 0 waived. **FAIL** (a changed, missing or errored route fails; a waived one carries its reason).

| Route | State | Detail |
|---|---|---|
| `/` | same |  |
| `/Products` | changed | line 1: before "<h1>Products</h1><ul><li>Coffee beans</li></ul>", after "<h1>Products</h1><ul><li>Coffee beans (12 in stock)</li></ul>" |
| `/Orders` | same |  |
```

With the waiver above, the same check passes, with exit code 0:

```output
Checked 2026-11-20T07:45:01.494Z: 3 route(s) — 2 same, 0 changed, 0 missing, 0 new, 0 error, 1 waived. **PASS** (a changed, missing or errored route fails; a waived one carries its reason).

| Route | State | Detail |
|---|---|---|
| `/` | same |  |
| `/Products` | waived | line 1: before "<h1>Products</h1><ul><li>Coffee beans</li></ul>", after "<h1>Products</h1><ul><li>Coffee beans (12 in stock)</li></ul>"; waived: The product list now shows the stock, on purpose. (sam, 2026-11-20) |
| `/Orders` | same |  |
```

The report is written to `docs/codelead/parity/latest/parity-report.json`, schema `codelead.page-parity/1`, beside the new capture.

## Waivers are not the other records of a decision

| You decided | Record it in |
|---|---|
| A page of the old application changes on purpose | `docs/codelead/parity-waivers.json` (this page) |
| A feature is not carried into a rebuild | `decisions.dropped` of the [rebuild manifest](rebuild-manifest.md#decisions). Parity counts it as `dropped by decision`. |
| A finding of the extraction gate is intended | `extraction-dispositions.json`. See [Extraction dispositions and security waivers](dispositions-and-waivers.md). |
| A security finding is not fixed | `security-waivers.json`, with [`/modernize --waive`](../chat-commands/modernize.md). |
| A read that the application refuses on purpose | `refusedByDesign` of the [journeys file](journeys-format.md#refusedbydesign). |

## See also

- [`codelead parity`](../cli/codelead-parity.md)
- [The walk and parity](../modernization/walk-and-parity.md)
- [Validation file](validation-file.md): declare `codelead parity --check` in the gate
- [Extraction dispositions and security waivers](dispositions-and-waivers.md)
- [Rebuild manifest](rebuild-manifest.md)
