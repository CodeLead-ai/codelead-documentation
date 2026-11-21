---
title: Roles, journeys and steps
description: The words of the Analyst and the Explorer - recordings, roles, journeys, steps, the journeys file, checks, replays, and the rules and facts that each cite their evidence.
kind: concept
area: analyst
date: 2026-11-20
---

# Roles, journeys and steps

The [Analyst](index.md) turns what happens on an application's screens into files that CodeLead can check and replay. This page explains each word that the Analyst and the Explorer use, and how one leads to the next. It also says which parts the Analyst observed and which parts it assumed.

**In short**
- A **recording** keeps what one session saw: the steps, the actions, the requests and a screenshot for each step.
- A **journey** is one task that a role does from the first page. The recorder cuts it into **steps** by itself.
- A **journeys file** (`codelead.journeys/1`) is the specification that [the walk](../modernization/walk-and-parity.md) replays. The Analyst drafts it from a recording, and you confirm it.
- **Rules** and **facts** are statements about the application. Each one cites the recording and the step that show it.

## Why these words exist

An old application often has no written specification. Its behaviour lives in its screens and in the heads of its users. The Analyst writes that behaviour down in a form that a program can check.

Each word on this page is one link in a chain:

```text
a person uses the application        the Explorer clicks through it
            |                                    |
            v                                    v
        recording  (codelead.recording/1, one per session or per role)
            |
            +--> draft --> journeys file (codelead.journeys/1) --> check --> replay with the walk
            |
            +--> rules  (rules.observed.json)
            +--> facts  (facts.observed.json)
```

> [!NOTE]
> **New to this?** A *specification* says what an application must do. Here it is a file of journeys: what a person does, step by step, and what the screen must show after each step.

## How it works

### Recordings and sessions

A **recording** is everything that one session saw on one application. It is the file `recording.json`, in the format `codelead.recording/1`, with `recording.md` beside it and a screenshot for each step under `steps/`.

Two commands write recordings:

- [`codelead analyst record`](reference/analyst-record.md) writes one recording while a person uses the application. See [Record a journey](record-a-journey.md).
- [`codelead analyst explore`](reference/analyst-explore.md) writes one recording for each role. No person drives. See [Explore an application](explore-an-application.md).

Every other command reads recordings. They read a person's recording and the Explorer's recording in the same way.

### Roles and accounts

A **role** is a kind of user of the application, such as a clerk or an administrator. A role has a key, such as `clerk`. A key starts with a letter (A to Z, a to z), then letters, digits or underscores. Each role acts in a browser window of its own, with its own cookies.

> [!IMPORTANT]
> In the Analyst, a role is a kind of *user*. It is not a model role (a kind of work for a model). See [Models and roles](../concepts/models-and-roles.md) for the other meaning.

How the roles sign in is the **identity** of the recording. There are three kinds:

| Identity | What it means | Option |
|---|---|---|
| New users | The application's users register through the screens. A replay registers new users with values of its own. | None (the default) |
| Given accounts | The users exist already, for example in a directory. Each role signs in as an **account**, by its sign-in name. | `--given-accounts` |
| Integrated | Windows authentication. The browser signs in by itself, as one account. There is one role. | `--integrated <account>` |

A password is never part of a recording. See [Passwords](passwords.md).

### Journeys and steps

A **journey** is one task that a role does from start to end, with a name that you give it, such as "create a product". Every journey starts from the application's first page, because the walk starts every journey there.

A **step** is a part of a journey where something observable happens. You do not mark the steps. The recorder cuts a journey into steps when one of these happens:

1. The page goes to another address.
2. A write that you sent reaches the server and is answered. A write is a `POST`, `PUT`, `PATCH` or `DELETE` request.
3. A dialog of the page opens or closes.
4. A form is sent.

Typing and choices between two of these points join the next step. So "fill the SKU, fill the name, select **Create product**" is one step. Each step gets a title from its action, such as `Create product on /Products/Create`. You can rename it.

**Expect this** marks something that the page shows and that must be true for the journey to count. The draft makes each mark a text outcome of its step. A field's value is never taken, and a password field least of all.

A **context** is a short text in your own words: what a journey is for, or what a step needs. It is optional, and it holds at most 4,000 characters.

### The journeys file

The **journeys file**, `journeys.json`, is a journey specification in CodeLead's format `codelead.journeys/1`. The walk reads it. See [Journeys format](../reference/journeys-format.md).

[`codelead analyst draft`](reference/analyst-draft.md) writes it from a recording. Each recorded action becomes an action of the walk, and each step gets **outcomes**: what must be true after it. Here is one step of a recording of the Basic Store sample, then the step that the draft makes of it:

<!-- described, not captured -->
```json
{
  "id": "s02",
  "title": "Create product on /Products/Create",
  "user": "clerk",
  "journey": "create a product",
  "urlBefore": "/Products/Create",
  "urlAfter": "/Products/Details/7",
  "actions": [
    { "kind": "fill", "control": { "label": "SKU", "selector": "#Sku" }, "value": "ACC-1001" },
    { "kind": "fill", "control": { "label": "Name", "selector": "#Name" }, "value": "Desk lamp" },
    { "kind": "click", "control": { "role": "button", "name": "Create product" } },
    { "kind": "navigate", "url": "/Products/Details/7" }
  ],
  "requests": [
    { "method": "POST", "path": "/Products/Create", "status": 302, "kind": "write", "form": true }
  ],
  "seen": { "headings": ["Desk lamp"], "alerts": [], "dialogs": [] },
  "expectations": [{ "kind": "text", "text": "Product \"Desk lamp\" was created." }],
  "screenshot": "steps/s02.png"
}
```

