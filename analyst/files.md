---
title: Files the Analyst writes
description: Every file and folder that the Analyst and the Explorer write, with its format, what it holds and a short example.
kind: reference
area: analyst
date: 2026-11-20
---

# Files the Analyst writes

**Applies to:** CodeLead 0.1 and later

The Analyst and the Explorer write plain files, in JSON for programs and in Markdown for people. This page lists each file, the command that writes it, its format and a short example. Every JSON file names its format in its `schema` key. No command that writes these files calls a model.

## Where the files go

Each session and each exploration gets one folder, `analyst-recordings/<TIME>/`, in the folder where you run the command. `<TIME>` is the start time in UTC, with dashes in place of colons, for example `analyst-recordings/2026-11-20T09-30-00Z/`. The `--out <FOLDER>` option of `record` and `explore` writes elsewhere.

The commands that read recordings write beside the first recording that they read, or into their own `--out` folder.

A folder of a person's recording:

```text
analyst-recordings/2026-11-20T09-30-00Z/
  recording.json            the recording (codelead.recording/1)
  recording.md              the same, to read
  index.json                the roles and journeys of the session (codelead.recording-index/1)
  steps/s01.png …           the page when each step ended
  journeys.json             after draft: the specification (codelead.journeys/1)
  journeys.md               after draft: what to confirm, and each step in words
  journeys.confirmed.json   after Confirm in VS Code: what a person confirmed
  rules.observed.json, .md  after rules (codelead.rules.observed/1)
  facts.observed.json, .md  after facts (codelead.facts.observed/1)
```

A folder of an exploration:

```text
analyst-recordings/2026-11-20T09-15-00Z/
  explore-clerk/            one folder for each role
    recording.json, recording.md, steps/
    journeys.json, journeys.md
  sitemap.json, sitemap.md  the screens (codelead.sitemap.observed/1)
  forms.json, forms.md      the forms (codelead.forms.observed/1)
  stories.json, stories.md  the user stories (codelead.stories.observed/1)
  rules.observed.json, .md  the rules of all the roles' recordings
  facts.observed.json, .md  the facts of all the roles' recordings
```

`--no-derive` leaves out each role's `journeys.*`, and the rules and the facts. An exploration writes no `index.json`.

## Summary

| File | Written by | Format |
|---|---|---|
| `recording.json` | [`record`](reference/analyst-record.md), [`explore`](reference/analyst-explore.md) | `codelead.recording/1` |
| `recording.md` | `record`, `explore` | Markdown |
| `index.json` | `record` | `codelead.recording-index/1` |
| `steps/s01.png` | `record`, `explore` | PNG |
| `journeys.json` | [`draft`](reference/analyst-draft.md), `explore` | `codelead.journeys/1` |
| `journeys.md` | `draft`, `explore` | Markdown |
| `journeys.confirmed.json` | The Confirm tab of the [Analyst view](../extension/analyst-view.md) | JSON, by item |
| `rules.observed.json`, `.md` | [`rules`](reference/analyst-rules.md), `explore` | `codelead.rules.observed/1` |
| `facts.observed.json`, `.md` | [`facts`](reference/analyst-facts.md), `explore` | `codelead.facts.observed/1` |
| `sitemap.json`, `.md` | `explore` | `codelead.sitemap.observed/1` |
| `forms.json`, `.md` | `explore` | `codelead.forms.observed/1` |
| `stories.json`, `.md` | `explore` | `codelead.stories.observed/1` |

[`check`](reference/analyst-check.md) and [`replay`](reference/analyst-replay.md) write nothing.

## recording.json

One session on one application, as one or more users, cut into steps. The recorder writes it again after every step, so a crash loses one step at most.

<!-- described, not captured -->
```json
{
  "schema": "codelead.recording/1",
  "app": "Basic Store",
  "base": "http://localhost:5080/",
  "api": "http://localhost:5080",
  "startedAt": "2026-11-20T09:30:00.120Z",
  "endedAt": "2026-11-20T09:41:12.004Z",
  "users": { "clerk": { "nick": "Clerk" } },
  "steps": [
    {
      "id": "s01",
      "title": "New product on /",
      "user": "clerk",
      "journey": "create a product",
      "startedAt": "2026-11-20T09:30:41.310Z",
      "endedAt": "2026-11-20T09:30:43.002Z",
      "urlBefore": "/",
      "urlAfter": "/Products/Create",
      "titleAfter": "New product - Basic Store",
      "actions": [
        { "kind": "click", "at": "2026-11-20T09:30:42.100Z", "control": { "role": "link", "name": "New product", "selector": "main > div > div:nth-of-type(2) > a:nth-of-type(3)" } },
        { "kind": "navigate", "at": "2026-11-20T09:30:42.380Z", "url": "/Products/Create" }
      ],
      "requests": [],
      "seen": { "headings": ["New product"], "alerts": [], "dialogs": [] },
      "screenshot": "steps/s01.png"
    }
  ]
}
```

