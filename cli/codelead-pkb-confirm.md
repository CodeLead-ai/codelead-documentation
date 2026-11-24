---
title: codelead pkb confirm
description: Confirm an entry of a project's knowledge, by a person's act with a name and a reason. The answers that cited it are marked.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb confirm

**Applies to:** CodeLead 0.1 and later

`codelead pkb confirm` raises an entry of the [PKB service](codelead-pkb.md) to **confirmed**. A person does it, with their name and a reason. The entry's history keeps both. The stored answers that cited the entry at its older version are marked, so that you can ask them again.

> [!IMPORTANT]
> A person can confirm an entry, or downgrade it. A person never makes an entry evidence-backed: only evidence does that. A confirm of an evidence-backed entry is refused, because it would lower the entry.

## Name

`codelead pkb confirm` - Confirms an entry, by a person's act with a name and a reason, and marks the answers that cited its older version.

## Synopsis

```console
codelead pkb confirm <ENTRY_ID> --by <NAME> --reason "<WORDS>" [--dir <FOLDER>] [--json]
codelead pkb confirm -h|--help
```

## Description

`confirm` changes the entry's trust through the store's one path for a trust change. The entry gets a new version. A history line says "confirmed", who did it and why.

Then each stored answer that cited the entry at an older version gains a mark. The mark says that the evidence the answer rested on changed, and how. The answer itself stays as it was. See [Marks](codelead-pkb-asks.md#marks).

The name and the reason are scanned before they are stored, with the project's own rules. A secret, a path of this machine or a private name in them is replaced. A line says how many things the scan replaced.

`confirm` goes through the service when one is named. Then the service's stream tells each page and client that the entry changed. See [How the commands find the service](codelead-pkb-serve.md#how-the-commands-find-the-service).

## Options

- **`<ENTRY_ID>`**

  The entry's id, such as `e-ffa9d7b67c42d3734e22`. [`codelead pkb entries`](codelead-pkb-entries.md) lists them.

- **`--by <NAME>`**

  Who confirms it. Required. The entry's history names them.

- **`--reason <WORDS>`**

  Why, in words. Required. The history keeps the reason, and the marks of the answers name it.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The command then writes the store on this machine, not through the service. The default is `<home>/.codelead/pkb`.

- **`--json`**

  Prints the result as one JSON document: `changed`, `marked`, and the entry as it is now.

## Examples

These examples use the sample project `tally`, through a service on port `7425`.

- Confirm a reported entry:

  ```console
  codelead pkb confirm e-ffa9d7b67c42d3734e22 --by person-a --reason "The project file says net48"
  ```

  ```output
  The entry e-ffa9d7b67c42d3734e22 ("The web project targets .NET Framework 4.8") is confirmed now, at version 2, by person-a.
  Stored answers marked: 0.
  ```

- Confirm an entry that two stored answers cited:

  <!-- described: the words that the command prints, as its tests check them -->
  ```output
  The entry e-0b1c2d3e4f5a6b7c8d9e ("Decided on card 12: invoice totals round half up") is confirmed now, at version 3, by person-a.
  Stored answers marked: 2, because they cited an older version of it. codelead pkb asks shows them; codelead pkb reask asks one again.
  ```

- Confirm an evidence-backed entry. It is refused, and nothing is written:

  ```console
  codelead pkb confirm e-b595123f83a4598ef02d --by person-a --reason "A test of the rule"
  ```

  ```output
  Entry e-b595123f83a4598ef02d is backed by evidence: a confirm would lower it.
  Nothing was written.
  ```

- Forget the reason:

  ```console
  codelead pkb confirm e-ffa9d7b67c42d3734e22 --by person-a
  ```

  ```output
  Say why, in words: --reason "<words>". The entry's history keeps the reason, and the marks of the answers name it.
  Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The entry is confirmed now, or it was confirmed already. |
| `1` | It started and failed. The words say why. |
| `2` | Refused: no id, no `--by`, no `--reason`, no such entry, an evidence-backed entry, the store does not answer, or the service named does not answer. |

## See also

- [`codelead pkb downgrade`](codelead-pkb-downgrade.md): the other act of a person on an entry.
- [`codelead pkb entries`](codelead-pkb-entries.md): the ledger, and one entry with its history.
- [`codelead pkb asks`](codelead-pkb-asks.md): the marks of the stored answers.
- [PKB service API](../reference/pkb-service-api.md): `POST /api/entries/<id>/confirm`.
- [`codelead pkb`](codelead-pkb.md)
