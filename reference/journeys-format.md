---
title: Journeys format
description: The full reference of codelead.journeys/1 - users, identity, clock, journeys, steps, every action and outcome, the placeholders, refusedByDesign, and the rules the walk checks before it runs.
kind: reference
area: reference
date: 2026-11-20
---

# Journeys format

**Applies to:** CodeLead 0.1 and later

A **journeys file** describes paths that people take through an application, step by step, with what each step must show. The [walk](../modernization/walk-and-parity.md) reads it and follows each step on the running application, as a person would. The format is `codelead.journeys/1`. One format and one engine serve CodeLead and the Analyst. This page lists every key and every rule.

> [!NOTE]
> **New to this?** For the ideas behind the format (roles, journeys, steps), read [Roles, journeys and steps](../analyst/concepts.md). For the files that the Analyst writes around a journeys file, read [Files the Analyst writes](../analyst/files.md).

## Where journeys files are

| File | Written by | Walked by |
|---|---|---|
| `docs/codelead/rebuild/journeys.json` in a rebuild | [`codelead rebuild init`](../cli/codelead-rebuild-init.md), merged from three sources; [`codelead rebuild journeys --refresh`](../cli/codelead-rebuild-journeys.md) refreshes its drafted journeys | the rebuild's acceptance walk, [`codelead walk`](../cli/codelead-walk.md) |
| `docs/codelead/rebuild/journeys.draft.json` in the old application | [`codelead model`](../cli/codelead-model.md) | nothing until a person confirms it |
| `docs/codelead/specs/<name>.journeys.json` | [`codelead spec draft`](../cli/codelead-spec.md) | `codelead spec walk`, `/implement --spec` |
| `journeys.json` of a recording | [`codelead analyst draft`](../analyst/reference/analyst-draft.md) | [`codelead analyst replay`](../analyst/reference/analyst-replay.md) |

You can write one by hand too. [`codelead analyst check`](../analyst/reference/analyst-check.md) checks a file, and the walk checks it again before it opens a browser.

## A journeys file

A small file: one user, one journey of three steps. The first step expects the application to refuse a wrong password:

```json
{
  "schema": "codelead.journeys/1",
  "app": "Items fixture",
  "users": {
    "alpha": { "nick": "Alpha" }
  },
  "journeys": [
    {
      "id": "items-fixture",
      "steps": [
        {
          "id": "s01",
          "title": "sign in with a wrong password",
          "user": "alpha",
          "do": [
            { "open": "/" },
            { "fill": { "label": "User name" }, "value": "{alpha.name}" },
            { "fill": { "label": "Password" }, "value": "{alpha.wrongPassword}" },
            { "click": ["Sign in"], "role": "button" }
          ],
          "expect": [
            { "path": "^/$" },
            { "responses": { "method": "POST", "path": "^/api/sign-in$", "status": 401 } },
            { "text": "Sign in" },
            { "text": "Wrong password" }
          ]
        },
        {
          "id": "s02",
          "title": "sign in",
          "user": "alpha",
          "do": [
            { "fill": { "label": "Password" }, "value": "{alpha.password}" },
            { "click": ["Sign in"], "role": "button" }
          ],
          "expect": [
            { "path": "^/items$" },
            { "responses": { "method": "POST", "path": "^/api/sign-in$" } },
            { "text": "Items" }
          ]
        },
        {
          "id": "s03",
          "title": "create an item",
          "user": "alpha",
          "do": [
            { "click": ["New item"], "role": "link" },
            { "fill": { "label": "Title" }, "value": "First item" },
            { "select": { "label": "Kind" }, "value": "Task" },
            { "click": ["Save"], "role": "button" }
          ],
          "expect": [
            { "path": "^/items/[^/]+$" },
            { "responses": { "method": "POST", "path": "^/api/items$" } },
            { "text": "First item" },
            { "text": "Task" }
          ]
        }
      ]
    }
  ]
}
```