| Key | What it holds |
|---|---|
| `base`, `api` | The application's address, and the origin of its server. |
| `identity`, `credentials` | `"given"` for given accounts. `"integrated"` beside it for Windows sign-in. Absent for new users. |
| `users` | Each role by its key, with its `account` under given accounts. |
| `journeys` | Optional. The context that you wrote for a journey, by its name. |
| `steps[].actions` | `navigate`, `fill`, `click`, `submit` or `choose`. A control is named by its role and name, or a field by its label, placeholder or place. `selector` is for a person reading. |
| `steps[].requests` | The method, the path, the status (0 for no answer), `read` or `write`, and the first 200 characters of a refusal. Never a body, a header or a cookie. |
| `steps[].seen` | The headings (h1 to h3), the alerts and the dialogs when the step ended. |
| `steps[].expectations` | Optional. What you marked with **Expect this**: `text` or `row`. |
| `steps[].context` | Optional. What you wrote about the step, at most 4,000 characters. |

A masked value looks like this. The value is the placeholder, never what was typed:

```json
{ "kind": "fill", "control": { "label": "Password", "selector": "#Password" }, "value": "{clerk.password}", "masked": true, "password": "own" }
```

A `navigate` action carries `status` only when the application did not show the address asked for. The status is 401 or 403 for a refusal, another status from 400 to 599 for a failure, and 0 for no answer. A redirect to the sign-in page also gets `sentToSignIn: true` beside `sentTo`, and `status` is then the redirect's status. Every other `navigate` has neither key:

<!-- described, not captured -->
```json
{ "kind": "navigate", "url": "/Manage", "status": 403 }
{ "kind": "navigate", "url": "/Manage", "sentTo": "/Account/Login", "sentToSignIn": true, "status": 302 }
{ "kind": "navigate", "url": "/Products/Details/99", "status": 404 }
```

## recording.md

The same recording, for a person. It lists each step with its user, its addresses, its actions in words, its requests, what was seen and its screenshot:

<!-- described, not captured -->
```markdown
## s01 · New product on /

- User: clerk
- Address: / → /Products/Create (page title "New product - Basic Store")
- From 2026-11-20T09:30:41.310Z to 2026-11-20T09:30:43.002Z

Actions:

1. clicked link "New product" (`main > div > div:nth-of-type(2) > a:nth-of-type(3)`)
2. went to /Products/Create

Requests:

- none
```

When the application refused an address or could not show it, the action says so after the address:

<!-- described, not captured -->
```markdown
7. went to /Manage (refused: answered 403)
7. went to /Manage (refused: sent to the sign-in page /Account/Login, 302)
7. went to /Products/Details/99 (failed: answered 404)
7. went to /Orders (failed: no answer)
```

## index.json

What the folder holds, for `record --continue`: the application, the sign-in mode, the roles, and each journey with its status and its steps. It holds no typed value and no password.

<!-- described, not captured -->
```json
{
  "schema": "codelead.recording-index/1",
  "app": "Basic Store",
  "base": "http://localhost:5080/",
  "api": "http://localhost:5080",
  "signIn": "form",
  "keepQuery": false,
  "roles": ["clerk"],
  "journeys": [{ "name": "create a product", "role": "clerk", "status": "ended", "steps": ["s01", "s02"] }]
}
```

A journey's `status` is `recording`, `paused`, `replaying`, `stopped` or `ended`. When the index and the recording disagree, the recording wins.

## steps/

One PNG for each step, named by the step's id: `steps/s01.png`, `steps/s02.png`. Each shows the page when the step ended. The recorder hides its own bar while it takes the picture. An edit of the steps moves and renames these files, so do not rename them by hand.

## journeys.json

The specification, in CodeLead's format `codelead.journeys/1`. The walk reads it. See [Journeys format](../reference/journeys-format.md) for every key. A short file that the check accepts:

