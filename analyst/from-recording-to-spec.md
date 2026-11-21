---
title: From a recording to a specification
description: Draft a journeys file from a recording, read what to confirm, check it, replay it on the old application, and know how the modernization program and spec-based validation use it.
kind: how-to
area: analyst
date: 2026-11-20
---

# From a recording to a specification

This guide turns a recording into a **journeys file**: a specification that CodeLead's walk can replay. You draft it, read what the draft asks you to confirm, check it, and replay it on the old application. A journeys file that passes this way is the yardstick that a rebuilt application is measured against.

**In short**
- [`codelead analyst draft`](reference/analyst-draft.md) writes `journeys.json`, the specification, and `journeys.md`, the list of what to confirm.
- [`codelead analyst check`](reference/analyst-check.md) says whether the walk can read it. Each problem names its place in the file.
- [`codelead analyst replay`](reference/analyst-replay.md) walks it on the running application. Every step must pass.
- The modernization program merges the journeys into the rebuild's specification. A draft counts when the check passes.
- None of these commands calls a model. The same recording always gives the same draft.

## Before you begin

- **A recording.** Make one with [Record a journey](record-a-journey.md), or use a role's recording from [Explore an application](explore-an-application.md). Each one is a `recording.json` in a folder under `analyst-recordings/`.
- **The old application runs**, for the replay. For the Basic Store sample, at `http://localhost:5080/`.
- **The passwords are set**, when the recording used given accounts. See [Passwords](passwords.md).

> [!NOTE]
> **New to this?** A draft is the Analyst's best reading of what you did. It is not yet a specification that you trust. You make it one when you read its list, fix what it asks, and see every step pass on the old application.

## Draft the journeys

Run the draft on the recording:

```console
codelead analyst draft analyst-recordings/2026-11-20T09-30-00Z/recording.json
```

The draft writes `journeys.json` and `journeys.md` beside the recording, then checks what it wrote:

<!-- described, not captured -->
```output
Drafted 9 steps in 4 journeys into analyst-recordings/2026-11-20T09-30-00Z/journeys.json and analyst-recordings/2026-11-20T09-30-00Z/journeys.md.
To confirm (journeys.md): 0 refused, 0 secret-looking, 0 with nothing to assert, 1 other.
analyst-recordings/2026-11-20T09-30-00Z/journeys.json: no problems; CodeLead can walk it.
```

The draft makes one journey for each journey that you recorded, in the same order. Each recorded action becomes an action of the walk:

| Recorded | In the journeys file |
|---|---|
| A click on a button or a link | `{ "click": ["Create product"], "role": "button" }`, by its name and role |
| A value typed into a field | `{ "fill": { "label": "SKU" }, "value": "ACC-1001" }`, by the field's label |
| An option chosen in a drop-down | `{ "select": { "label": "Status" }, "value": "Cancelled" }`, by the option's text |
| An address typed | `{ "open": "/" }`. The walk opens only the first page, and clicks its way from there. |

Each step expects four things, in this order. First the address that it ended on, then the writes that the server accepted or refused. Then the headings and alerts that the page showed, and last what you marked with **Expect this**.

In VS Code, right-click the recording in the [Analyst view](../extension/analyst-view.md) and select **Draft from a recording**.

## Read what to confirm

`journeys.md` starts with a list, **To confirm**, in four groups:

| Group | What it lists | What you do |
|---|---|---|
| **Refused by design?** | Each request that the server refused with 400 to 499, and each sign-in refused with its form shown again. | Confirm that the refusal is the application's rule, not an accident of the recording. |
| **Values that look like secrets** | A wrong password, drafted as `{clerk.wrongPassword}`. A masked value in a field that is not a password. A value that looks like a token, an e-mail address or a long number. | Confirm the placeholder, or replace the value. |
| **Nothing to assert: confirm by eye** | Steps that check only the address that they end on. | Add what the page must show. |
| **Other things to confirm** | An address that was typed instead of reached by a click, a control with no name, a server error, a query that was dropped. | Fix the journeys file, or record that step again. |

Here is the list of a Basic Store recording whose first journey types a wrong password, then the right one:

<!-- described, not captured -->
```markdown
## To confirm

### Refused by design?

- s01: `POST ^/Account/Login$` was answered 200 and the page showed the sign-in form again ("Invalid login attempt."): a sign-in refused with no refusal status, as the recording says. CodeLead's walk takes a 200 as accepted, and a step can expect only a refusal from 400 to 499, so the draft declares neither: s01 expects the address it ended on and what the page said. Confirm it is on purpose.

### Values that look like secrets

- s01: types a password that was not the account's into "Password" (the recording says so: it is not a password clerk signed in with): the draft types `{clerk.wrongPassword}` there, never what was typed, so the walk types a wrong password where the person did (a value it builds to differ from clerk's password). Confirm that is meant.

### Nothing to assert: confirm by eye

None.

### Other things to confirm

None.
```

Under the list, `journeys.md` shows each step in plain words:

<!-- described, not captured -->
```markdown
### s02 · Log in on /Account/Login (clerk)

- Do: fill "Password" with {clerk.password}; click button "Log in"
- Expect: the address matches `^/$`; the server accepts `POST ^/Account/Login$`; the page shows `Dashboard`
```

> [!TIP]
> **For experienced users:** a journey always starts from the first page. A recording that started a journey elsewhere gets a note in **Other things to confirm**. Record from the first page, or add `{ "open": "/" }` and the clicks that lead there.

## Check the journeys file

The check reads the file as the walk reads it before it runs anything. It needs no browser and no application:

```console
codelead analyst check analyst-recordings/2026-11-20T09-30-00Z/journeys.json
```

A file with no problem says how much it holds:

```output
analyst-recordings/2026-11-20T09-30-00Z/journeys.json: no problems (1 journey, 2 steps, 1 user).
```

The exit code is 0. The draft runs the same check, so a fresh draft normally passes. Run the check again after each change that you make by hand.

## Read a failed check

A check that fails lists every problem. Each line starts with the place in the file, as a path into the JSON:

```console
codelead analyst check analyst-recordings/2026-11-20T10-00-00Z/journeys.json
```

```output
analyst-recordings/2026-11-20T10-00-00Z/journeys.json: 3 problems
  journeys[0].steps[0].do[0].open: opens "/Products/Create": a walk starts at "/" and reaches every other page by clicking (no address typed).
  journeys[0].steps[1].do[1].value: {buyer.name} names the user "buyer", who is not in users.
  journeys[0].steps[1].requires[0]: requires the unknown step "s99".
```

The exit code is 1. Read `journeys[0].steps[1].do[1]` as: the first journey, its second step, its second action. Fix each line:

| Problem | Fix |
|---|---|
| `opens "/Products/Create"` | Replace the open with `{ "open": "/" }` and the clicks that reach the page, for example `{ "click": ["New product"], "role": "link" }`. |
| `names the user "buyer", who is not in users` | Use a user that `users` declares, for example `{clerk.name}`, or add the user. |
| `requires the unknown step "s99"` | Name an earlier step of the same file, or remove `requires`. |

A file that is not JSON at all gives one problem for the whole file:

```console
codelead analyst check bad.json
```

```output
bad.json: 1 problem
  (the file): is not JSON: Expected double-quoted property name in JSON at position 35 (line 2 column 1)
```

In VS Code, **Check journeys** puts each problem in the **Problems** panel, at its place in the file. See [Messages and what to do](../reference/messages.md).

## Replay on the old application

The replay walks the journeys file in a real browser, as a new user would. It starts at the first page, reaches every page by clicking, and judges each step:

```console
codelead analyst replay analyst-recordings/2026-11-20T09-30-00Z/journeys.json --base http://localhost:5080/
```

<!-- described, not captured -->
```output
PASS    create-a-product · New product on / — clicked "New product"
PASS    create-a-product · Create product on /Products/Create — clicked "Create product"

2 steps: 2 pass, 0 fail, 0 blocked. 0 findings.
```

The exit code is 0 only when every step passed and there was no finding. A **finding** is something that no step expected, such as a refused request or an error on a page.

When the application does not run, the first step fails and every later step is **blocked**:

<!-- described, not captured -->
```output
FAIL    create-a-product · New product on / — the page threw: page.goto: net::ERR_CONNECTION_REFUSED at http://localhost:5080/
BLOCKED create-a-product · Create product on /Products/Create — waits on s01

2 steps: 0 pass, 1 fail, 1 blocked. 0 findings.
```