This file is the Analyst's test fixture, `packages/analyst/test/golden/journeys.json`, shortened to three steps.

## Top-level keys

| Key | Type | Required | What it holds |
|---|---|---|---|
| `schema` | string | Yes | `codelead.journeys/1`. A file of another schema is not a journeys file. |
| `app` | string | No | The application's name, for people. |
| `identity` | object | No | How the visitors sign in. The default is `registered`. |
| `clock` | object | No | How the walk reads the server's day. |
| `users` | object | Yes | The users of the journeys, by key. |
| `refusedByDesign` | list | No | The reads that the application refuses on purpose. |
| `journeys` | list | Yes | The journeys. At least one step in all. |

Any other key is refused.

## identity

| Key | Values | What it means |
|---|---|---|
| `mode` | `registered` | The default. The walk registers new users through the screens, in every run. |
| | `windows` | The application takes the visitor from Windows authentication (Negotiate or NTLM). |
| | `given` | The application takes the visitor from something that registers nobody through its screens: single sign-on, a directory, an invitation, or its own accounts. |
| `credentials` | `explicit` | Required under `windows` and `given`. Each user's browser answers the server's challenge with its own account and password. Several roles in one walk. |
| | `integrated` | One account only. Its password when the walk is given one, else the session of the machine that the walk runs on. |

Under `windows` and `given`, every journey starts at the root: the first action of its first step is `{ "open": "/" }`.

A password is never written in a journeys file. Under `windows` and `given`, the walk reads each user's password when it runs, where it needs one:

- from the environment variable `CODELEAD_WALK_PASSWORD_<USER>`, where `<USER>` is the user's key in capitals. For the user `user`, it is `CODELEAD_WALK_PASSWORD_USER`;
- else from the credentials file that `CODELEAD_WALK_CREDENTIALS` names, outside the repository. The file maps each account to its password. In a rebuild's run, `run.unattended.credentials` of the [run profile](run-profile.md#rununattended) can name it.

The variable wins over the file. See [Passwords](../analyst/passwords.md).

### What a given identity does, and what it does not

Under `given` and `explicit`, the walk gives each user's browser the account and its password. The browser answers the server's own authentication challenge with them: an answer `401` that asks for an account. The walk registers nobody. The identity does nothing more to sign a user in.

A sign-in form sends no such challenge. So on a site that signs users in with its own form, the identity signs nobody in before a journey. Each journey signs in by its own steps. It opens `/`, then a step types `{<user>.account}` and `{<user>.password}` into the form, and clicks its button.

[`codelead model`](../cli/codelead-model.md) drafts the journeys of such a site in this way. This draft comes from CodeLead's tests, shortened. The old application sent a visitor who opened `/` to its sign-in page, with the address that the visitor came from:

```json
{
  "identity": { "mode": "given", "credentials": "explicit" },
  "users": { "user": { "account": "walk-user-1@example.com" } },
  "journeys": [
    {
      "id": "orders.edit-post",
      "steps": [
        {
          "id": "orders.edit-post-1",
          "title": "Open the application at its root as user (`walk-user-1@example.com`): … where the next step signs in",
          "user": "user",
          "do": [{ "open": "/" }],
          "expect": [{ "path": "^/Account/SignIn\\?(?:[^#]*&)?[Rr][Ee][Tt][Uu][Rr][Nn][Uu][Rr][Ll]=%2[Ff](?:&|$)" }]
        },
        {
          "id": "orders.edit-post-2",
          "title": "Sign in as user on the sign-in page `~/Account/SignIn`: …",
          "user": "user",
          "do": [
            { "fill": { "textbox": 0 }, "value": "{user.account}" },
            { "fill": { "textbox": 1 }, "value": "{user.password}" },
            { "click": ["Sign in"] }
          ],
          "expect": [{ "path": "^/$" }]
        }
      ]
    }
  ]
}
```

