---
title: The walk and parity
description: How the walk runs the journeys on a running application, step by step, and how parity decides that the new application does what the old one did.
kind: concept
area: modernization
date: 2026-11-25
---

# The walk and parity

The walk uses a running application the way a new person does. It follows each journey in a real browser, step by step, and marks each step pass, fail or blocked. Parity is the decision that the new version does what the old version did. For a rebuild, parity needs a clean walk.

**In short**
- You know what the walk does, what pass, fail and blocked mean, and why the walk needs the application running.
- You know the two kinds of parity: page parity over HTTP, and the parity of a rebuild.
- You know what a waiver is, and where you record one.

## Why they exist

Tests check the code from the inside. A person uses the application from the outside: they open a page, click a link, fill a form, and expect to see something. The walk checks that outside view. A rebuild has new pages and new code, so no test of the old code can prove it. The journeys can, because they describe what users do, not how the code does it.

## The walk

[`codelead walk`](../cli/codelead-walk.md) reads the journeys and follows each one in Playwright's Chromium:

- It starts at the application's first screen, and reaches every page by clicking. It never jumps to an address.
- It prepares nothing by hand. A new user registers through the screens. For a journey with a given account, the walk signs in as that account.
- After each step, it checks what the step expects, and marks the step **pass** or **fail**.
- When only the step's expectations failed it, the report says which ones, by their place in the step. For example, `"unmet": [1]` is the step's second expectation.
- A step after a failed step is **blocked**: it could not be tried.

The walk needs the application running at the address that you give with `--base`. In the Rebuild stage, CodeLead starts the rebuild's API and web application from the run profile, walks them, and stops them after. When you walk by hand, start them yourself first.

Before the first journey, the walk sends one request to the base address. When nothing answers, it stops with exit code 2. It walks no step, runs no reset and writes no report. On the Basic Store sample's rebuild, before its servers started:

```output
Nothing answers at http://127.0.0.1:5791 (connect ECONNREFUSED 127.0.0.1:5791): nothing was walked, the run profile's reset was not run and no walk report was written. Start the application first (for a rebuild: the API and the web application its run profile names, run.api and run.web in docs/codelead/rebuild/rebuild.json), then walk again.
```

When the servers run, the walk prints one line for each step. Each step names its place in the old code, so a failure points at what the new code must do. When every step passes, the last line counts only passes, and the exit code is 0.

> [!NOTE]
> **New to this?** A journey's step says what a person does and what they must see after it. For example: click "Create product", then see the message "Another product already uses this SKU.". See [Journeys format](../reference/journeys-format.md).

### An honest walk

The walk refuses to record a result that it cannot trust. It stops with exit code 2 when the server lends a development identity, for example through a request header such as `x-dev-user`. Serve the application without one. The report also records what was walked: the code's stamp and the journeys' content hash. Parity trusts a walk only for the code and the journeys that it walked.

A walk shows one tree. So CodeLead stamps the code when the walk begins and again when it ends. When the two stamps differ, the walk judged no single tree, and the part is not walked. The words then name the files that differ, a few and the count of the rest. When the run profile's day reset changed one of them, they say that too:

<!-- described, not captured -->
```output
The code changed while the walk ran (…): no single tree was walked, and parity will not treat this walk as current. What differs: api/Api/sample-seed.db (removed). The day reset (scripts/reset-sample-seed.mjs) removed api/Api/sample-seed.db, a tracked file: it should be ignored, not tracked. The way out: …
```

