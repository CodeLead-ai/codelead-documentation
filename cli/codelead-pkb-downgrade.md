---
title: codelead pkb downgrade
description: Downgrade an entry of a project's knowledge, by a person's act with a name and a reason. The answers that cited it are marked.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb downgrade

**Applies to:** CodeLead 0.1 and later

`codelead pkb downgrade` lowers an entry of the [PKB service](codelead-pkb.md) to **downgraded**: a person says that it no longer holds. The person gives their name and a reason, and the entry's history keeps both. The stored answers that cited the entry at its older version are marked.

## Name

`codelead pkb downgrade` - Downgrades an entry, by a person's act with a name and a reason, and marks the answers that cited its older version.

## Synopsis

```console
codelead pkb downgrade <ENTRY_ID> --by <NAME> --reason "<WORDS>" [--dir <FOLDER>] [--json]
codelead pkb downgrade -h|--help
```

## Description

A downgraded entry stays in the store, with its history. A search still finds it. Ask never gives it to the model as a source, and a later answer never rests on it.

`downgrade` works as [`codelead pkb confirm`](codelead-pkb-confirm.md#description) does:

1. It scans the name and the reason with the project's own rules.
2. It changes the entry's trust through the store's one path for a trust change. The entry gets a new version.
3. It marks each stored answer that cited the entry at an older version. See [Marks](codelead-pkb-asks.md#marks).

Any entry can be downgraded, an evidence-backed one too. Through the service, the stream tells each client that the entry changed, and how many answers it marked.

> [!NOTE]
> **New to this?** A mark says that the evidence an answer rested on changed. The answer stays as it was. [`codelead pkb reask`](codelead-pkb-reask.md) asks the question again, from what the project knows now.

## Options

- **`<ENTRY_ID>`**

  The entry's id, such as `e-91a3e6deb0d285d7b045`. [`codelead pkb entries`](codelead-pkb-entries.md) lists them.

- **`--by <NAME>`**

  Who downgrades it. Required. The entry's history names them.

- **`--reason <WORDS>`**

  Why, in words. Required. The history keeps the reason, and the marks of the answers name it.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The command then writes the store on this machine, not through the service. The default is `<home>/.codelead/pkb`.

- **`--json`**

  Prints the result as one JSON document: `changed`, `marked`, and the entry as it is now.

## Examples

These examples use the sample project `tally`, through a service on port `7425`.

- Downgrade a finding of the walk that no longer holds:

  ```console
  codelead pkb downgrade e-91a3e6deb0d285d7b045 --by person-b --reason "The list answers again since the customer is optional"
  ```

  ```output
  The entry e-91a3e6deb0d285d7b045 ("Walk finding silent-failure at step bravo-list: GET /api/invoice/list 500 and nothing on the page said so") is downgraded now, at version 2, by person-b.
  Stored answers marked: 0.
  ```

  The service's stream told each client at the same moment:

  ```output
  id: 15
  event: entry
  data: {"id":"e-91a3e6deb0d285d7b045","project":"tally","summary":"Walk finding silent-failure at step bravo-list: GET /api/invoice/list 500 and nothing on the page said so","trust_level":"downgraded","version":2,"what":"downgraded by person-b","by":"person-b"}
  ```

- Downgrade an entry that a stored answer cited. The answer is marked:

  <!-- described: the words that the command prints, as its tests check them -->
  ```output
  The entry e-0b1c2d3e4f5a6b7c8d9e ("Decided on card 12: invoice totals round half up") is downgraded now, at version 2, by person-b.
  Stored answers marked: 1, because they cited an older version of it. codelead pkb asks shows them; codelead pkb reask asks one again.
  ```

  The stream then also sends an `answers` event, with the owner, the count and the words of the mark.

- Downgrade an entry that is downgraded already. Nothing changes:

  ```console
  codelead pkb downgrade e-7cb74370cbf98f2023cc --by person-a --reason "Still true"
  ```

  ```output
  The entry e-7cb74370cbf98f2023cc was downgraded already: nothing changed.
  Stored answers marked: 0.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The entry is downgraded now, or it was downgraded already. |
| `1` | It started and failed. The words say why. |
| `2` | Refused: no id, no `--by`, no `--reason`, no such entry, the store does not answer, or the service named does not answer. |

## See also

- [`codelead pkb confirm`](codelead-pkb-confirm.md): the other act of a person on an entry.
- [`codelead pkb entries`](codelead-pkb-entries.md): the ledger, and one entry with its history.
- [`codelead pkb asks`](codelead-pkb-asks.md) and [`codelead pkb reask`](codelead-pkb-reask.md): the marked answers, and asking again.
- [PKB service API](../reference/pkb-service-api.md): `POST /api/entries/<id>/downgrade` and the stream.
- [`codelead pkb`](codelead-pkb.md)
