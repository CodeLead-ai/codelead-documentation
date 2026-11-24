---
title: The rebuild
description: How CodeLead builds a new application beside the old one, part by part, from a template, a plan and the journeys, and runs it unattended under caps.
kind: concept
area: modernization
date: 2026-11-20
---

# The rebuild

A rebuild is a new application, made in its own folder beside the old one, that does what the old one did. CodeLead makes the folder from a template, plans the work in small parts, and builds the parts one after another. Each part is checked against the old behaviour before the next one starts. Two stages do this work: Set up the rebuild, then Rebuild.

**In short**
- You know when to rebuild and when to upgrade in place.
- You know what `codelead rebuild init` makes: the template, the manifest, the run profile, the journeys and the plan.
- You know how the plan cuts the work into parts and lanes, and how a step's two ends order the parts.
- You know what the unattended run does, where it stops, and what a parked part is. You know why a later part cannot undo an accepted one.

## When to rebuild, and when to upgrade in place

| Upgrade in place, with [transforms](transforms.md) | Rebuild |
|---|---|
| The old code can stay, and parts of it can move to current .NET. | The old stack has no way forward, for example ASP.NET MVC 5 on System.Web. |
| A class library, a project format, a dependency, a security finding. | The web layer: the pages and the actions behind them. |
| Each change is small, and the pins prove it. | Each part is new code, and the journeys prove it. |

The two work together. On the Basic Store sample, Upgrade in place moves the data layer into a library and gives it a current target. The rebuild then makes a new API and a new web application in place of the MVC pages.

The rebuild reads the [System Model](system-model.md) and its feature map, so it needs an application that the model reads: today, ASP.NET MVC. A Web Forms project has no rebuild of its pages.

## Set up the rebuild

[`codelead rebuild init`](../cli/codelead-rebuild-init.md) makes the rebuild. The program's Set up the rebuild stage runs it with your answers. It needs two answers when the old application signs users in and reaches a database. Without them, it refuses and makes nothing:

```output
Refused: Answers the manifest needs are missing: identity: the old application signs users in (20 account features). Pass --identity keep (the new API keeps the accounts, with ASP.NET Core Identity over the old user tables) or --identity drop (the rebuild has no sign-in; the account features are dropped); database: the old application's code reaches a database (4 tables). Pass --database <connection name> (the name of the connection string the new API reads, kept out of the repository) or --database sample-seed (the new API starts on a sample database, put back before every journey).
  The way out: Run the command again with the answers, for example: codelead rebuild init --out <folder> --identity keep --database sample-seed
Nothing was made.
```

With the answers, on the Basic Store sample:

```console
codelead rebuild init --out ../basic-store-rebuild --identity keep --database sample-seed --no-install
```

```output
Rebuild of webapp (.) from the template dotnet-next-shadcn (…/dotnet-next-shadcn), in ../basic-store-rebuild.
Template: 43 files copied.
Ports: the API on 5081 (the template's 5080 moved), the web on 3000; the old application answers on 5080 (the program's address), 44366 (webapp.csproj's IISUrl), 52895 (webapp.csproj's DevelopmentServerPort), and both run side by side for the walk.
Git: the folder is a git repository; what is made is committed at the end.
Install: skipped (--no-install): run npm install in ../basic-store-rebuild before the first run.
Manifest: docs/codelead/rebuild/rebuild.json: aspnet-mvc-to-api-and-spa; API api/Api/Api.csproj (net10.0), web web; identity keep, database sample-seed, channel none.
Run profile: API npm run api:dev on http://localhost:5081, web npm run dev on http://localhost:3000; clock America/Los_Angeles; parking after 3 attempts; day reset node scripts/reset-sample-seed.mjs; unattended caps 12 parts, $20, 480 minutes.
Journeys: docs/codelead/rebuild/journeys.json: 20 journeys (model 20, analyst 0, explorer 0); 0 duplicates removed.
CodeLead: initialized (config, repository map, a board); no licence in the old application to copy; .codelead/CONVENTIONS.md to fill in; the gate .codelead/validation.json: API build (npm run api:build), API tests (npm run api:test), web build (npm run build), web tests (npm run test).
Plan: 80 parts: 68 planned, 12 dropped (10 parts dropped: nothing live reaches them); blockers: none. Written: docs/codelead/rebuild/rebuild-plan.json and docs/codelead/rebuild/rebuild-plan.md.
Dry run: refused:
  - The model of the default role, the preflight role, the architect role, the repair role does not answer: lmstudio (http://127.0.0.1:9/v1) answered no list of its models (…).
Evidence: .codelead/evidence/ev-20261120082318-c5dj0f.json (rebuild-init · failure).
Commit: efee766 "CodeLead: the rebuild's scaffold from dotnet-next-shadcn", with this machine's git user: the rebuild starts on a clean tree.
Not ready to run: answer the dry run's problems above; codelead rebuild run --dry-run in the folder checks again.
```