```json
{
  "id": "s02",
  "title": "Create product on /Products/Create",
  "user": "clerk",
  "do": [
    { "fill": { "label": "SKU" }, "value": "ACC-1001" },
    { "fill": { "label": "Name" }, "value": "Desk lamp" },
    { "click": ["Create product"], "role": "button" }
  ],
  "expect": [
    { "path": "^/Products/Details/[^/]+$" },
    { "responses": { "method": "POST", "path": "^/Products/Create$" } },
    { "text": "Desk lamp" },
    { "text": "Product \"Desk lamp\" was created." }
  ]
}
```

The draft also writes `journeys.md`. It lists what you must **confirm** before you trust the draft. The list has four groups: refused by design, values that look like secrets, steps with nothing to assert, and other things. See [From a recording to a specification](from-recording-to-spec.md).

### Checks and replays

A **check** reads a journeys file against the format, as the walk reads it before it runs anything. It reports every problem with its place in the file. [`codelead analyst check`](reference/analyst-check.md) runs it. It needs no browser and no application.

A **replay** walks a journeys file against the running application. It uses CodeLead's walk: it starts at the first page, reaches every page by clicking, and marks each step **pass**, **fail** or **blocked**. [`codelead analyst replay`](reference/analyst-replay.md) runs it. A replay writes nothing.

### Rules and facts

A **rule** is something that the application was seen to enforce. [`codelead analyst rules`](reference/analyst-rules.md) states the rules of one or more recordings, in five kinds:

| Kind | What the recording showed |
|---|---|
| `refusal` | A write that the server refused with a status from 400 to 499, while the page showed a message. |
| `validation` | Such a refusal whose message names a field that the step filled. A wrong password refused is a validation too. |
| `effect` | A write that the server accepted, after which the page showed new headings or texts. |
| `access` | A read refused with 401 (it needs a signed-in user) or 403 (it needs a role that the user does not have). Also a screen refused when it was opened: answered 401 or 403, or sent to the sign-in page. |
| `navigation` | A write answered with a redirect, or an accepted write after which the page went to another address. |

A **fact** is something that the recordings show of the application. [`codelead analyst facts`](reference/analyst-facts.md) lists four kinds: the screens, the server's surface (each method and path), the users, and the journeys.

### Evidence: observed and assumed

Every rule and every fact has the status `observed` and cites its **evidence**: the recording and the step, written as `recording:step`. A rule also cites the request, its answer and what the page showed. Here is one rule, as `rules.observed.md` writes it:

<!-- described, not captured -->
```markdown
- **navigation** `navigation:POST /Products/Create 302@/Products/Create` (observed): After "Create product" on `/Products/Create` (`POST /Products/Create`, 302), the server redirects to `/Products/Details/{id}`.
  - `recording.json:s02` (Create product on /Products/Create; clerk): `POST /Products/Create` 302; the page showed "Desk lamp"
```

A rule holds only as far as the steps that it cites. The Analyst reads no source code.

Some parts of a draft are not observed. They are the Analyst's assumptions, and `journeys.md` lists the ones that you must confirm:

- **An id in an address is made general.** The step ended on `/Products/Details/7`, and the draft expects `^/Products/Details/[^/]+$`. It assumes that any product's page is right.
- **A refused write is expected on purpose.** The draft expects the refusal again, and asks you to confirm it.
- **A wrong password can be a guess.** Old recordings do not say whose password was typed. The draft then guesses from the server's answers.
- **A story is a draft.** The Explorer's stories say "a user story to confirm, not a requirement read from code".

> [!NOTE]
> **For team leads:** a journeys file that a person confirmed is the strongest check of a rebuild. The rebuild is measured against what real users did, not against what a model read in the code. See [The walk and parity](../modernization/walk-and-parity.md).

## What it is not

- **Not a recorder of click positions.** The draft names each control by its role and name, and each field by its label. So a replay still works when the layout changes.
- **Not a model.** No Analyst command calls a model. The same recording always gives the same draft, rules and facts.
- **Not a reader of code.** The Analyst sees only the screens and the requests that the pages send. For what the code says, see [The dossier](../modernization/dossier.md).
- **Not a proof of everything.** A replay checks the outcomes that the journeys file names, and nothing else. See [Limits](limits.md).

## Related content

- [Analyst and Explorer overview](index.md)
- [Record a journey](record-a-journey.md)
- [Explore an application](explore-an-application.md)
- [From a recording to a specification](from-recording-to-spec.md)
- [Files the Analyst writes](files.md)
- [Journeys format](../reference/journeys-format.md)
- [`codelead analyst`](../cli/codelead-analyst.md)
- [Glossary](../concepts/glossary.md)