```json
{
  "schema": "codelead.journeys/1",
  "app": "Basic Store",
  "users": { "clerk": {} },
  "journeys": [
    {
      "id": "create-a-product",
      "steps": [
        {
          "id": "s01",
          "title": "New product on /",
          "user": "clerk",
          "do": [{ "open": "/" }, { "click": ["New product"], "role": "link" }],
          "expect": [{ "path": "^/Products/Create$" }, { "text": "New product" }]
        }
      ]
    }
  ]
}
```

## journeys.md

The draft in words. It starts with **To confirm**, in four groups, then shows each step as **Do** and **Expect** lines. See [Read what to confirm](from-recording-to-spec.md#read-what-to-confirm).

## journeys.confirmed.json

The Confirm tab of the Analyst view writes it beside the draft. No command of the Analyst writes or reads it. Each item of `journeys.md` is kept by a key: its step, its kind and its first ten words.

<!-- described, not captured -->
```json
{
  "s01 · refused · `POST ^/Account/Login$` was answered 200 and the page showed the sign-in": {
    "confirmed": true,
    "note": "the store refuses a wrong password on purpose",
    "says": "`POST ^/Account/Login$` was answered 200 and the page showed the sign-in form again (\"Invalid login attempt.\"): … Confirm it is on purpose."
  }
}
```

A new draft keeps what was confirmed, wherever the item is still there.

## rules.observed.json and rules.observed.md

What the application was seen to enforce. Each rule has a kind (`refusal`, `validation`, `effect`, `access` or `navigation`), a screen, a statement, the status `observed`, and every step that shows it. `notObserved` lists the requests that gave no rule.

<!-- described, not captured -->
```json
{
  "schema": "codelead.rules.observed/1",
  "recordings": [{ "file": "recording.json", "app": "Basic Store", "base": "http://localhost:5080/", "startedAt": "…", "endedAt": "…", "steps": 9 }],
  "rules": [
    {
      "id": "validation:POST /Account/Login 200:Password@/Account/Login",
      "kind": "validation",
      "screen": "/Account/Login",
      "statement": "The application refuses a wrong password on `/Account/Login`: \"Log in\" (`POST /Account/Login`) is refused (answered 200, the sign-in form shown again) when \"Password\" holds a password that is not the account's, and the page says \"Invalid login attempt.\".",
      "observed": [
        {
          "recording": "recording.json", "step": "s01", "title": "Log in on /Account/Login", "user": "clerk",
          "request": { "method": "POST", "path": "/Account/Login", "status": 200, "refusedSignIn": true },
          "shown": ["Invalid login attempt."],
          "typed": { "field": "Password", "value": "{clerk.password}", "password": "other" }
        }
      ],
      "status": "observed"
    }
  ],
  "notObserved": []
}
```

`rules.observed.md` groups the rules by screen, and ends with **What was not observed**. A recording is cited by its path from the output folder, so the links in the Markdown open it.

A screen that the application refused when it was opened gives an `access` rule under that screen. Its request is the page's own document, marked `"navigation": true`, with `sentTo` for a redirect to the sign-in page. Its observation says `signedIn`: whether the user had signed in before. In this application, only an administrator may open the page **Manage**:

<!-- described, not captured -->
```markdown
## On `/Manage` (1)

- **access** `access:GET /Manage 403@/Manage` (observed): The screen `/Manage` needs a role that clerk does not have: opening it was answered 403 to clerk.
  - `explore-clerk/recording.json:s08` (reach /Manage by the link "Manage": refused, answered 403 (Not allowed); clerk): `GET /Manage` 403 (the screen opened); the page showed "Not allowed" and "Only the administrator may manage."
```

A screen that could not be opened (a 404, a server error, no answer) gives no rule. It is listed under **What was not observed**, with the kind `navigation`.

## facts.observed.json and facts.observed.md

What the recordings show of the application, for its dossier: `screens`, `requests` (the server's surface), `users`, `journeys`, and `identity` (how the users came to exist). Each entry has the status `observed` and the steps that it was seen in. The Markdown reads like the System Model's `model.md`, with "observed" where the model says "read".

<!-- described, not captured -->
```markdown
## The server's surface (2)

| Request | Reads or writes | Statuses seen | Called from | Observed |
|---|---|---|---|---|
| `POST /Account/Login` | write | 200, 302 | `/Account/Login` | `recording.json:s01`, `recording.json:s02` |
| `POST /Products/Create` | write | 302 | `/Products/Create` | `recording.json:s04` |
```

A screen that was refused to a user when it was opened is in the user's `refusedScreens`, and in the screen's `refusedTo`. That user is not one of the screen's `users`, the users who reached it. A screen that a user could not open is in `failedScreens` and `failed`, and is no refusal. The Markdown says both in words:

<!-- described, not captured -->
```markdown
- `/Manage` (observed): title "Manage"; headings "Manage"; reached by admin; refused to clerk (answered 403, the page showing "Not allowed"; `explore-clerk/recording.json:s08`) (`explore-admin/recording.json:s08`, `explore-clerk/recording.json:s08`)
- **clerk** (account `store.clerk`) (observed): reached `/`, `/Clients`, `/Products`, `/Orders`; refused the screen `/Manage` (answered 403; `explore-clerk/recording.json:s08`) (9 steps: …)
```

## sitemap.json and sitemap.md

Every screen that the Explorer reached. A screen is an address and its headings. For each screen, it lists the roles that reached it and the roles that it was refused to. It also gives the shortest way to it, its forms, its downloads and where its controls lead. Then the roles and how each signed in, the findings, what the limits left unvisited, and what the crawl never follows.

The JSON's keys are `schema`, `base`, `limits`, `keepQuery`, `roles`, `screens`, `findings`, `notVisited` and `notFollowed`. One role, as the JSON writes it:

```json
{ "role": "clerk", "identity": "given", "account": "store.clerk", "recording": "explore-clerk/recording.json", "signIn": { "step": "s01", "accepted": true }, "screens": 9, "refused": 1 }
```

## forms.json and forms.md

Every form on the screens, found without sending it. For each form: its fields in order, its button, and its method and address as the form declares them. Then its frame, and the screens where each role saw it. A form that the Explorer never sends says why:

<!-- described, not captured -->
```markdown
### form:/Products/Create "Create product"

- On: `/Products/Create` (New product), seen by clerk (`explore-clerk/recording.json:s06`)
- Sent by: "Create product"; POST `/Products/Create`
- Fields:
  1. "SKU", text
  2. "Name", text
  3. "Unit price", number
  4. "Description", textarea

### form:/Account/Login "Log in"

- On: `/Account/Login` (Log in), seen by clerk (`explore-clerk/recording.json:s02`)
- Sent by: "Log in"; POST `/Account/Login`
- Never sent by the explorer: it has a password field: the explorer never sends a form that asks for a password
- Fields:
  1. "Email", text
  2. "Password", password
```

With `--try-forms`, each form that was sent has a list, **The explorer's tries**. Each line names the try: sent empty, or sent with values the explorer made up. Then come the role and its step, the writes with their statuses, what the page said, and the address after.

## stories.json and stories.md

User stories, one for each role and each screen that it reached. "Can see" lines name headings, tables with their columns, and lists. "Can do" lines name forms to send, controls that lead elsewhere, and downloads. A refused screen is a "cannot reach" line. Each story cites its step and its screen. They are stories to confirm, not requirements read from code.

<!-- described, not captured -->
```markdown
### On `/Products/Create` (New product)

As clerk, I can see and do the following here.

- I can see the heading "New product"
- I can fill in "SKU", "Name", "Unit price" and "Description", and send "Create product"
- I can go to /Products (Products) by the link "Products"

Observed: `explore-clerk/recording.json:s06`; the screen `screen:/Products/Create (New product)` in the site map.
```

## What the Analyst reads

| File | Read by | What for |
|---|---|---|
| `.env` in the current folder | `codelead analyst`, when it starts | The variables `CODELEAD_WALK_PASSWORD_<ROLE>` that the Settings page wrote. A variable set in the terminal wins. |
| The file that `CODELEAD_WALK_CREDENTIALS` names | `explore`, `replay` | The passwords of given accounts, by account, when a role's variable is not set. It must be outside the folder where the command runs. |
| The licence of the folder | `codelead analyst`, before every subcommand | The pro plan. See [Licence and plans](../concepts/licence-and-plans.md). |

The Analyst writes no evidence record and nothing under `.codelead/`. The modernization program reads its files: the Observe stage counts each `analyst-recordings/<TIME>/journeys.json`, and [`codelead rebuild init`](../cli/codelead-rebuild-init.md) reads those files and each exploration's `stories.json`.

## See also

- [Roles, journeys and steps](concepts.md)
- [Journeys format](../reference/journeys-format.md)
- [Passwords](passwords.md)
- [The Analyst view](../extension/analyst-view.md) and [the Explorer view](../extension/explorer-view.md)
- [Files and folders](../reference/files-and-folders.md)
- [`codelead analyst`](../cli/codelead-analyst.md)
