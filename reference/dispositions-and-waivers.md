---
title: Extraction dispositions and security waivers
description: The two files in which a person records a reason - extraction-dispositions.json for the findings of the extraction gate, and security-waivers.json for the security findings left unfixed - their keys, their formats and who reads them.
kind: reference
area: reference
date: 2026-11-20
---

# Extraction dispositions and security waivers

**Applies to:** CodeLead 0.1 and later

Two gates of a modernization stop on findings that may be intended. CodeLead does not decide that a finding is intended. A person records the reason, under the finding's key, in a file that is committed with the old application. The reason explains the finding: it never changes what CodeLead found.

| File | For | Schema | Written by |
|---|---|---|---|
| [`docs/codelead/rebuild/extraction-dispositions.json`](#extraction-dispositions) | the findings of the extraction gate | `codelead.extraction-dispositions/1` | you, and the program's model stage for the ASP.NET Identity scaffold |
| [`docs/codelead/security-waivers.json`](#security-waivers) | the security findings that you decided not to fix | `codelead.security-waivers/1` | [`/modernize --waive`](../chat-commands/modernize.md#--waive) |

Both live in the **old** application's repository, and both belong in git. For a decision about the **new** application, use the `decisions` of the [rebuild manifest](rebuild-manifest.md#decisions).

## Extraction dispositions

[`codelead model`](../cli/codelead-model.md) reads the old application into its [System Model](../modernization/system-model.md). Then it runs the **extraction gate**: a set of rules that find what the model cannot account for. An entry point that nothing links to is one such finding. Each finding has a key. A finding that is intended needs a reason under its key, or the rebuild's plan stays blocked.

### The file

| Fact | Value |
|---|---|
| Path | `docs/codelead/rebuild/extraction-dispositions.json` in the old application. |
| Schema | `codelead.extraction-dispositions/1`. Required. |
| Read by | `codelead model` (the gate), the program's model stage, `/modernize --rebuild-plan`, [`codelead rebuild init`](../cli/codelead-rebuild-init.md), and [`codelead register --dispositions`](../cli/codelead-register.md). |
| In git | Yes. |

No file means no reason. A file that is there and is not a record of reasons is refused. A typing mistake must never lose a reason, nor reopen every finding without a word:

```output
docs/codelead/rebuild/extraction-dispositions.json is not a record of reasons: expected { "schema": "codelead.extraction-dispositions/1", "dispositions": { "<key>": "<reason>" } }.
```

### Format

```json
{
  "schema": "codelead.extraction-dispositions/1",
  "dispositions": {
    "unexplained-orphan:GET /Account/ConfirmEmail": "ASP.NET Identity scaffold route; nothing live reaches it; answer identity=keep",
    "entry-point-without-trigger:GET /Home/Contact": "The contact page is reached from a link in the e-mails that the store sends, not from a page.",
    "unread-field:Product.LegacyCode": "Kept for the export to the old warehouse system, which reads the table directly."
  }
}
```

| Key | Type | What it holds |
|---|---|---|
| `schema` | string | `codelead.extraction-dispositions/1`. |
| `dispositions` | object | By the gate's key: the reason, as a text. A reason that is not a text is refused. |

### The keys

The gate report, `docs/codelead/rebuild/extraction-gate.md`, prints each finding beside its key. Copy the key from there.

| Key form | Explains |
|---|---|
| `<rule>:<subject>` | The finding of that rule with that subject. |
| `<rule>:<subject>@<file>:<line>` | One of two findings of a rule that share a subject at two places, such as two functions of one name in two files. |
| `unexplained-orphan:<file>:<line>` | Every orphan that only that place keeps. |

The rules that take a reason:

| Rule | Its finding |
|---|---|
| `entry-point-without-trigger` | An entry point that nothing triggers. |
| `unreached-code` | Code that no entry point reaches. |
| `push-without-handler-or-sender` | A real-time message that nothing handles, or that nobody sends. |
| `surface-without-layout-or-includer` | A view that no layout or page includes. |
| `unexplained-orphan` | Something that is neither live nor dead. |
| `trigger-leads-nowhere` | A link or a form that leads to no entry point. |
| `site-root-reaches-nothing` | The site's root reaches no action. |
| `unread-field` | A field that nothing reads. |
| `schema-unread` | A part of the database schema that the code never reads. |
| `schema-contradicted` | A part of the schema that the code contradicts. |

Two rules take no reason: `accounting`, which checks the extraction's own arithmetic, and `unused-disposition`. `unused-disposition` lists a reason whose key matches no finding, as a warning that never blocks. Its warning names the key, so that you find the line to fix in the file: `the reason recorded under "<key>" explains no finding: …`. An empty reason gives `holds no reason`, and a key that names several findings gives `explains nothing`.

### The ASP.NET Identity scaffold

An MVC 5 application with individual accounts brings pages for e-mail confirmation, password reset, external logins, phone numbers and two-factor sign-in. An application that never links them leaves them neither live nor dead, and the gate stops on each. On the Basic Store sample, that is 36 findings.

When every open finding is one of these pages, the program's model stage records the reasons itself, one per finding:

```text
ASP.NET Identity scaffold route; nothing live reaches it; answer identity=keep
```

It keeps every reason that is already in the file, and never replaces a person's words. When any other finding is open, it records nothing, and the stage waits: `The extraction gate needs your reasons`.

## Security waivers

The dossier's security survey finds the security findings of the old application: `security-exposure.json`. The in-place stage fixes most of them under the pins. A finding whose fix changes **who may do what** is a decision for a person. Such a finding is an action that writes without an authorization requirement, or a CORS policy that lets other origins call with the user's credentials. You fix it on purpose, or you waive it with a reason.

### The file

| Fact | Value |
|---|---|
| Path | `docs/codelead/security-waivers.json` in the old application. |
| Schema | `codelead.security-waivers/1`. |
| Written by | `/modernize --waive <finding id or key> --reason <why>`. |
| Read by | `/modernize --transform S`, the in-place stage, and the [audit pack](../modernization/audit-pack.md). |
| In git | Yes. |

### Format

<!-- described, not captured -->
```json
{
  "schema": "codelead.security-waivers/1",
  "waivers": [
    {
      "key": "unauthorized-write-action|WebApplication1/Controllers/OrdersController.cs|POST Orders/Create has no authorization requirement",
      "reason": "Guests place orders on purpose; the payment step signs them in",
      "by": "Sam",
      "date": "2026-11-20"
    }
  ]
}
```

| Key | Type | What it holds |
|---|---|---|
| `key` | string | The finding's key. |
| `reason` | string | Why the finding stays as it is. |
| `by` | string | The git `user.name` of the repository, or `operator` when there is none. |
| `date` | string | The day of the waiver, `YYYY-MM-DD`. |

A file that is a list alone, without `waivers`, is read too. A second waiver for the same key replaces the first.

### The keys

A finding's id, such as `S-004`, changes between runs of the survey. Its key does not:

| Finding class | Key |
|---|---|
| `vulnerable-package`, `end-of-life-runtime` | `<class>\|<title>`, the title without its count of known issues or its "is out of support". |
| every other class | `<class>\|<file or project>\|<title>` |

`/modernize --waive` takes the id or the key. With an id, it reads `docs/codelead/dossier/security-exposure.json` to find the key.

```console
/modernize --waive S-004 --reason "Guests place orders on purpose; the payment step signs them in"
```

<!-- described, not captured -->
```output
Waived POST Orders/Create has no authorization requirement — Guests place orders on purpose; the payment step signs them in. Recorded in docs/codelead/security-waivers.json (1 waiver(s)); the audit pack lists it under 5-security/.
```

Without the survey's report, the command says `No security exposure report at docs/codelead/dossier/security-exposure.json: run /modernize --plan first, or waive by key (class|file|title).`

### The three sides of a finding

| Side | Classes | What the in-place stage does |
|---|---|---|
| A decision for a person | `unauthorized-write-action`, `permissive-cors` | Waits, unless you answered `security`. See [Modernization state file](modernization-state-file.md#answers). |
| Automatic | the classes that a patch in place can close, such as `sql-from-strings`, `raw-html`, `missing-anti-forgery`, `insecure-cookie`, `debug-configuration` | Fixes it under the pins. |
| Closed elsewhere | `end-of-life-runtime`, `vulnerable-package`, `binary-without-source` | Names it with what closes it (the platform upgrade, a package upgrade, a dependency replacement or the rebuild). It composes no patch, and the stage does not wait or fail for it. |

A waived finding is never taken by `/modernize --transform S --security all`, nor by a class name.

### In the audit pack

The audit pack's `5-security/dispositions.json`, schema `codelead.security-dispositions/1`, lists each finding with its state:

| State | Meaning |
|---|---|
| `fixed` | The finding was there before, and is gone now. |
| `open` | The finding is still there, with no waiver. |
| `waived` | A waiver names it. The row carries `<reason> (<by>, <date>)`. |
| `new` | The finding is there now, and was not there before. |

## See also

- [The System Model](../modernization/system-model.md) and [`codelead model`](../cli/codelead-model.md)
- [Explain a finding of the extraction gate](../how-to/explain-a-gate-finding.md)
- [Waive a security finding](../how-to/waive-a-security-finding.md)
- [`/modernize`](../chat-commands/modernize.md)
- [The audit pack](../modernization/audit-pack.md)
- [Parity waivers](parity-waivers.md)
