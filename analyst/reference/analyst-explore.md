---
title: codelead analyst explore
description: Explore a running web application by itself, as each role, within its safety layers and limits, and write a site map, the forms, the stories and a recording for each role.
kind: reference
area: analyst
date: 2026-11-20
---

# codelead analyst explore

**Applies to:** CodeLead 0.1 and later

`codelead analyst explore` explores a running application by itself, with no person and no model. As each role, it signs in, then clicks every link, menu item and button that sends no form, breadth first from the first page. It records each new screen as a step, one recording for each role.

## Name

`codelead analyst explore` - Explores a running web application by itself, as each role.

## Synopsis

```console
codelead analyst explore --base <URL> [--given-accounts --as <ROLE>=<ACCOUNT> ...]
                         [--integrated <ACCOUNT>] [--try-forms] [--max-screens <N>] [--max-depth <N>]
                         [--keep-query] [--api <ORIGIN>] [--headed] [--out <FOLDER>] [--no-derive]
codelead analyst help explore
```

## Description

The Explorer drives Playwright's Chromium without a window, unless you give `--headed`. The folder must have the pro plan. The Explorer calls no model, and it gives the same result for the same application.

### The roles

- **A visitor**, with neither `--as` nor `--integrated`: one anonymous role, `visitor`.
- **Given accounts**, with `--given-accounts` and one `--as` for each role. Each role signs in through the sign-in form of the first page. When the first page has none, the Explorer clicks its **Log in** or **Sign in** link once. Then it signs in on the page that the link leads to.
- **Windows sign-in**, with `--integrated`: one role, `alpha`, whose browser answers the application's challenge by itself.

The Explorer checks every role and reads every password before a browser opens. A role's password comes from `CODELEAD_WALK_PASSWORD_<ROLE>`, else from the credentials file that `CODELEAD_WALK_CREDENTIALS` names. See [Passwords](../passwords.md).

### What it does for each role

1. **The sign-in**, for given accounts.
2. **The crawl**, breadth first from the first page. It clicks each link, menu item and button outside a form, and reads the page that it leads to. A screen is an address and its headings.
3. **The forms**, on every screen, listed without sending them.
4. **The tries**, with `--try-forms` only: each form sent empty, then with made-up values.

It never clicks a control that signs out, deletes or removes. It never follows another origin, `mailto:`, `tel:`, `javascript:`, a new window, a download or a control with no name. It visits at most three addresses of one pattern that names a record.

### Then it writes

The site map, the forms and the stories of all the roles. Then, unless you give `--no-derive`, each role's draft, checked, and the rules and the facts of all the roles' recordings. See [Explore an application](../explore-an-application.md).

## Options

- **`--base <URL>`**

  Required. The application's address, `http` or `https`.

- **`--given-accounts`**

  The application's users exist already. Each role that `--as` names signs in as its account. It uses the sign-in form of the first page, or of the page that its **Log in** or **Sign in** link leads to.

- **`--as <ROLE>=<ACCOUNT>`**

  A role and the account that it signs in as, with `--given-accounts`. Repeat it for each role. A role's name starts with a letter (A to Z, a to z), then letters, digits or underscores. Two names that differ only in case, such as `Clerk` and `clerk`, are one role, and are refused. Its password is read from `CODELEAD_WALK_PASSWORD_<ROLE>`, the role in capitals. Without it, the password is read from the file that `CODELEAD_WALK_CREDENTIALS` names. It is never written or printed. The default is no role.

- **`--integrated <ACCOUNT>`**

  The application signs its users in with the browser's own session, Windows authentication. The browser answers its challenge as this account, with its password when one is given: `CODELEAD_WALK_PASSWORD_ALPHA`, else the file that `CODELEAD_WALK_CREDENTIALS` names. Otherwise it uses this computer's session, as the account signed in to it. There is one role, `alpha`.

- **`--try-forms`**

  After the crawl, sends each form twice, empty and then with made-up values, and records what the application answered. It never sends a form with a password field, or a form whose button deletes, removes or signs out. Use it on a copy of the application only: the forms write what they write.