- The first step is the visitor's: it opens `/` and expects the sign-in page. Its address carries the return address `/` in `returnUrl`, written `%2f` or `%2F`. The pattern takes both.
- The second step signs in. It expects the page where the old sign-in sent the account: here `/`, the address that the visitor came from.
- The feature's own steps follow, as the signed-in user.
- The account `walk-user-1@example.com` is a placeholder. Replace it with a test account that the application signs in, and give its password in `CODELEAD_WALK_PASSWORD_USER`.

## clock

| Key | Type | What it holds |
|---|---|---|
| `timeZone` | string | The server's time zone, an IANA name such as `Europe/Berlin`, or `UTC`. A [run profile](run-profile.md#clock) or `--time-zone` can say it instead. |
| `dateFormat` | string | How the application writes a date, for `{today}`, such as `dd/MM/yyyy`. |
| `midnightMarginSeconds` | whole number | A journey bound to one day does not start this close to the server's midnight. The default is 300. `0` never waits. |

## users

Each key names a user, and the steps name the user by that key. A key starts with a letter (A to Z), then letters, digits or underscores, such as `alpha` or `clerk_2`. The walk fills in a placeholder such as `{clerk.password}` only for such a name. So every reader of a spec refuses another key: [`codelead walk`](../cli/codelead-walk.md), `codelead spec walk`, the merge of the journeys in `codelead rebuild init`, and [`codelead analyst check`](../analyst/reference/analyst-check.md) and `replay`. The walk's refusal says the rule and the way out:

```output
The journey spec is refused and nothing was walked: The user key "1clerk" is not one a placeholder can name: a user's key starts with a letter (A to Z), then letters, digits or underscores, such as alpha or clerk_2, and "1clerk" does not start with a letter. Rename it in the spec's users and in its steps.
```

| Key | Type | What it holds |
|---|---|---|
| `nick` | string | The display name. The run's stamp is added, unless the value places `{stamp}` itself. |
| `name` | string | The sign-in name of a user that the walk registers. It must hold `{stamp}`, because every walk registers its users anew. |
| `password` | string | The password that the walk registers the user with. The default satisfies ASP.NET Core Identity's default rules. |
| `newPassword` | string | A second password, for a journey that changes it. |
| `location` | `{ latitude, longitude }` | Where the user's browser says it is. Without it, the browser refuses to say. |
| `account` | string | Under `windows` or `given` only: the account that the walk signs in as, such as `STORE\\walk-clerk`. Under `integrated`, `{computer}` stands for the account signed in to the machine. The System Model's draft writes a placeholder, such as `walk-user-1@example.com`, for a person to replace with a test account. |

All of them are optional. A user's own values can hold no placeholder but `{stamp}`.

## journeys

| Key | Type | Required | What it holds |
|---|---|---|---|
| `id` | string | Yes | The journey's id. Unique in the file. |
| `steps` | list | Yes | The steps, at least one. |
| `description` | string | No | What the journey is about, in a person's words, at most 4000 characters. The walk carries it to the report and judges nothing by it. |
| `source` | `{ kind, file }` | No | Where a merged journey came from: `kind` is `model`, `analyst` or `explorer`, and `file` the file it was read from. `rebuild init` writes it. |
| `day` | `one` or `fresh` | No | `one`: the steps must fall on one day of the server's. `fresh`: the same, with an account that has done nothing yet today. Such a journey starts at the root. |
| `clock` | `controlled` | No | The journey needs a server clock that CodeLead sets. It is walked only where the run profile declares how, with consent. |
| `startAt` | `HH:mm` or `HH:mm:ss` | No | With `clock: "controlled"`: the time that the clock is set to before the journey. The default is `23:58`. |

## steps

