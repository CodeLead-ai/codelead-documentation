---
title: codelead analyst check
description: Check a journey specification against the codelead.journeys/1 format, as CodeLead's walk reads it, and print every problem with its place in the file.
kind: reference
area: analyst
date: 2026-11-20
---

# codelead analyst check

**Applies to:** CodeLead 0.1 and later

`codelead analyst check` checks a journey specification against the `codelead.journeys/1` format. It applies the rules that CodeLead's walk applies before it runs anything. Where the walk stops at the first problem, the check reports every problem, each with its place in the file. The command calls no model, opens no browser and needs no running application.

## Name

`codelead analyst check` - Checks a journey specification and prints every problem.

## Synopsis

```console
codelead analyst check <JOURNEYS>
codelead analyst help check
```

## Description

Use the check after every change that you make to a journeys file by hand. [`codelead analyst draft`](analyst-draft.md) runs the same check on what it drafts.

The check reads, among other rules:

- The shape of the file: the `schema`, the users, the journeys, the steps, and only the keys that the format knows.
- User keys by the Analyst's one rule for a role's name: a letter (A to Z, a to z), then letters, digits or underscores.
- Unique step ids, and `requires` that names earlier steps only.
- Placeholders such as `{clerk.name}` that name a user of `users`.
- An `open` only of `/`. The walk reaches every other page by clicking.
- A `fill` or a `select` that names its field by exactly one of its label, its placeholder or its place.
- A refusal that a step expects, with a text for what the user sees.
- Passwords: `{clerk.wrongPassword}` only as a fill's value. Under given accounts, a password field holds only `{clerk.password}` or `{clerk.wrongPassword}`.
- A journey's `description` and a step's `note`: text, not empty, at most 4,000 characters.

The check does not test what the walk decides only when it runs. It does not test whether the passwords are set. It does not test whether a step that reads the server's day has the server's time zone.

Each problem starts with a path into the JSON. `journeys[0].steps[1].do[1]` means the first journey, its second step, its second action. A problem of the whole file says `(the file)`. See [From a recording to a specification](../from-recording-to-spec.md#read-a-failed-check).

## Arguments

- **`<JOURNEYS>`**

  The `journeys.json` to check.

## Examples

- Check a journeys file of the Basic Store sample that has no problem:

  ```console
  codelead analyst check analyst-recordings/2026-11-20T09-30-00Z/journeys.json
  ```

  ```output
  analyst-recordings/2026-11-20T09-30-00Z/journeys.json: no problems (1 journey, 2 steps, 1 user).
  ```

- Check a file with three problems. The exit code is 1:

  ```console
  codelead analyst check analyst-recordings/2026-11-20T10-00-00Z/journeys.json
  ```

  ```output
  analyst-recordings/2026-11-20T10-00-00Z/journeys.json: 3 problems
    journeys[0].steps[0].do[0].open: opens "/Products/Create": a walk starts at "/" and reaches every other page by clicking (no address typed).
    journeys[0].steps[1].do[1].value: {buyer.name} names the user "buyer", who is not in users.
    journeys[0].steps[1].requires[0]: requires the unknown step "s99".
  ```

- Check a file whose user key starts with a digit. The exit code is 1:

  ```console
  codelead analyst check journeys.json
  ```

  ```output
  journeys.json: 1 problem
    users.1clerk: a user key is a role's name, or no placeholder could name it: a role's name starts with a letter (A to Z), then letters, digits or underscores, such as manager or clerk_2, and "1clerk" does not start with a letter.
  ```

- Check a file that is not JSON:

  ```console
  codelead analyst check bad.json
  ```

  ```output
  bad.json: 1 problem
    (the file): is not JSON: Expected double-quoted property name in JSON at position 35 (line 2 column 1)
  ```

- Check a file that does not exist:

  ```console
  codelead analyst check nothere.json
  ```

  ```output
  nothere.json: 1 problem
    (the file): cannot be read: ENOENT: no such file or directory, open 'nothere.json'
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | No problem. The walk can read the file. |
| `1` | At least one problem, including a file that cannot be read or is not JSON. Also a folder without the pro plan. |

## Files

| File | What `check` does with it |
|---|---|
| `<JOURNEYS>` | Reads it. The check writes nothing. |

## See also

- [From a recording to a specification](../from-recording-to-spec.md)
- [`codelead analyst draft`](analyst-draft.md): it runs the same check.
- [`codelead analyst replay`](analyst-replay.md): walk the file once it passes.
- [Journeys format](../../reference/journeys-format.md)
- [`codelead walk`](../../cli/codelead-walk.md)
- [The Analyst view](../../extension/analyst-view.md): **Check journeys** puts each problem in the Problems panel.