The sample database is the usual case. Every walk changes it: its journeys write, and the day reset puts the seed back before each one. So it must never be tracked. See [The data access](rebuild.md#the-data-access).

The passwords of the journeys' accounts come from `CODELEAD_WALK_PASSWORD_<USER>`, or from the file that `CODELEAD_WALK_CREDENTIALS` names. They never come from the journeys, and the walk never prints them. See [Passwords](../analyst/passwords.md).

### The checks before a walk

CodeLead never walks blind. Once the servers answer, and before the first step, it makes its own checks. A failed check means no walk: the part is "not walked", and it is not blamed.

**The data after the day reset.** A walk on a database that the API cannot read judges every screen wrongly. So CodeLead runs the day reset once against the started servers. Then it asks an endpoint that reads data. It takes that endpoint from the plan. The endpoint's old action only reads, it answers GET with no parameter in its route, and it is never the part being judged. Its old action also needs no value of the request. A list or a dashboard fits. A page that takes a row's id does not, for example `Details(int? id)`, which goes back to the list without one. When the walk has an account with a password, CodeLead signs it in first. The check fails on a 500 or no answer, and on an empty answer when the seed's rows are known. For example:

<!-- described, not captured -->
```output
sailors.create-post:endpoint: landed (walk r0); it was not walked (check-failed: the data after the day reset: after the day reset the API answers 500 at GET /api/home/index ("Microsoft.Data.Sqlite.SqliteException (0x80004005): SQLite Error 1: 'no such table: Orders'."): the reset removes the sample database and the API does not make it again).
```

The words say why, by what the reset is: CodeLead's reset, its earlier one, or yours. The plan then plans `data-access` again, as [the data access](rebuild.md#the-day-reset-and-the-api-one-contract) says. The part that was not walked waits on that check, and the run judges it again by itself once a walk gets past it. A check that passes is said once in a session: `Acceptance: CodeLead's check of the data: after the day reset GET /api/sailors/index answers 200, with the seed's row …`. Before any endpoint reads data, there is no such check.

Some answers say nothing of the data, such as a 401, a 404 or a 409. Then CodeLead asks the next endpoint, at most three in all. When none of them can tell, the line says so plainly. Nobody should read it as a pass:

<!-- described, not captured -->
```output
Acceptance: CodeLead's check of the data: the data was NOT checked after the day reset: GET /api/clients/details answers 409; GET /api/orders/index answers 401, which say nothing of the data (the check asks at most 3 endpoints). The walk goes on, and nothing here shows that the API reads its data.
```

The walk of the Rebuild stage makes the same check before it walks, with the same words.

In a rebuild whose API holds a fake directory, CodeLead signs the walk's first account in itself before the walk. It sends the fields of the old sign-in form: the account, the password, and each hidden field, empty. When the API refuses it (HTTP 401 or 403), the walk is not run, and the line says `the API refused the walk's own account …`. Any other answer says nothing about the account, so the walk runs and its own sign-in step judges. An account may have no roles, and the fake directory must accept it so. When its code shows that it needs roles, the plan plans the sign-in's endpoint again, to be revised in place.

The walk of the part `user-directory` is different. Once the API's code holds the real directory, CodeLead starts the API with `SignIn__Directory=real`. It gives the fake directory no account. Its own sign-in check then goes through the real directory, with the walk's own account and its password from the environment. Only a 2xx answer passes, because that account is one of the old application's own. Then the same account with a wrong password, and an account no database holds, must both be refused. A failed check is that part's to repair, and the line says `through its real directory UserDirectory`. The accepted line says which directory the check went through. Every other walk stays on the fake directory. See [The user directory](rebuild.md#the-user-directory).

### The walk of a rebuild's part

In the Rebuild stage, the walk judges each part that lands. The part is accepted when its own steps pass, no step regresses, and no new finding counts.

**What a regression is.** A step regresses when it passed in the walk before the change and fails in the walk after it. It is that change's regression, whoever owns the step. The step may be the own step of another part, landed and not accepted. It passed before this change, so this change broke it. The acceptance line says whose step it is:

<!-- described, not captured -->
```output
Acceptance (walk r0): not accepted. No step of the journeys covers this request. Regressions: account.sign-out-2, clients.create-post-2. They are own steps of sign-in:screen, landed and not accepted: they passed before this change, so they are this change's. New findings: none.
```

A step that failed before the change is no regression, even when it fails again. A new finding at such a step, at another unfinished part's own step, is still that part's. A step that passes in one walk after the change and fails in the other is flaky, and it does not count.

A step has two ends: the page where its control is, and the page that it reaches. The step judges the screen that it reaches, once the page that holds its control exists. See [A step has two ends](rebuild.md#a-step-has-two-ends). When an earlier step fails, the walk cannot reach the part's own steps. Then no walk judges the part, and it is not finished. The run judges it again once that step passes. An accepted part stays accepted only while its own steps are the ones its walk judged. See [When a part's steps change](rebuild.md#when-a-parts-steps-change).

A step can pass on empty data. For example, "click Clients, the address is the clients' list" passes when the list is empty. So an endpoint is not accepted on its contract alone when its old action reads or writes the database. CodeLead reads the endpoint's code too. An endpoint that reaches no data is accepted on its contract, and it reads no data yet. It is revised once the data access lands. The screens judged on its empty answer are judged again. See [The data access](rebuild.md#the-data-access).

The judge reads some findings before it counts them:

- **The signed-out state.** The caller-identity route, such as `GET /api/me`, tells the web application who is signed in. It answers 401 to a user who has not signed in yet in the journey. That is the signed-out state of the shell, not a fault. The judge shows it and does not count it: `Signed out, as expected (not counted): GET /api/me (identity-seam) answered 401 to a user not signed in yet, the shell's signed-out state, at <STEPS>.` A 401 after the user signed in still counts. See [Expected findings](#expected-findings).
- **The old model's own refusal.** The old action refuses the form with the same message, and the step passes on the old application too. The judge shows it and does not count it. See [Expected findings](#expected-findings).
- **A cross-origin block.** An origin is the scheme, the host and the port of an address. The API is up at another origin than the page's, and the browser gets no answer at all from it. The judge names this a cross-origin block: the API's CORS policy refuses the web application's origin, or the credentials that its calls send. A CORS policy is the API's rule for which origins may call it. The finding says so, in place of a wrong route.
- **A page not accepted yet.** The step's control is on the page of a part that is not accepted yet. The finding says so, for example `nothing to click named "New client" on / (the page of home.index:screen, which is not accepted yet)`.
- **What another part builds.** The step failed only on what another part builds, and that part does not build it yet. An example is the signed-in name that the old shell showed, which the shell's menu builds. The part judged gets no repair round, and waits on that part. See [What the old shell shows](rebuild.md#what-the-old-shell-shows).
- **A step with no action of its own.** The draft could name nothing to act on, so no walk can pass the step. No part is judged on it, and a person completes it. See [Steps a person completes](../reference/journeys-format.md#steps-a-person-completes).

The rebuild's walk reads the journeys' old addresses as the new ones. Some steps click the first link to an old address, because the link's text is computed. Such a step clicks the first link to the new screen's address. For example, `/Orders/Details` becomes `/orders/details`. See [A link whose text is computed](../reference/journeys-format.md#a-link-whose-text-is-computed).

An old address can carry a value of its row, such as the id in `^/Products/Edit/[^/]+$`. The rebuild carries it in the query, so the walk then asks for it: the step passes at `/products/edit?id=3`, and fails at `/products/edit`. See [Where an old redirect lands](rebuild.md#where-an-old-redirect-lands).

### Expected findings

Some findings are expected. CodeLead decides this with one rule, in one place. A part's acceptance, the walk's summary and its exit code, parity, the Rebuild stage's line and the audit pack all read it. An expected finding stays listed, apart, with its reason. It is only not counted.

- **The signed-out state.** The caller-identity route, such as `GET /api/me`, answers 401 to a user who is not signed in. That is before the journey's sign-in, or after its sign-out.
- **The old model's own refusal.** A step sends the form as the old page shows it, for example with an empty SKU. The new endpoint answers 400 with the old model's own message, `The SKU is required.`. The old action refuses the same form with the same words. All of these must be true:
  - the step passes;
  - each request of the finding is a planned endpoint that answered 400;
  - the answer carries a message that the old model writes for that endpoint (the plan reads it from the old code);
  - the journeys' proof passes the step on the old application.

On the Basic Store sample's full run, the walk's last lines then read:

<!-- described, not captured -->
```output
39 steps: 38 pass, 1 fail, 0 blocked. 12 findings.
…
Expected, not counted: 11 × GET /api/me 401 where nobody is signed in (…); 1 × POST /api/products/create 400 with the old model's own message "The SKU is required." (the old action refuses the same form: …; at products.create-post-5, which passes on the old application as well).
Not the rebuild's: orders.cancel-5 fails on the old application as well (the proof of 2026-11-25): a defect of the drafted journey, not of the rebuild. Every other step passes.
```

Anything else stays a finding. A 401 after the user signed in counts. So does a 400 with other words, or at a step that fails, or at a step that the old application does not pass.

### The old application's journeys

The journeys that you record in the Observe stage can also run on the old application. [`codelead analyst replay`](../analyst/reference/analyst-replay.md) walks a recording's `journeys.json` against the old application's address. A replay that passes on the old application shows that the journey itself is right, before the rebuild is measured against it.

The drafted journeys get the same proof. Once the rebuild holds its copy, the stage Initialize the rebuild asks you before it walks them on the old application. In the rebuild, `codelead rebuild journeys --prove` does the same at any time. A step that fails there is a defect of the draft. No part is judged on it, and the plan warns with `draft-defect`. For example, a draft that expects the edit form's own address after "Save", where the old application goes on to the client's page, fails there. See [`codelead rebuild journeys`](../cli/codelead-rebuild-journeys.md#with---prove-the-journeys-walked-on-the-old-application).

Such a step is never held against the rebuild. Wherever a failing step is said, it is said apart:

<!-- described, not captured -->
```output
Not the rebuild's: orders.cancel-5 fails on the old application as well (the proof of 2026-11-25): a defect of the drafted journey, not of the rebuild. Every other step passes.
```

- **The walk** says it after its counts, as above. When it adds `Every other step passes.`, the walk's exit code is 0.
- **A part's acceptance** counts no regression and no finding at that step. Its line says `Not this part's: …`.
- **Parity** reads a walk that fails only on such steps, and on steps blocked behind them, as passing. Its note names them.
- **The stage** says `the walk passed but for what is not the rebuild's (…)`.
- **A part whose own steps wait only on such a step** is your turn: correct the draft. The run does not wait to judge it.

A finding that counts, at another step, still counts. So a walk whose only failing step is the draft's, with such a finding at another step, does not pass. The Rebuild stage's line then says what is true, with the walk's counts and the draft's step apart. For example, with one page error at another step:

<!-- described, not captured -->
```output
… the walk found no failing step of the rebuild's, and 1 finding (39 steps at `http://localhost:3000`: 38 pass, 1 fail, 0 blocked, 1 finding that counts; expected, not counted: …; not the rebuild's: orders.cancel-5 fails on the old application as well (the proof of 2026-11-25): a defect of the drafted journey, not of the rebuild); parity says what is left.
```

The stage's line says one of these about the walk:

| The walk | The stage's words |
|---|---|
| Every step passed, with no finding that counts. | `the walk passed`, or, with expected findings, `the walk passed but for what is not the rebuild's (…)` |
| Only the draft's steps fail, and the steps behind them, with no finding at another step. | `the walk passed but for what is not the rebuild's (…)` |
| Only the draft's steps fail, and findings that count remain at other steps. | `the walk found no failing step of the rebuild's, and 1 finding (…)` |
| A step of the rebuild fails or is blocked. | `the walk found what fails (…)`, with the counts, and the draft's steps apart |
| The walk recorded nothing, for example because nothing answered. | `the walk recorded nothing (its words are above)` |
| The walk could not start. | `the walk was not run (…)`, with the reason |

Some controls are rendered only under a condition. For example, the old order page writes "Cancel this order" inside `@if (Model.Status == Order.Placed)`. The drafted step then says so, with the condition and its place: `the old view renders it only when`, then `Model.Status == …` and `Views/Orders/Details.cshtml:25`. The draft does not guess which row meets the condition. When the proof finds nothing to click there, its words name the condition the page did not meet.

## Parity

*Parity* means that the new version does what the old version did. CodeLead has two kinds, for two different changes. Both are jobs of [`codelead parity`](../cli/codelead-parity.md).

### Page parity over HTTP

Use page parity when the same application changes and its pages must not, for example after a change of its user interface framework.

1. On the unchanged application, capture a baseline: `codelead parity --capture <URL>`. CodeLead fetches every GET route, removes what changes at every request, and stores each page.
2. Make the change.
3. Check the changed application: `codelead parity --check <URL>`. Each route is `same`, `changed`, `missing`, `new`, `error` or `waived`.

The check fails on a route that changed, is missing or gave an error, unless the route is waived. Declare the check in the [gate](../concepts/validation-and-the-gate.md) of the change, so that every landing runs it.

### Parity of a rebuild

`codelead parity --rebuild` fetches nothing. It reads the rebuild's files, and it is the gate for calling the rebuild finished. It checks:

- every feature of the old application's map: landed as an endpoint and a screen, or dropped by a recorded decision;
- the pins: each feature that carries pins has a contract test that runs and sends the feature's endpoint, its method and its route;
- the wiring of the new code: what a person cannot reach, what the code calls and nothing serves;
- the last walk: a clean walk of the code as it is now, with the rebuild's own journeys.

A contract test proves an endpoint only when it calls that endpoint. So parity reads what each test sends, not only the name of its file. Two statuses of a pin are blockers of their own:

- `calls another endpoint`: the tests run, and none sends the endpoint. They send another method or route.
- `not read`: the tests run, and CodeLead cannot read what they send, such as a route held in a constant.

In this example from CodeLead's tests, the route of the GET's test is held in a constant. The Pins table reads:

<!-- described, not captured -->
```output
| clients.create-get | GET /api/clients/create | 1: Create_ShowsTheForm | ClientsCreateGetContractTests.cs (1 test runs; what 1 sends cannot be read) | **not read** |
```

Write the route as text in the test's request, so that CodeLead can read it. See [`codelead parity`](../cli/codelead-parity.md#the-pins-of-a-rebuild) for every status.

#### A feature that the plan builds in a part of its own

Some old features get no endpoint and no screen of their own in the map. The sign-in is one: the map's note says `replaced by the new API's identity scheme`. The plan builds the sign-in in two parts, `sign-in:endpoint` and `sign-in:screen`. Each part says in the plan which old features it builds, with its endpoint or its screen's route. Parity reads that, and checks them like any other:

<!-- described, not captured -->
```output
| account.sign-in-post | auth | POST /Account/SignIn (SignInModel model) | POST /api/account/sign-in ✓ | — | **landed** | as the plan builds it: the sign-in's endpoint (the part sign-in:endpoint, accepted), POST /api/account/sign-in; … |
```

- **The pins** of such a feature are read in the test file that its part's request asks for, `SignInContractTests.cs`.
- **A screen alone.** The old sign-in page becomes a screen with no endpoint. No contract test can run on it, so its pins stay a blocker. The row says why, and parity does not read the screen's own tests.
- **An older plan.** A plan made before plans named these features says nothing of them. Parity then says `plan again` in the row's reason.

#### What the wiring does not count

The wiring section counts the findings that are left to resolve. Some cases are not the rebuild's to resolve. Parity reads why in the rebuild's own files.

- **A page reached by a redirect.** A screen saves, then goes where the endpoint's answer says: `router.push(res.redirectTo)`. The endpoint writes that address, for example `redirectTo = $"/clients/details?id={stored.Id}"`. So the page is reached, and its row in the route table says how:

  <!-- described, not captured -->
  ```output
  | /clients/details | ClientsDetailsPage | only as the result of the redirect the endpoint POST /api/clients/edit answers (redirectTo in api/Api/Controllers/ClientsController.cs:169) |
  ```

  When no endpoint answers a redirect to a page, and nothing else leads there, the page is still a finding.
- **An endpoint that the run profile calls.** Nothing in the web application calls `GET /health`. The run profile's start check calls it: every walk and every run wait for it to answer 200 (`run.api.ready` in `rebuild.json`). So the finding is listed apart, with its reason, and not counted:

  <!-- described, not captured -->
  ```output
  - uncalled-endpoint · GET /health (api/Api/Controllers/HealthController.cs:9): nothing in the SPA calls this endpoint; the run profile's start check calls it: every walk and every run start the API and wait for /health to answer 200 before anything else (run.api.ready in docs/codelead/rebuild/rebuild.json)
  ```

Another endpoint that nothing calls is still a finding.

Parity checks the whole rebuild, at its end. Each part is also protected on the way. Before a patch lands, the patch check refuses one that breaks what an accepted part built: an endpoint, a screen's route or a test. See [What an accepted part built stays built](rebuild.md#what-an-accepted-part-built-stays-built).

The screens are read in the web application's layout: the route table of a React Router application, or the folders of a Next.js app router. On the Basic Store sample's new rebuild, before any part landed, it says what is left:

```output
**Routes:** 63 features — 0 landed, 0 endpoint only, 0 screen only, 0 endpoint not read, 26 screen not read, 0 dropped by decision, 37 missing.
…
**Walk:** none — the rebuild has never been walked (`codelead walk --base <url>`): nothing shows that a new user can do what the old application allowed.
**Finished:** no — 5 blockers:
```

This report was captured before CodeLead read the routes of a Next.js app router, the web application of the default template. Its `26 screen not read` comes from that gap. Today the numbers of a new rebuild differ.

The rebuild is finished only when this report says `Finished: yes`. The report is `docs/codelead/parity/rebuild-parity.md`.

## Waivers

A **waiver** records a difference that you expect, with its reason. A waived route passes the page parity check, and the report shows its reason. Waivers live in `docs/codelead/parity-waivers.json`:

```json
{
  "waivers": [
    { "route": "/Home/About", "reason": "The About page has new text on purpose.", "by": "<your name>", "date": "2026-11-20" }
  ]
}
```

In a rebuild, a feature that you decide not to carry is not waived. It is dropped by a recorded decision in the manifest, and parity counts it as `dropped by decision`. A security finding that you decide not to fix has its own record: [`/modernize --waive`](../chat-commands/modernize.md#--waive). See [Parity waivers](../reference/parity-waivers.md).

## Details

- **The journeys**: `docs/codelead/rebuild/journeys.json` in the rebuild, or `--journeys <FILE>`.
- **The walk's report**: `docs/codelead/parity/walk.md` and `walk.json` in the rebuild. `--no-write` prints it and writes nothing, for any application.
- **The time zone**: the walk reads the server's day in the run profile's zone, or in `--time-zone <ZONE>`.
- **Page parity's files**: the baseline in `docs/codelead/parity/baseline/`, each check in `docs/codelead/parity/latest/`. The routes come from the dossier's structural inventory, or from `--routes`.
- **Exit codes** of `codelead walk`: `0` every step passed, or only the draft's steps did not pass. `1` a step of the rebuild failed or was blocked, or a finding counts. An [expected finding](#expected-findings) does not count. `2` nothing honest could be recorded. See [Exit codes](../cli/codelead-walk.md#exit-codes). Of `codelead parity --rebuild`: `0` finished, `1` a blocker is open.
- **A browser**: the walk needs Playwright's Chromium. [`codelead doctor`](../cli/codelead-doctor.md) checks that it starts.

## What it is not

- **The walk is not a screenshot test**. It checks what each step expects, such as a page, a message or an address. It does not compare pixels.
- **Page parity is not for a rebuild**. A rebuild has new pages, so they differ by design. The parity of a rebuild checks features, contracts and a clean walk.
- **A waiver is not a silent pass**. It carries its reason, its author and its date, and the report shows them.

## Related content

- [`codelead walk`](../cli/codelead-walk.md) and [`codelead parity`](../cli/codelead-parity.md)
- [The rebuild](rebuild.md)
- [Journeys format](../reference/journeys-format.md) and [Parity waivers](../reference/parity-waivers.md)
- [Record a journey](../analyst/record-a-journey.md)
- [`codelead analyst replay`](../analyst/reference/analyst-replay.md)
- [`codelead spec`](../cli/codelead-spec.md): journeys written in plain words
- [Glossary](../concepts/glossary.md)