| Key | Type | Required | What it holds |
|---|---|---|---|
| `id` | string | Yes | The step's id. Unique in the whole file, not only in its journey. |
| `title` | string | Yes | What the step does, in a few words. |
| `user` | string | Yes | A key of `users`. |
| `do` | list of actions | Yes | The actions, in order. An empty list for a step that only looks. |
| `expect` | list of outcomes | Yes | What must hold once the actions are done. At least one. |
| `requires` | list of step ids | No | The steps that must have passed first, each before this one. The default is the previous step of the same journey. `[]` means none. |
| `covers` | list of part ids | No | The parts of a rebuild's plan that the step judges, such as `orders.create:screen`. The plan reads it. The walk does not. Without it, the plan reads both ends of the step: the page where its control is, and the page that it reaches. |
| `note` | string | No | A person's words beside the step, at most 4000 characters. The walk prints it in its report, and judges nothing by it. |

### Actions

| Action | Shape | What the walk does |
|---|---|---|
| open | `{ "open": "/" }` | Opens the application at its root. Only `/`: every other page is reached by clicking. |
| click | `{ "click": ["Save", "Submit"], "role": "button", "inRow": "First item", "optional": true }` | Clicks the first control in sight that goes by one of the names. `role` is `link` or `button`. `inRow` looks in the innermost row that holds the text. `optional` is for a dismissal, and may find nothing. A control folded away in a closed menu is opened first. |
| fill | `{ "fill": { "label": "Title" }, "value": "First item" }` | Types the value into a field in sight, named by exactly one of `label`, `placeholder`, or `textbox` (its position among the text fields, from 0). |
| select | `{ "select": { "label": "Kind" }, "value": "Task" }` | Chooses an option of a drop-down, found as a fill finds its field, or by `selectbox`. The option is the first whose text is the value, else the first whose value is. |
| clickNth | `{ "clickNth": { "selector": ".product", "n": 2 } }` | Clicks the n-th element, from 0, that the selector matches. |
| dragEach | `{ "dragEach": { "items": ".card", "handle": ".grip", "targets": ".slot" } }` | Drags every item onto the next free target. |
| wait | `{ "wait": 500 }` | Waits so many milliseconds. |

The walk looks for a control in the page, then in each of its frames. A step's **own actions** are a click that is not optional, a fill, a select, a clickNth and a dragEach.

### Outcomes

| Outcome | Shape | What must hold |
|---|---|---|
| path | `{ "path": "^/orders$" }` | The page's path matches the regular expression. The query counts only when the pattern holds an escaped `\?`, written `\\?` in JSON. |
| text | `{ "text": "Order saved", "user": "clerk" }` | The page's text, or a frame's, matches the pattern. `user` reads another user's page. |
| notText | `{ "notText": "Error" }` | The page's text does not match. |
| count | `{ "count": { "selector": "tr.order", "min": 1, "max": 20 } }` | How many elements the selector matches. `min` is 1 unless set. |
| link | `{ "link": "Download" }` | A link goes by that name. |
| responses | `{ "responses": "ok" }` | The step's last own action sent a write that the server accepted, and nothing that the step saw was refused. |
| | `{ "responses": { "method": "GET", "path": "^/api/orders$" } }` | That request reached the server during the step, and no answer to it was a refusal. |
| | `{ "responses": { "method": "POST", "path": "^/api/orders$", "status": 409 } }` | That request was refused with exactly that status, from 400 to 499, as the step expects. The step must then assert what the user sees with a `text` outcome. |
| | `{ "responses": { …, "body": { "keys": ["id", "total"], "items": 3 } } }` | Also: the answer's body has those fields, or that many items. |

### Placeholders