- **`--max-screens <N>`**

  At most N screens for each role. A whole number from 1. The default is 200.

- **`--max-depth <N>`**

  At most N moves from the first page. A move is a click or the sign-in. A whole number from 0. The default is 8.

- **`--keep-query`**

  Keeps the query part of addresses. Then `/Products?active=true` is another screen than `/Products`, and the recordings keep the query.

- **`--api <ORIGIN>`**

  The origin of the application's server, when it is not the address's origin.

- **`--headed`**

  Shows the browser while the Explorer works.

- **`--out <FOLDER>`**

  Where everything is written. The default is `./analyst-recordings/<TIME>/`.

- **`--no-derive`**

  Writes the recordings, the site map, the forms and the stories only. No draft, no rules and no facts.

## Examples

- Explore the Basic Store sample as a visitor:

  ```console
  codelead analyst explore --base http://localhost:5080/
  ```

  <!-- described, not captured -->
  ```output
  visitor: s01 reached / (Dashboard).
  visitor: s02 reached /Clients (Clients) by the link "Clients".
  …
  Explored as visitor into analyst-recordings/2026-11-20T09-15-00Z: analyst-recordings/2026-11-20T09-15-00Z/explore-visitor/recording.json.
  Written: sitemap.json and sitemap.md, forms.json and forms.md, stories.json and stories.md, rules.observed.json and rules.observed.md, facts.observed.json and facts.observed.md.
  visitor's draft: analyst-recordings/2026-11-20T09-15-00Z/explore-visitor/journeys.json and journeys.md; no problems, CodeLead can walk it.
  ```

- Explore as two given accounts, with their passwords in the terminal:

  ```console
  export CODELEAD_WALK_PASSWORD_CLERK='<CLERK_PASSWORD>'
  export CODELEAD_WALK_PASSWORD_ADMIN='<ADMIN_PASSWORD>'
  codelead analyst explore --base http://localhost:5080/ --given-accounts --as clerk=store.clerk --as admin=store.admin
  ```

  <!-- described, not captured -->
  ```output
  clerk: s01 signed in on /Account/Login, reached from / through "Log in".
  clerk: s01 reached / (Dashboard) by the sign-in.
  …
  admin: … screens (0 refused), 0 not visited, … never followed; recording in explore-admin/recording.json.
  Explored as clerk, admin into analyst-recordings/2026-11-20T09-15-00Z: …
  ```

- Explore a copy of the application, send its forms, and stop at 50 screens for each role:

  ```console
  codelead analyst explore --base http://localhost:5080/ --try-forms --max-screens 50
  ```

  <!-- described, not captured -->
  ```output
  …
  visitor: s27 tried "Create product" on /Products/Create, sent empty: POST /Products/Create …; it ended on /Products/Create.
  visitor: s28 tried "Create product" on /Products/Create, sent with values the explorer made up: POST /Products/Create …; it ended on ….
  …
  ```

  Each try line gives the writes that the try caused, with their statuses, and the address after. `forms.md` lists the same tries under each form.

- Explore with given accounts whose passwords are not set. Nothing opens, and the exit code is 2:

  ```console
  codelead analyst explore --base http://localhost:5080/ --given-accounts --as clerk=store.clerk --as admin=store.admin
  ```

  ```output
  The explorer signs in as each account with its password, read from CODELEAD_WALK_PASSWORD_<ROLE> (the role in capitals), else the file CODELEAD_WALK_CREDENTIALS names (each account mapped to its password; not set here), and has none for these: CODELEAD_WALK_PASSWORD_CLERK (clerk, store.clerk), CODELEAD_WALK_PASSWORD_ADMIN (admin, store.admin). Set them in this terminal, or name with CODELEAD_WALK_CREDENTIALS a file outside this folder that maps each account to its password (the explorer reads a password, and never writes or prints one), then explore again. Nothing was explored.
  ```

