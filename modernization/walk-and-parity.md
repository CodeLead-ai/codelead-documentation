---
title: The walk and parity
description: How the walk runs the journeys on a running application, step by step, and how parity decides that the new application does what the old one did.
kind: concept
area: modernization
date: 2026-11-20
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

The passwords of the journeys' accounts come from `CODELEAD_WALK_PASSWORD_<USER>`, or from the file that `CODELEAD_WALK_CREDENTIALS` names. They never come from the journeys, and the walk never prints them. See [Passwords](../analyst/passwords.md).

In a rebuild whose API holds a fake directory, CodeLead signs the walk's first account in itself before the walk. When the API refuses it, the walk is not run, and the line says `the API refused the walk's own account …`. An account may have no roles, and the fake directory must accept it so. When its code shows that it needs roles, the plan plans the sign-in's endpoint again, to be revised in place.

### The walk of a rebuild's part

In the Rebuild stage, the walk judges each part that lands. The part is accepted when its own steps pass, no step regresses, and no new finding counts. A step has two ends: the page where its control is, and the page that it reaches. The step judges the screen that it reaches, once the page that holds its control exists. See [A step has two ends](rebuild.md#a-step-has-two-ends). When an earlier step fails, the walk cannot reach the part's own steps. Then no walk judges the part, and it is not finished. The run judges it again once that step passes. An accepted part stays accepted only while its own steps are the ones its walk judged. See [When a part's steps change](rebuild.md#when-a-parts-steps-change).

The judge reads some findings before it counts them:

- **The signed-out state.** The caller-identity route, such as `GET /api/me`, tells the web application who is signed in. It answers 401 to a user who has not signed in yet in the journey. That is the signed-out state of the shell, not a fault. The judge shows it and does not count it: `Signed out, as expected (not counted): GET /api/me (identity-seam) answered 401 to a user not signed in yet, the shell's signed-out state, at <STEPS>.` A 401 after the user signed in still counts.
- **A cross-origin block.** An origin is the scheme, the host and the port of an address. The API is up at another origin than the page's, and the browser gets no answer at all from it. The judge names this a cross-origin block: the API's CORS policy refuses the web application's origin, or the credentials that its calls send. A CORS policy is the API's rule for which origins may call it. The finding says so, in place of a wrong route.
- **A page not accepted yet.** The step's control is on the page of a part that is not accepted yet. The finding says so, for example `nothing to click named "New client" on / (the page of home.index:screen, which is not accepted yet)`.

### The old application's journeys

The journeys that you record in the Observe stage can also run on the old application. [`codelead analyst replay`](../analyst/reference/analyst-replay.md) walks a recording's `journeys.json` against the old application's address. A replay that passes on the old application shows that the journey itself is right, before the rebuild is measured against it.

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
- **Exit codes** of `codelead walk`: `0` every step passed, `1` a step failed or was blocked, `2` nothing honest could be recorded. Of `codelead parity --rebuild`: `0` finished, `1` a blocker is open.
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