Read the new lines:

- `Ports:` the new API and the old application run side by side during the walk. The old application answers on 5080, so CodeLead moved the API from the template's 5080 to 5081. It reads the old ports from the program's address, the run profile, the project file and the launch settings.
- `Commit:` CodeLead committed what it made, with this machine's git user. The Rebuild stage then starts on a clean tree. `--no-commit` leaves the commit to you. When git has no user, the line says `Commit: not made: no git user is configured …`.
- `Run profile:` how CodeLead starts the new API and web application. An origin is the scheme, the host and the port of an address, so `http://localhost:5081` and `http://localhost:3000` are two origins. When the System Model reads a sign-in action in the old code, the rebuild signs users in with a cookie. With `--identity keep` and two origins, `init` writes one more check into the run profile. The check asks whether the API lets the web application's origin call it with the session's cookie. In that case, the line names the check before the caps, for example:

  <!-- described, not captured -->
  ```output
  Run profile: API npm run api:dev on http://localhost:5081, web npm run dev on http://localhost:3000; clock America/Los_Angeles; parking after 3 attempts; day reset node scripts/reset-sample-seed.mjs; the check "the SPA's origin may call the API with the session's cookie" (OPTIONS http://localhost:5081/api/me, before every walk); unattended caps 12 parts, $20, 480 minutes.
  ```

  See [Run profile](../reference/run-profile.md#checks).

This run had no model server, so the dry run is refused. With a model server that answers, the dry run passes. The last line then says `Ready: run codelead rebuild run in ../basic-store-rebuild.` With `--no-commit`, or when the commit was not made, it says `Ready: commit the folder, then run codelead rebuild run in …`.

### The template

The default template is `dotnet-next-shadcn`: an ASP.NET Core API with its tests, and a React web application, with their build and test scripts. CodeLead looks for the template by name in `greenfield.templates` of the configuration, then in the folder that `CODELEAD_TEMPLATES_DIR` names, then in `../codelead-templates/`. `--template` also takes a folder or a git address. See [Template manifest](../reference/template-manifest.md).

### The manifest

`docs/codelead/rebuild/rebuild.json` is the rebuild's manifest. It names the old application, its feature map and its web project. It names the targets, and it keeps the decisions. On the Basic Store sample:

```json
"source": {
  "repository": "../basic-store",
  "featureMap": "../basic-store/docs/codelead/rebuild/feature-map.json",
  "webProject": "webapp",
  "libraries": []
},
"targets": {
  "api": { "project": "api/Api/Api.csproj", "tests": "api/Api.Tests/Api.Tests.csproj", "stack": "aspnetcore", "framework": "net10.0", "routePrefix": "/api" },
  "web": { "path": "web", "stack": "react, next, tailwind, shadcn-ui, typescript" }
},
"decisions": {
  "identity": "keep: the new API keeps the old application's accounts and signs users in with ASP.NET Core Identity over the old user tables (codelead rebuild init --identity keep)",
  "database": "sample-seed: the new API starts on a sample database seeded at start-up (api/Api/sample-seed.db); the day reset (run.reset) deletes it before every journey, so that each journey starts on the seed",
  "channel": "none"
}
```

See [Rebuild manifest](../reference/rebuild-manifest.md).

> [!IMPORTANT]
> With `identity` keep, no part of the plan builds the directory that checks a password outside Development and the tests. The sign-in part registers a stand-in directory for Development and the tests only, so a build for production signs nobody in. The plan says so in a note on the part `sign-in:endpoint`, under **Notes on parts** in `rebuild-plan.md`. A person adds the real directory.

### The run profile

The manifest's `run` block is the **run profile**. It says how CodeLead starts the new API and the new web application, where each one answers, and which page says that it is ready. It also holds the clock's time zone, the day reset that puts the database back before each journey, and the caps of an unattended run. See [Run profile](../reference/run-profile.md).

### The journeys, from three sources

A **journey** is a path that a person takes through the application, step by step, with what each step must show. CodeLead merges the rebuild's journeys, `docs/codelead/rebuild/journeys.json`, from three sources:

1. **The Analyst's recordings**, from the Observe stage. A recording counts when its `journeys.json` passes the walk's check as a whole.
2. **The System Model's draft**, `journeys.draft.json`.
3. **The Explorer's stories**, from the Read stage: one journey for each screen that it reached.

When two journeys are the same, CodeLead keeps one. It judges this by where the journey ends and what its last step expects. The Analyst's journey wins over the model's, and the model's over the Explorer's. See [Journeys format](../reference/journeys-format.md).

`rebuild init` makes this copy once. The copy is the rebuild's own: you commit it, and you can edit it. When `codelead model` runs again in the old application, its draft can change. For example, a newer CodeLead drafts a sign-in for a site that signs in its own accounts. The copy does not change by itself.

The plan warns with `journeys-draft-moved` when the copy stands otherwise than the draft. Its note names the journeys that the draft writes otherwise, and the command to run. [`codelead rebuild journeys --refresh`](../cli/codelead-rebuild-journeys.md) replaces the copy's drafted journeys with the draft's. It keeps the Analyst's and the Explorer's journeys and the journeys written by hand. It also keeps a drafted journey that a person marked with a `note` or a `covers` list. Review the change with `git diff`, and commit it. An accepted part whose own steps the refresh changes is judged again, as [below](#when-a-parts-steps-change).

## The plan and its parts

The plan cuts the rebuild into **parts**. CodeLead computes it from the System Model, the feature map, the manifest and the journeys, with no model call. A part is one of these:

- a **prerequisite**, such as `identity-seam`, the place where the new code reads who the user is;
- a part of the **shell**: its start-up, its menu, its real-time channel;
- an **endpoint** of the new API, such as `clients.index:endpoint`;
- a **screen** of the new web application, such as `clients.index:screen`.

Each part has its needs: the parts that must land before it. Each part has a state:

| State | What it means |
|---|---|
| `planned` | To compose. A part whose accepted work is no longer in the code is `planned` again. So is a part that a person [reopened](#reopen-a-part). |
| `accepted` | It landed, and its walk accepted it on its own steps as they are now. When its own steps changed since that walk, it is [accepted before, to be judged again](#when-a-parts-steps-change). |
| `landed`, `landed-not-accepted` | It landed, and its walk did not accept it. `landed` still has repair rounds. |
| `satisfied` | It is in the code already, and its own steps pass in the walk. It is not composed again. |
| `not-walked` | It landed, and no walk could judge it. It is not finished. What needs it goes on, with a warning. When its own steps wait on a failing step of another part, the plan names that part. See [A step has two ends](#a-step-has-two-ends). |
| `parked` | Its attempts did not land. It waits for a person. |
| `dropped` | The old application does not need it. |

A **blocker** is a reason that stops the whole plan, such as a finding of the extraction gate that nobody explained. Nothing is composed while a blocker is open.

The plan drops what the old application does not need:

- a part of the shell with nothing to rebuild, such as a real-time channel that the old application never opens;
- a feature that the System Model calls dead, or that a decision drops;
- a feature that nothing live reaches. Its reason in the extraction gate says so, for example the ASP.NET Identity template's pages that no menu or link opens. The plan drops these whatever the `identity` answer is, and counts them in its summary.

[`codelead rebuild --plan`](../cli/codelead-rebuild.md) prints the plan. On the Basic Store sample:

```output
Rebuild plan of webapp: 80 parts (10 parts dropped: nothing live reaches them), 0 blockers. Nothing is written: /modernize --rebuild-plan writes the plan's files.
Parts, in order:
  1. identity-seam (prerequisite; lane foundations): planned
  2. shell:startup (the shell's start-up; lane foundations): dropped
  3. shell:menu (the shell's menu; lane foundations): planned
  4. shell:channel (the shell's channel; lane foundations): dropped
  5. clients.create-get:screen (screen; lane clients): planned
  6. clients.index:endpoint (endpoint; lane clients): planned
…
Lanes: foundations (4 parts) then 5 lanes: clients (14 parts, chain 4), orders (15 parts, chain 4), products (14 parts, chain 4), home (6 parts, chain 2), manage (27 parts, chain 5); critical path: identity-seam, shell:menu, manage.index:screen, manage.manage-logins:screen, manage.link-login:endpoint, manage.link-login:screen.
```

On this sample, 12 of the 80 parts are dropped. Two are parts of the shell with nothing to rebuild: the old layout's start-up calls no entry point, and the layout opens no real-time channel. The other 10 belong to seven Manage pages of the ASP.NET Identity template, such as `manage.add-phone-number-get`, that nothing live reaches.

### How a screen is made

A screen part asks for a page of the new web application. CodeLead reads the web application's layout from its packages and folders, and asks for the screen in that layout:

- **A Next.js app router**, as in the default template: the screen is the page file of its route's folder. The screen at `/clients/create` is `web/src/app/clients/create/page.tsx`, with its test beside it, such as `page.test.tsx`. No router file changes, because the folder is the route. A route parameter such as `{id}` becomes a folder `[id]`.
- **A React Router application**: the screen is a component in the existing pages folder. Its route is registered where the other pages are registered.

The rest of the rebuild reads the same layout. For example, [parity](walk-and-parity.md#parity-of-a-rebuild) reads an app router's routes from its folders. It reads a React Router application's routes from its route table.

### The site's root

The old application's root, `/`, often shows the page of one action, such as Home/Index, without a change of the address. The screen part of that feature then builds `/` too, so the rebuilt root shows the same screen. `rebuild-plan.md` says so on the part: `- The site's root: this screen is served at / as well (the old root, <PLACE>, reached home.index)`. When the old root sent a visitor to its sign-in page, the part's request asks the rebuilt root to do the same.

Sometimes the model cannot tell which page the root shows, for example when the old root reaches two such pages. Then the plan warns with `site-root`, and no part builds `/`. The root stays as the template left it. The same warning says why when the root's feature has no screen part in the plan.

### A step has two ends

A step of a journey often clicks a control on one page and reaches another page. For example, a step clicks the link "New client" on the home page, and reaches the screen at `/clients/create`. The plan reads both ends. The step judges the screen that it reaches, once the page that holds its control exists. So the plan builds the page with the control first: the screen at `/clients/create` needs the home screen. The home screen's request asks for the link "New client" to `/clients/create`, even before that screen exists.

Sometimes a screen lands, and the page with its control is not accepted yet. Its own step cannot pass, and nothing else fails. Then the part is **accepted on what regresses**: the walk found no regression and no new finding, and only that step waits. The parts that need it wait too. Once the page's part is accepted, the run judges the waiting part again by itself, with no model call.

A step of a journey also waits on the steps before it. When an earlier step fails, the walk cannot reach the later ones. Then a part whose own steps come later lands, and no walk judges it. For example, the screen at `/clients/index` lands, and its step waits on the sign-in step, which fails. The plan names the part whose step it waits on, and that part's state:

<!-- described, not captured -->
```output
landed (ev-<ID>), and its acceptance was not judged (own-steps-unreachable): its own step clients.edit-post-3 waits on clients.edit-post-2, which fails, a step of sign-in:screen (accepted before, its steps changed since: to be judged again); the run judges it again by itself once that step passes
```

Such a part is not finished, and the run counts it as work left. Once the stored walk of the tree reaches its steps, the run judges it again by itself, with no model call. When the plan cannot tell which part owns the step, it says so. A page and a step follow one rule: a part that another part keeps from being judged waits on that part.

When the plan cannot tell which page a step acts on, it warns with `step-page-unknown`. A title that names the control's place, or a `covers` list in the journey, settles it. See [Journeys format](../reference/journeys-format.md#steps).

### When a part's steps change

A part stays accepted only while the walk that accepted it judged its own steps as the journeys write them now. A later stored walk of the tree, in which those steps pass, counts too. The steps can change in two ways:

- The journeys change. For example, `codelead rebuild journeys --refresh` brings in journeys that begin with a sign-in.
- The plan gives the part new steps. For example, the sign-in's screen now owns the typed sign-in of every journey.

Then the part is **accepted before, its steps changed since: to be judged again**. The plan warns with `steps-changed`. The parts that need it wait. The run judges the part again by itself first, with no model call:

- When its own steps pass, it stays accepted.
- When they fail, the judgement offers no repair round. The plan plans the part again: `accepted (ev-<ID>) by a walk of its earlier steps; its own step now, <STEP>, fails in the walk of <TIME>: revise it`.

The next composition revises in place what the part built, as for a [reopened part](#reopen-a-part). Its request says why, and quotes the walk's words for the failing steps. The check of accepted parts protects what the part built until its revision lands.

### Lanes

The parts are grouped in **lanes**: first the **foundations**, then one lane for each area of the application. A lane's **chain** is its longest line of parts that wait on each other. The **critical path** is the longest chain of the whole plan.

A run claims a lane, not a single part. Two clones of the rebuild can run two lanes at the same time, each with its own walk database. A lane that another clone claims is passed by.

### The dry run

Before a real run, check the plan in two ways. Neither calls a model.

- `/modernize --transform T9 --next --dry-run`, in the rebuild's chat, composes every planned part and checks each quote and name against the code. On the Basic Store sample it ends with `Dry run: 68 parts composed (17 as if a prerequisite the plan plans had landed), 325,919 characters in 68 requests; 35 mismatches; 0 refused. Not composed: 12 dropped.`, with no part refused. Before the first part lands, a mismatch usually means a file that a part creates and that is not there yet.
- [`codelead rebuild run --dry-run`](../cli/codelead-rebuild-run.md) checks the run profile. It checks that each role's model server answers, that the walk has its passwords, and that the plan has no blocker.

> [!NOTE]
> A part waits on a real-time channel only when its old action pushes to clients through a hub, for example with `Clients.All.<method>(…)` or an `IHubContext`. A data set that happens to be named `Clients`, such as the Basic Store's `db.Clients`, is data, not a channel. So the dry run refuses none of the sample's parts.

## The unattended run

[`codelead rebuild run`](../cli/codelead-rebuild-run.md) runs the plan with no person. The Rebuild stage runs it on every lane. For each part whose needs are met:

1. **Compose**: a session asks the model for the part's code, as [`/implement`](../chat-commands/implement.md) does.
2. **Validate**: the gate of the rebuild builds the API and the web application, and runs their tests.
3. **Walk**: [the walk](walk-and-parity.md) follows the part's steps of the journeys on the running application.
4. **Judge**: the part is accepted, repaired, or parked. A part whose own step waits on another part's page is accepted on what regresses, as [above](#a-step-has-two-ends).

A part with no step of its own, such as an endpoint, is accepted on what regresses: no step that passed before fails. Its acceptance line says what the walk exercised when some steps fail or are blocked:

<!-- described, not captured -->
```output
Acceptance (walk r0): no step of the journeys covers this request; nothing that passed before fails, in a walk where 9 steps pass, 9 fail and 21 are blocked: none is this part's own; accepted on what regresses, and to be judged by its own steps when the journeys reach it: first walked with clients.create-post-3.
```

The run then says `accepted on what regresses (no step of its own yet)`, and never `accepted at the first walk`. The run summary counts the two kinds of acceptance apart.

When a check fails, the **model ladder** decides what comes next. First repair rounds on the compose model. Then the part moves to a stronger repair model. Then an intervention: an agent on a frontier model takes the part over once, within limits of turns, minutes and dollars. When that fails too, the part is **parked** for a person, and the run continues with the next part. See [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md).

The run stops at the end of the plan, at a cap, when the file `.codelead/STOP` appears, or on Ctrl+C. The part that is running finishes first.

> [!WARNING]
> The run calls models for every part, and answers `Apply this patch?` itself. With a paid provider, every part costs money. The caps of a new rebuild are 12 parts, 20 US dollars and 480 minutes, with 8 repair rounds for each session. In an earlier rebuild of another application, one part cost about 0.55 US dollars to compose and accept.

### What an accepted part built stays built

A later part must never undo an accepted part. Before any patch lands, the patch check checks it against every accepted part of the plan. It refuses a patch that:

- removes an endpoint that an accepted part built;
- removes the page or the route of an accepted screen;
- removes a test that an accepted part added, or changes what the test sends or asserts.

The refusal says what the patch breaks, and how to add the change beside it. The model gets these words for its next patch. This holds for every patch: the first one, its retries, every repair round and the intervention.

For example, the part `clients.create-get:endpoint` builds `GET /api/clients/create`, and its walk accepts it. Later, the part `clients.create-post:endpoint` builds `POST /api/clients/create`, at the same path. Its request already says how:

<!-- described, not captured -->
```output
The API already answers GET /api/clients/create (the part clients.create-get:endpoint, in ClientsController): add the POST as a second action beside it; that action and its tests stay as they are.
```

A patch that replaces the GET action with the POST action is refused before it lands:

<!-- described, not captured -->
```output
accepted_parts: the patch breaks what the accepted part clients.create-get:endpoint built (ev-…): it removes GET /api/clients/create (after it, no action of the API answers it; …/ClientsController.cs declared it at line …, the action Create()); it changes what 2 tests the part added in …/ClientsCreateGetContractTests.cs send (…: GET /api/clients/create becomes POST /api/clients/create). POST /api/clients/create is another endpoint than GET /api/clients/create, which the part clients.create-get:endpoint built and its tests prove: add a second action and a new test file; leave Create() and ClientsCreateGetContractTests.cs as they are.
```

A patch that adds the POST as a second action, with a test file of its own, passes. The GET action and its tests stay as they were.

Two more rules:

- **A part that landed and is not accepted yet** is warned for, not refused. Its work is not proven yet, and a later part may be the one that fixes it.
- **An accepted part whose endpoint, screen's route or tests are gone from the code** is accepted no more. This can happen after a person's own `/implement` in the rebuild: the check warns for it, and does not refuse it. The plan plans the part again, to be rebuilt. It names the part whose landing removed the work, when the commits tell. That other part keeps its own state, with a note. The run says it before it starts: `Note: <PART_ID> is accepted no more (<RECORD>): …; it is planned again, to be rebuilt.`

### Parked parts

A parked part is a card on the rebuild's board, with the reason. It is your turn. Read the card, then fix the part by hand, or plan it again with `/modernize --transform T9 --reopen <PART_ID> --reason <WHY>`, and run again. See [`/modernize --transform T9`](../chat-commands/modernize.md#--transform-t9).

A part is parked after 3 attempts that did not land, by default. It is parked at once when the patch check refused two attempts in a row with the same problems, on the same request. A third attempt would meet the same check. The reason then says `the patch check refused the same change twice: <problem>`. See [Run profile](../reference/run-profile.md#the-keys-of-run).

### Reopen a part

Sometimes you want a part composed again, although it holds work already. For example, its request was composed from an older reading of the System Model, and the plan warns with `composed-older-reading`. Reopen the part, with your reason:

```console
/modernize --transform T9 --reopen sign-in:screen --reason "the model read again: ReturnUrl is a hidden field"
```

<!-- described, not captured -->
```output
Rebuild (T9) reopen: sign-in:screen (screen): /account/sign-in
It is accepted (accepted by its walk (ev-<ID>)). Reason: the model read again: ReturnUrl is a hidden field
Evidence record (reopened): <YOUR_REBUILD>/.codelead/evidence/ev-<ID>.json
Nothing in the tree was changed: what sign-in:screen built stands as it is, and its next composition revises it in place. The parts that need it keep their states.
Part sign-in:screen: planned (planned again by a person (2026-11-24): the model read again: ReturnUrl is a hidden field; accepted before (ev-<ID>)). Next part: sign-in:screen (/account/sign-in). It is the same part: a person reopened it, and the next --next composes it again, revising in place what it built.
```

What happens then:

- You can reopen a part that holds work: accepted, landed and not accepted, landed and not walked, or satisfied. A part that is accepted on what regresses counts too.
- A parked part is reopened as before. Its attempts are counted again from zero.
- A planned part has nothing to reopen, and neither has a dropped part. CodeLead refuses both, says why, and writes nothing.
- The reopen writes one evidence record. It calls no model, and it changes nothing in the tree.
- The plan reads the part as planned again, until it lands again. `rebuild-plan.md` lists it under `Reopened by a person`.
- The next composition revises in place what the part built. Its request quotes your reason once, as a reason and not as an instruction.
- The parts that need it keep their states. Their need reads `met before, reopened since`, and nothing else is planned again.
- The check of accepted parts still protects what the part built, until its revision lands. Its own revision can change its own code and tests.
- The old landing is not judged again: `--judge-again` refuses it. The walk of the next landing judges the part anew.
- [`codelead rebuild run`](../cli/codelead-rebuild-run.md) takes the part like any planned part. The run summary lists it under `Reopened by a person`.

## Details

- **Folders**: the rebuild is `../<folder>-rebuild` by default, the `rebuildFolder` answer of the program. It is a git repository of its own, with CodeLead initialized in it. Its first commit is the scaffold, `CodeLead: the rebuild's scaffold from <template>`, unless you give `--no-commit`.
- **The ports**: the API and the web application never take a port that the old application answers on, nor each other's. A moved port is written into the API's launch settings and the web's dev script, or set as `ASPNETCORE_URLS` or `PORT` in the run profile.
- **The answers**: `--identity keep|drop`, `--database <CONNECTION_NAME>|sample-seed`, `--framework net10.0|net8.0`, `--template <NAME|PATH>`. A second `init` into a folder that it made says so and changes nothing.
- **The plan's files**: `docs/codelead/rebuild/rebuild-plan.json` and `rebuild-plan.md`, written by `rebuild init` and by `/modernize --rebuild-plan`.
- **The caps**: `run.unattended` in the manifest (`maxParts`, `maxCostUsd`, `maxMinutes`, `repairsPerSession`), or `--max-parts`, `--max-cost-usd` and `--max-minutes` for one run.
- **The ladder**: `run.models` in the manifest (`compose`, `repair`, `intervene`, `escalate`). With no `models` block, the ladder is off.
- **The run summary**: `.codelead/rebuild-run-<stamp>.md` and `.json`: each part, its outcome and its cost, and the accepted parts whose work no longer holds in the code. It also lists the parts that a person reopened. See [Rebuild run summary](../reference/rebuild-run-summary.md).
- **Planned again**: the plan's warning `contract-broken` names an accepted part whose work is gone from the code. The warning `contract-changed` names the identity seam when a cookie session across two origins needs a CORS policy that the seam's code lacks. A CORS policy is the API's rule for which origins may call it. The plan then revises the seam in place, once. When the revision landed and the code still lacks the policy, it is a person's turn. The same warning names the sign-in's endpoint when the API refused the walk's own account and its fake directory requires roles. The revision then makes roles optional, once.
- **What the check of accepted parts reads**: the plan, the evidence records and git, with no model call. It reads tests written in C# (xUnit, NUnit and MSTest) and in TypeScript or JavaScript (`it` and `test`). A changed test file in another language is a warning. The check never runs outside a rebuild.
- **The report**: `codelead rebuild-report` writes `.codelead/rebuild-report.html`, which the wizard shows.
- **Evidence**: `rebuild init` writes one record, `rebuild-init`. Each part's session writes its own.

## What it is not

- **A rebuild does not change the old code**. It reads the old application, and writes only in its own folder.
- **A rebuild is not finished when the run ends**. It is finished when [parity](walk-and-parity.md#parity-of-a-rebuild) says so, after a clean walk.
- **A parked part is not a failure of the run**. It is a part that needs a person, and the run says so.

## Related content

- [The eight stages](the-eight-stages.md#6-set-up-the-rebuild)
- [`codelead rebuild init`](../cli/codelead-rebuild-init.md), [`codelead rebuild`](../cli/codelead-rebuild.md) and [`codelead rebuild run`](../cli/codelead-rebuild-run.md)
- [`/modernize --rebuild-init`](../chat-commands/modernize.md#--rebuild-init) and [`--rebuild-plan`](../chat-commands/modernize.md#--rebuild-plan)
- [The System Model](system-model.md)
- [The walk and parity](walk-and-parity.md)
- [Unattended runs and the run lock](../concepts/unattended-runs.md)
- [Rebuild manifest](../reference/rebuild-manifest.md) and [Run profile](../reference/run-profile.md)