When the recording used given accounts and a password is missing, nothing is walked, and the exit code is 2:

```output
Nothing was walked: the walk signs in as the accounts it is given, each with its password, and has none for clerk (store.clerk): set CODELEAD_WALK_PASSWORD_CLERK, or name with CODELEAD_WALK_CREDENTIALS a file outside the repository that maps each account to its password.
```

> [!WARNING]
> A replay does what the journeys say. A journey that creates a product creates one again on every replay. Replay on a copy of the application, or on one whose database you can reset.

## Confirm the journeys

1. Read every item of **To confirm** in `journeys.md`.
2. Change `journeys.json` where an item says so. Add outcomes to steps that check only an address.
3. Check the file again, then replay it again.
4. In VS Code, select **Confirm** on the draft. The Confirm tab shows each item with a box and a note. Your choices go to `journeys.confirmed.json` beside the draft.

The journeys file is confirmed when you trust each item and every step passes on the old application.

> [!NOTE]
> **For team leads:** `journeys.confirmed.json` records what a person confirmed, item by item, with a note. The rebuild reads `journeys.json` itself. So a change that you decide on goes into `journeys.json`, not only into the Confirm tab.

## How CodeLead uses the specification

### In the modernization program

The [eight stages](../modernization/the-eight-stages.md) use the journeys in two places:

- **Observe**: the stage waits for your recordings. It counts each folder `analyst-recordings/<time>/` that holds a `journeys.json`. So draft each recording beside it, which is the default.
- **Rebuild**: [`codelead rebuild init`](../cli/codelead-rebuild-init.md) merges the rebuild's journeys, `docs/codelead/rebuild/journeys.json`, from three sources: your recordings first, then the System Model's draft, then the Explorer's stories. When two journeys end in the same place and expect the same thing, it keeps the first.

A recording's journeys count only when the walk's check accepts the whole file. A file that it refuses is left out whole, with the walk's words. So run `codelead analyst check` until it says `no problems`.

Then the walk measures the rebuild against the same journeys. See [The walk and parity](../modernization/walk-and-parity.md) and [The rebuild](../modernization/rebuild.md).

### In spec-based validation

- **Walk your journeys on any application.** [`codelead walk`](../cli/codelead-walk.md) walks a journeys file with `--journeys <FILE> --no-write`, as the replay does. Use it on a new application that must behave like the old one.
- **Find what no spec covers.** [`codelead spec coverage`](../cli/codelead-spec.md) runs the Explorer, or reads an exploration's `sitemap.json` and `forms.json`. It says which screens and forms no spec names.

## Check that it worked

- `codelead analyst check` says `no problems`, and its exit code is 0.
- `codelead analyst replay` on the old application ends with `0 fail, 0 blocked. 0 findings.`
- Every item of **To confirm** is read, and fixed or confirmed.
- In the modernization program, `codelead modernize continue` counts your recording: `Record the journeys: done (1 recording drafted).`

## Details

- **`--out <FOLDER>`** writes the draft elsewhere. The Observe stage and `rebuild init` look only beside the recordings, so keep the default for the program.
- **Integrated sign-in**: a recording made with Windows sign-in drafts `"identity": { "mode": "given", "credentials": "integrated" }`, one user with its account, and no sign-in step.
- **Contexts**: what you wrote in the context boxes goes into `journeys.md`. It goes into `journeys.json` only when the walk accepts the keys `description` and `note`. The draft asks the walk each time.
- **Old recordings**: a recording made before journeys existed drafts one journey from all its steps.
- **The check does not test** whether the passwords are set, or whether the server's time zone is declared. The walk decides both when it runs.

## See also

- [`codelead analyst draft`](reference/analyst-draft.md), [`check`](reference/analyst-check.md) and [`replay`](reference/analyst-replay.md)
- [Roles, journeys and steps](concepts.md)
- [Journeys format](../reference/journeys-format.md)
- [The walk and parity](../modernization/walk-and-parity.md)
- [`codelead rebuild init`](../cli/codelead-rebuild-init.md)
- [The Analyst view](../extension/analyst-view.md)
- [Limits](limits.md)
