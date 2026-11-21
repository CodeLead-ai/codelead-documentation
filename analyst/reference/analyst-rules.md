---
title: codelead analyst rules
description: State what the application was seen to enforce, from one or more recordings, each rule marked observed with its recording and step as evidence.
kind: reference
area: analyst
date: 2026-11-20
---

# codelead analyst rules

**Applies to:** CodeLead 0.1 and later

`codelead analyst rules` reads one or more recordings and states what the application was seen to enforce. It writes `rules.observed.json` and `rules.observed.md`. Each rule is marked `observed`, with the recording and the step that show it. The command reads files only: it calls no model and opens no browser.

## Name

`codelead analyst rules` - States the rules that one or more recordings show.

## Synopsis

```console
codelead analyst rules <RECORDINGS>... [--out <FOLDER>]
codelead analyst help rules
```

## Description

The rules come in five kinds, read the same way for every application. Most examples come from the Basic Store sample. The `refusal` and `access` examples come from an application whose pages call a script API, because the store shows neither:

| Kind | What the recording showed | Example statement |
|---|---|---|
| `refusal` | A write refused with 400 to 499, while the page showed a message. | ``The application refuses "Save" on `/Orders/Create` (`POST /api/orders`, 422) when the page says "An order needs at least one line".`` |
| `validation` | Such a refusal whose message names a field that the step filled. A wrong password refused is one too. | ``The application refuses a wrong password on `/Account/Login`: "Log in" (`POST /Account/Login`) is refused (answered 200, the sign-in form shown again) when "Password" holds a password that is not the account's, and the page says "Invalid login attempt.".`` |
| `effect` | An accepted write, after which the page showed new headings or texts. | ``After "Create product" on `/Products/Create` (`POST /Products/Create`, 200), the page shows "Another product already uses this SKU.".`` |
| `access` | A read refused with 401 or 403. Also a screen refused when it was opened: answered 401 or 403, or sent to the sign-in page. | ``The screen `/Reports` needs a role that clerk does not have: `GET /api/reports` was answered 403 to clerk.`` |
| `navigation` | A write answered with a redirect, or an accepted write after which the page went elsewhere. | ``After "Create product" on `/Products/Create` (`POST /Products/Create`, 302), the server redirects to `/Products/Details/{id}`.`` |

The same statement from several steps, in one recording or in several, is one rule with each step as its evidence. When the page shows a value that the step typed, the rule says "what was typed into" the field. So the rule holds for any value.

A screen refused when it was opened gives its `access` rule under the screen that was refused. The rule says what the screen needs, from how it was refused:

<!-- described, not captured -->
```markdown
The screen `/Manage` needs a role that clerk does not have: opening it was answered 403 to clerk.
The screen `/Manage` needs a signed-in user: opening it before clerk signed in sent the page to the sign-in page `/Account/Login` (302).
The screen `/Manage` is refused to clerk as signed in: opening it sent the page to the sign-in page `/Account/Login` (302).
```

`rules.observed.md` groups the rules by screen: the address where the action was made, with each id written `{id}`. It ends with **What was not observed**: the requests that gave no rule. Examples are a write refused with no message on the page, and a server error. A screen that could not be opened (a 404, a server error, no answer) is listed there too, with the kind `navigation`. It is a failure, not a refusal.

A rule holds only as far as the steps that it cites. The Analyst reads no code. See [Limits](../limits.md#what-rules-and-facts-cannot-say) for what the rules cannot say.

## Arguments

- **`<RECORDINGS>...`**

  One or more `recording.json` files to read. A person's recordings and the Explorer's can be read together.

## Options

- **`--out <FOLDER>`**

  Where the files are written. The default is beside the first recording. A recording is cited by its path from this folder.

## Examples

- State the rules of one recording of the Basic Store sample:

  ```console
  codelead analyst rules analyst-recordings/2026-11-20T09-30-00Z/recording.json
  ```

  <!-- described, not captured -->
  ```output
  Observed 3 rules in 1 recording into analyst-recordings/2026-11-20T09-30-00Z/rules.observed.json and analyst-recordings/2026-11-20T09-30-00Z/rules.observed.md.
  By kind: 0 refusal, 1 validation, 0 effect, 0 access, 2 navigation. Not observed: 0 requests that gave no rule (listed at the end of rules.observed.md).
  ```

- State the rules of two recordings together, into a folder of their own:

  ```console
  codelead analyst rules analyst-recordings/2026-11-20T09-30-00Z/recording.json analyst-recordings/2026-11-20T10-15-00Z/recording.json --out docs/store-rules
  ```

  <!-- described, not captured -->
  ```output
  Observed 9 rules in 2 recordings into docs/store-rules/rules.observed.json and docs/store-rules/rules.observed.md.
  By kind: 1 refusal, 1 validation, 2 effect, 2 access, 3 navigation. Not observed: 2 requests that gave no rule (listed at the end of rules.observed.md).
  ```

- Read one rule in `rules.observed.md`, with its evidence:

  <!-- described, not captured -->
  ```markdown
  ## On `/Products/Create` (1)

  - **navigation** `navigation:POST /Products/Create 302@/Products/Create` (observed): After "Create product" on `/Products/Create` (`POST /Products/Create`, 302), the server redirects to `/Products/Details/{id}`.
    - `recording.json:s02` (Create product on /Products/Create; clerk): `POST /Products/Create` 302; the page showed "Desk lamp"
  ```

- Give a file that is not a recording. Nothing is written, and the exit code is 2:

  ```console
  codelead analyst rules analyst-recordings/2026-11-20T09-30-00Z/journeys.json
  ```

  <!-- described, not captured -->
  ```output
  analyst-recordings/2026-11-20T09-30-00Z/journeys.json: schema: expected "codelead.recording/1", found "codelead.journeys/1".
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The rules were written. |
| `1` | No recording was given, or an option is not known. Also a folder without the pro plan. |
| `2` | A recording cannot be read, is not JSON, or is not a `codelead.recording/1` recording. Nothing was written. |

## Files

| File | What `rules` does with it |
|---|---|
| `<RECORDINGS>` | Reads them. |
| `rules.observed.json` | Writes the rules, `codelead.rules.observed/1`, with `recordings`, `rules` and `notObserved`. |
| `rules.observed.md` | Writes the rules by screen, and what was not observed. |

See [Files the Analyst writes](../files.md#rulesobservedjson-and-rulesobservedmd).

## See also

- [Roles, journeys and steps](../concepts.md#rules-and-facts)
- [`codelead analyst facts`](analyst-facts.md): the screens, the server and the users of the same recordings.
- [`codelead analyst record`](analyst-record.md) and [`codelead analyst explore`](analyst-explore.md): make the recordings.
- [Limits](../limits.md)
- [The dossier](../../modernization/dossier.md)
- [The Analyst view](../../extension/analyst-view.md)