| Placeholder | Stands for |
|---|---|
| `{stamp}` | The run's stamp. |
| `{<user>.name}`, `{<user>.nick}` | The user's sign-in name and display name. Under a given identity, `{<user>.name}` is the account. |
| `{<user>.account}` | Under a given identity: the account. |
| `{<user>.password}`, `{<user>.newPassword}` | The user's passwords. Only as a fill's value. Under a given identity, `{<user>.password}` is the password that the walk is given: see [identity](#identity). |
| `{<user>.wrongPassword}` | A value that is certainly not the user's password, for a sign-in that must be refused. Only as a fill's value. |
| `{today}`, `{today:<format>}` | The server's date. |
| `{now:<format>}` | The server's time, for example `{now:HH:mm}`. |

`{today}` and `{now:…}` are read in `text` outcomes only. The report and the step lines write the placeholder, never a password.

## refusedByDesign

A read that the application refuses, redirects or holds open on purpose. Without an entry, a refused read is a finding. A server error is never by design.

| Key | Type | What it holds |
|---|---|---|
| `path` | string, required | A pattern for the request's path. |
| `method` | `GET`, `HEAD` or `OPTIONS` | The default is every read method. |
| `status` | whole number, required | 400 to 499: the refusal. 301, 302, 303, 307 or 308: a redirect. 0: the application holds the read open, such as a long poll. |
| `reason` | string, required | Why, in words. |
| `steps` | list of step ids | The steps where the refusal is expected. The default is every step. |

```json
"refusedByDesign": [
  { "path": "^/api/me$", "method": "GET", "status": 401, "reason": "An anonymous visitor asks who is signed in, and the API says nobody." }
]
```

The report lists these entries and does not count them.

In a rebuild, the acceptance judge also sets this refusal aside by itself, with no entry. The caller-identity route answers 401 to a user who has not signed in yet in the journey. That is the shell's signed-out state. The judge shows it as `Signed out, as expected (not counted): …`. A 401 of that route after the user signed in still counts. See [The walk of a rebuild's part](../modernization/walk-and-parity.md#the-walk-of-a-rebuilds-part).

## The rules that the walk checks

The walk checks the whole file before it opens a browser. It refuses a file that breaks a rule, and walks nothing. The words of some refusals:

- `Journey id "orders" is used twice.` and `Step id "s01" is used twice.`
- `Journey "orders" has no step: a walk of nothing proves nothing.`
- `Step "s03" asserts no outcome: a step that only clicks proves nothing.`
- `Step "s02" names the unknown user "clerk".`
- `Step "s04" requires "s05", which comes after it: a step waits only on steps before it.`
- `Step "s02" uses the unknown placeholder {alpha.email} (known: …).`
- `The user "alpha" has the fixed name "alice": every walk registers its users anew, so the name must hold {stamp}.`
- `Journey "orders" does not begin at the root: under a given identity every journey starts at "/" as its account, its first step's first action { "open": "/" }.`
- A step that expects a refusal and asserts no `text` outcome.
- A step that asserts `responses: "ok"` and has no action of its own.
- A step whose only clicks are optional and that asserts no `path`.

## What a walk reports

Each step ends `pass`, `fail` or `blocked`. A step that did not reach a judgement is blocked, for example because a step that it requires failed. The walk writes its report, `codelead.walk/1`, as `.walk.json` and `.walk.md` beside the spec, or in `docs/codelead/parity/` for a rebuild. See [The walk and parity](../modernization/walk-and-parity.md).

In a rebuild, some requests get no answer at all from the API, at another origin than the page's. The judge names such a finding a cross-origin block. The API's CORS policy refused the web application's origin, or the credentials that its calls send.

## See also

- [Roles, journeys and steps](../analyst/concepts.md)
- [Files the Analyst writes](../analyst/files.md)
- [`codelead walk`](../cli/codelead-walk.md) and [`codelead spec`](../cli/codelead-spec.md)
- [`codelead analyst check`](../analyst/reference/analyst-check.md) and [`codelead analyst replay`](../analyst/reference/analyst-replay.md)
- [The walk and parity](../modernization/walk-and-parity.md)
- [Specification files](spec-files.md)
- [Run profile](run-profile.md)