- Name a credentials file inside the folder where the Explorer runs. Nothing opens, and the exit code is 2:

  ```console
  CODELEAD_WALK_CREDENTIALS=walk-credentials.json codelead analyst explore --base http://localhost:5080/ --given-accounts --as clerk=store.clerk
  ```

  ```output
  CODELEAD_WALK_CREDENTIALS names a file inside the folder the Explorer runs in: the passwords are kept in a file outside it, where nothing commits them. Name with CODELEAD_WALK_CREDENTIALS a file outside this folder (or set CODELEAD_WALK_PASSWORD_<ROLE>, the role in capitals), then explore again. Nothing was explored.
  ```

- Name a role that starts with a digit. Nothing opens, and the exit code is 2:

  ```console
  codelead analyst explore --base http://localhost:5080/ --given-accounts --as 1clerk=store.clerk
  ```

  ```output
  --as 1clerk=store.clerk: a role's name starts with a letter (A to Z), then letters, digits or underscores, such as manager or clerk_2, and "1clerk" does not start with a letter. Nothing was explored.
  ```

- Name one role twice, in two cases:

  ```console
  codelead analyst explore --base http://localhost:5080/ --given-accounts --as Clerk=store.clerk --as clerk=store.other
  ```

  ```output
  --as names the role clerk twice (as Clerk and clerk, which read one variable, CODELEAD_WALK_PASSWORD_CLERK): each role is one account. Nothing was explored.
  ```

- Name an account without `--given-accounts`:

  ```console
  codelead analyst explore --base http://localhost:5080/ --as clerk=store.clerk
  ```

  ```output
  --as names an account that exists already: add --given-accounts (the application's users exist already, and each role signs in as its account).
  ```

- Give an address without `http://`:

  ```console
  codelead analyst explore --base localhost:5080
  ```

  ```output
  --base is not an http or https address: localhost:5080 (for example http://localhost:5199).
  ```

- Give a limit that is not a whole number from 0:

  ```console
  codelead analyst explore --base http://localhost:5080/ --max-depth -1
  ```

  ```output
  --max-depth is a whole number from 0, not "-1".
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The exploration was written. A role that could not sign in still gives 0: its site map row says why. |
| `1` | The exploration stopped on an error, such as Chromium not installed. Also `--base` missing, and a folder without the pro plan. |
| `2` | Refused before a browser opened: a password not given, a role's name that breaks the rule, a role named twice, `--as` without `--given-accounts`, `--integrated` with `--as`, an address or a limit that is not valid. Also a credentials file that is inside the folder, missing, or not JSON. |

## Files

| File | What `explore` does with it |
|---|---|
| `<OUT>/explore-<ROLE>/recording.json`, `recording.md`, `steps/` | Writes each role's recording, `codelead.recording/1`. |
| `<OUT>/explore-<ROLE>/journeys.json`, `journeys.md` | Writes each role's draft. Not with `--no-derive`. |
| `<OUT>/sitemap.json`, `sitemap.md` | Writes the screens, `codelead.sitemap.observed/1`. |
| `<OUT>/forms.json`, `forms.md` | Writes the forms, `codelead.forms.observed/1`. |
| `<OUT>/stories.json`, `stories.md` | Writes the stories, `codelead.stories.observed/1`. |
| `<OUT>/rules.observed.*`, `facts.observed.*` | Writes the rules and the facts. Not with `--no-derive`. |
| `.env` | Reads `CODELEAD_WALK_PASSWORD_<ROLE>`, when the variable is not set in the terminal. |
| The file that `CODELEAD_WALK_CREDENTIALS` names | Reads a role's password by its account, when the role's variable is not set. It must be outside the folder where the Explorer runs. |

`<OUT>` is `--out`, or `analyst-recordings/<TIME>/`. See [Files the Analyst writes](../files.md).

## See also

- [Explore an application](../explore-an-application.md)
- [The Explorer view](../../extension/explorer-view.md)
- [`codelead analyst record`](analyst-record.md): record the journeys that matter.
- [`codelead analyst draft`](analyst-draft.md) and [`codelead analyst replay`](analyst-replay.md)
- [Passwords](../passwords.md)
- [Limits](../limits.md)
- [`codelead spec`](../../cli/codelead-spec.md): `spec coverage` runs the Explorer.
