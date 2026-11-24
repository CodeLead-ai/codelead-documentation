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
> With `identity` keep, the sign-in part registers a stand-in directory for Development and the tests only. The part `user-directory` builds the real directory, which checks a password against the old user table. Until it is accepted, a build for production signs nobody in. When CodeLead cannot read the old password check, there is no such part, and a person adds the real directory. See [The user directory](#the-user-directory).

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
- the **data access**, `data-access`, the way the new API reaches the old database;
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

A step with no action of its own can never pass, when it expects to reach another page. The draft writes such a step only when it can name nothing that the walk can find. See [Steps a person completes](../reference/journeys-format.md#steps-a-person-completes). No part is judged on such a step:

- The plan leaves it out of every part's own steps, and warns with `person-step`.
- The part's `Steps` line says `for a person to complete (no action of its own, not judged): <STEP>`.
- A part that waits behind such a step says so, and names the way out.
- A landed part whose walk failed only on such a step is judged again by the run itself, with no model call.

### What the old shell shows

An old layout often shows more than its controls. For example, it shows the signed-in name beside "Sign out": `<span>@User.Identity.Name</span>`. The System Model records where. The shell's menu part asks for it, beside the table of controls. On the Basic Store sample:

<!-- described, not captured -->
```output
What the old shell shows of the session: the signed-in user's name (`User.Identity.Name`, BasicStore.Web/Views/Shared/_Layout.cshtml:22), shown when signed in. Render it in the menu as text while someone is signed in, and no name for a visitor: read it from the field userName of the answer of GET /api/me, the API's caller-identity seam (the part identity-seam), through the API client. The walk's sign-in steps look for that name on the page.
```

A menu can land before its request asked for the name. CodeLead then reads the rebuilt shell's files, with no model call. They are its layout and every file it imports, by a relative path, the tsconfig's `paths` or `@/`. When no file shows the name, the plan plans the menu again, to be revised in place:

<!-- described, not captured -->
```output
accepted (ev-<ID>) before its request asked it to show the signed-in name the old shell shows (`User.Identity.Name` at BasicStore.Web/Views/Shared/_Layout.cshtml:22): no file of the shell renders the name's field (userName) as text: the menu shows no signed-in name: planned again, to be revised in place
```

Its next request names the files the menu built, and the patch is shown them. See [What a revised part is shown](#what-a-revised-part-is-shown).

When CodeLead cannot tell from the code, the menu gets a note for a person. An example is a name handed to another component.

A step can fail only on that name. For example, the sign-in step lands at `/` and finds no name. The walk says which of a step's expectations failed. The sign-in screen cannot add the name, so it gets no repair round. It is accepted on what regresses, and it waits on the menu. Once the menu shows the name, the run judges the screen again by itself, with no model call. A step that fails on its address as well is repaired as before.

### Where the sign-out goes

The menu's request also says where its sign-out sends the user. CodeLead reads it from the old sign-out action's redirect, and translates it to this application's route. On the Basic Store sample, the old action ends with `return RedirectToAction("SignIn")`:

<!-- described, not captured -->
```output
The sign-out: "Sign out" is a control that calls POST /api/account/sign-out through the API client (…), not a link to a page; once it has answered, the SPA goes to /account/sign-in, where the old sign-out sends the user (GET /Account/SignIn, its redirect at BasicStore.Web/Controllers/AccountController.cs:61).
```

The rule:

- An old sign-out that redirects to the site's root goes to `/`.
- An old sign-out whose redirect CodeLead cannot read goes to `/`. The request then says `the old sign-out's destination was not read`, and why.
- A menu that landed before its request said this is read again, with no model call. When its sign-out goes elsewhere, the plan plans the menu again, to be revised in place.
- A menu that lacks the signed-in name too gets one revision for both.
- When CodeLead cannot tell from the code where the rebuilt sign-out goes, the menu gets a note for a person.

### When a part's steps change

A part stays accepted only while the walk that accepted it judged its own steps as the journeys write them now. A later stored walk of the tree, in which those steps pass, counts too. The steps can change in two ways:

- The journeys change. For example, `codelead rebuild journeys --refresh` brings in journeys that begin with a sign-in.
- The plan gives the part new steps. For example, the sign-in's screen now owns the typed sign-in of every journey.

Then the part is **accepted before, its steps changed since: to be judged again**. The plan warns with `steps-changed`. The parts that need it wait. The run judges the part again by itself first, with no model call:

- When its own steps pass, it stays accepted.
- When they fail, the judgement offers no repair round. The plan plans the part again: `accepted (ev-<ID>) by a walk of its earlier steps; its own step now, <STEP>, fails in the walk of <TIME>: revise it`.

The next composition revises in place what the part built, as for a [reopened part](#reopen-a-part). Its request says why, and quotes the walk's words for the failing steps. The check of accepted parts protects what the part built until its revision lands.

### What a revised part is shown

A part can be composed again to be revised in place. A person reopened it, its changed steps failed, its contract asks for more, or its endpoint moved. The model must then see what the part built, not guess it.

The part's evidence records name the files its patches changed. CodeLead reads them, keeps the files the tree still holds, and names them in the request:

<!-- described, not captured -->
```output
What it built lives in web/src/components/shell-menu.tsx, web/src/app/layout.tsx and web/src/components/shell-menu.test.tsx. The context below holds these files as they are now: revise them there, and write nothing from memory.
```

The session then hands those files to the patch, as they are now, and says so:

<!-- described, not captured -->
```output
Revised in place (its records' patches, from ev-<ID>): the patch's context holds what it built, as it stands now: web/src/components/shell-menu.tsx, web/src/app/layout.tsx and web/src/components/shell-menu.test.tsx.
```

The rule for what the patch is shown:

- First the files the revision's own words name, then the code, then its tests.
- At most 4 files, and 40,000 characters in all. A file over 30,000 characters is not shown whole.
- A file it does not show whole is still named, as not shown, with the limit.
- A file the part built and someone deleted since is not named.

These sentences only describe where the work lives. The gates owe the patch none of these files, so a revision may leave one of them alone. On a revision, the menu's request no longer says it replaces the shell's navigation, and its tests are the part's own.

### The data access

An endpoint can answer the right shape over no data. For example, a list endpoint returns `new List<ClientDto>()`, and its contract test checks only the answer's shape. The walk then passes the step "click Clients" on an empty list.

So the plan has a part `data-access` when two things hold. The manifest's `database` decision gives the new API a database. And the System Model says that old actions read or write it. Every endpoint whose old action reads or writes the database needs this part. An endpoint that reads no data, such as an empty form, does not need it.

How the part works depends on the manifest's decision `dataAccess`. There are two choices.

**`port`, the default for an old application on Entity Framework 6.** The rebuild ports the data access to EF Core inside the new API. The API references none of the old projects, and the old application is not changed for it. The part builds three things:

- an EF Core context of the API's own, the port of the old one, such as `api/Api/Data/StoreContext.cs`. It holds an entity for each table that the plan's endpoints reach. Each entity is mapped to its real table with `ToTable`. The entities go in the folder `api/Api/Data`, one file each or beside the context, as the model writes them;
- the EF Core provider's package, and the context's registration in `Program.cs`;
- with `sample-seed`, the sample database, made at start-up from copies of the old application's schema and seed scripts.

The copies live in the rebuild, in the API project's folder `SampleSeed`, so the new application builds without the old repository beside it. `codelead rebuild init` makes them. For a rebuild made before, CodeLead makes them when it composes `data-access`, and the part's landing commits them. A note beside them, `copied.json`, keeps where each came from and its hash. The copies keep the statements and leave the comments out. Some seeds write a password in clear text in a statement. Such a seed is not copied: the part waits, and its words say what a person must do. When the old script changes later, the plan says so:

<!-- described, not captured -->
```output
The source's schema changed since the copy: the old application's database/schema.sql is not the file it was (copied as api/Api/SampleSeed/schema.sql at 2026-11-24T12:00:00.000Z). Copy it again: …
```

The sample database file itself, such as `api/Api/sample-seed.db`, is never tracked. The API makes it at start-up, and the day reset deletes it before every journey. A tracked file that disappears while the walk runs changes the tree, and no walk can be made. So CodeLead writes its ignore lines into the rebuild's `.gitignore`, under a comment that says they are CodeLead's. It does so at `codelead rebuild init`, and at the start of every run or `--next`. When the file is tracked already, CodeLead takes it out of the index by its path, `git rm --cached -- <path>`. It commits that with the ignore lines, in a commit of its own. The file on disk stays, and the line says so:

<!-- described, not captured -->
```output
Note: The sample database api/Api/sample-seed.db was tracked, and the day reset deletes it before every journey: CodeLead ignores it in .gitignore and took api/Api/sample-seed.db out of the index, in a commit of its own (…); the file on disk stays.
```

A part's landing commits what its patches changed or created, and CodeLead's own files, and nothing else. A file that the gate made, such as a test run's results, is left out of the commit. The line names it: `Left out of the commit: api/Api.Tests/TestResults, which the gate made (not the patch's): …`.

No request of the rebuild names a path outside it. The old application's files are named by their place there, such as `BasicStore.Web.Data/DAL/StoreContext.cs in the old application`.

Its request shows the old context and entities, as an endpoint's request shows the old action. It also lists the places where the port is not mechanical:

- EF6 named the tables by its conventions, and EF Core takes the set's name. So each entity names its table.
- The keys and relationships of the old `OnModelCreating` are ported one for one. For example, `HasRequired(..).WithMany(..)` becomes `HasOne(..).WithMany(..).IsRequired()`.
- EF Core loads no navigation by itself. So each endpoint uses `Include` for what it reads.
- Nothing creates or migrates the schema, because the old context turned its initializer off.
- A query that EF Core cannot translate runs on the rows read, as the old code does.

A large application has many entities, so one session ports at most eight. The others come with the endpoint that first reaches each one. That endpoint's request shows the entity's old source and asks to add it to the context.

**`same-library`, a person's choice.** The rebuilt API references the old application's data library and reaches the data through it. The part adds the reference, registers the provider by code and registers the library's context. Sometimes the old application stands in the way. Its library may target the .NET Framework alone, such as `net48`. Or its context may read its connection only by name from `Web.config`. Then the part waits, and its note says what to run in the old application's folder:

<!-- described, not captured -->
```output
The data access cannot be built as the old application stands: BasicStore.Web.Data targets net48 only, which an API on net10.0 cannot reference: in the old application's folder (../basic-store), codelead modernize run in-place runs T1 for the rebuild …
```

The Upgrade in place stage then runs [T1](transforms.md) on that library, under the old application's pins. See [the eight stages](the-eight-stages.md#4-upgrade-in-place).

[`codelead rebuild init`](../cli/codelead-rebuild-init.md) writes the decision. You can also answer `dataAccess=port` or `dataAccess=same-library` with `codelead modernize answer`. A rebuild made before the decision existed records none. The plan then reads it as `port` when the old application is on Entity Framework 6, and says so in a line:

<!-- described, not captured -->
```output
The manifest records no data-access decision (a rebuild made before it existed): the old application is on Entity Framework 6, so the plan reads it as port: …
```

CodeLead also reads each rebuilt endpoint's code, with no model call. With `port`, it looks for the API's own EF Core context in the action, or in a member of the controller that the action calls. With `same-library`, it looks for the old library's context or types. An endpoint that reaches neither is **accepted on its contract, and it reads no data yet**. The part's `Data` line in `rebuild-plan.md` says so:

<!-- described, not captured -->
```output
- Data: the old action reads Clients; accepted on its contract, and it reads no data yet (api/Api/Controllers/ClientsController.cs:33)
```

Once `data-access` has landed, the plan plans such an endpoint again, to be revised in place. Its request opens with `(revised in place for its data: it was accepted on its contract and read no data)`. It quotes the stub's first line, and it asks for a test of a row that the seed holds. An empty answer then fails the test. When a revision whose request asked for the data still reads none, the part gets a note: a person's turn.

With `port`, the revised endpoint's request says it reaches the same rows over the same database, through the rebuild's EF Core context. It no longer says "the same library calls".

An endpoint's request also quotes what the old action uses and its quoted lines do not declare. Examples are a constant of its class, a static field, and another type's constant, such as `Order.Placed`. Each one comes with its declaration, its file and its line, at most six of them. A value that the old configuration gives is said as such, with what `Web.config` sets. For example:

<!-- described, not captured -->
```output
LowStockBelow (BasicStore.Web/Controllers/HomeController.cs line 14):
private const int LowStockBelow = 5;
```

A field that holds a dependency, such as the data context, is never quoted: the API has its own.

A screen judged while its endpoint read no data is judged again by the run itself, with no model call. When its own steps fail then, it is planned again, as [when a part's steps change](#when-a-parts-steps-change).

A screen's own step may need a row that an empty list cannot give. For example, the step clicks "Edit" on the clients' list, and the list's endpoint reads no data yet. CodeLead reads, with no model call, that the old view writes this control once per row. Such a step is owed to the data, and no repair round runs for it. The acceptance line says `it needs rows that clients.index:endpoint does not read yet`. The screen waits on that endpoint. Once the endpoint reads the data, the run judges the screen again by itself. When CodeLead cannot tell whether the step needs a row, the line says so, and the step is judged as before.

The plan says the count at its top, so that the accepted parts never overstate:

<!-- described, not captured -->
```output
Data access: of 17 endpoints whose old action reads or writes the database, 0 read the data in the code; 17 are accepted on their contract and read no data yet (home.index:endpoint, clients.create-post:endpoint, clients.details:endpoint and 14 more); the part data-access is planned.
```

An endpoint that reads no data is never finished. The run takes it as work left, and the run summary, the stage and the audit pack say the same count.

### The user directory

A **user directory** answers one question: is this account and this password right? The old application has one. For example, the Basic Store's sign-in reads a row of `Users` and calls `PasswordHasher.Verify`.

The sign-in part builds a stand-in directory, `FakeUserDirectory`, for Development and the tests. Its accounts come from configuration, not from the database. So with `--identity keep`, the plan adds the part `user-directory`. It builds the real directory, `UserDirectory`, behind the same interface, `IUserDirectory`.

The part needs two others: `data-access`, because it reads the accounts through the rebuild's data access, and `sign-in:endpoint`, which declares the interface. It waits until both have landed.

**The password check is read from the old code, never invented.** CodeLead reads it with no model call:

1. It finds the call of the old sign-in action that passes the typed password, such as `PasswordHasher.Verify(model.Password, user.PasswordHash, user.PasswordSalt)`.
2. It finds where the old repository declares that member, and quotes its file whole.
3. It checks that the old check calls nothing but .NET's own base library, such as `Rfc2898DeriveBytes` and `Convert`.

The part's request shows the old sign-in action and the old check's source. It asks for a port that gives the same answer for the same stored hash and salt. That means the same algorithm, the same iteration count, the same key length, the same encoding and the same comparison. A default of the old framework is written out. For example, an old constructor called without a hash algorithm takes HMAC-SHA1 on the .NET Framework.

When CodeLead cannot read the check to its source, the plan has no `user-directory` part. For example, the old sign-in calls `Membership.ValidateUser`, a membership provider set up outside the code. Other examples are a library with no source here, or a directory of the operating system's accounts. The note on `sign-in:endpoint` then says what was read and what was not:

<!-- described, not captured -->
```output
No part of this plan builds the directory that checks a password outside Development and the tests: … the old sign-in's password check cannot be read to its source, so the plan builds no directory of its own: CodeLead never invents a password check. Read: the old sign-in action Account.SignIn (…); the call that checks the password, `Membership.ValidateUser(email, model.Password)` (…). Not read: Membership.ValidateUser is not declared in the old repository …
```

It is a person's turn: port the check and record it in the manifest under `decisions.identity.directory`.

**How the two directories are chosen at start-up.** `Program.cs` makes the choice once, and a test can see it:

- In Development, the stand-in directory stays, unless the setting `SignIn:Directory` is `real`.
- In every other environment, the real directory is registered, and the stand-in never is.

**Roles.** The real directory answers the roles the old application gives a user. When the old application names no role, as the Basic Store does, it answers none. An account without roles still signs in.

**The proof, with no password in the repository.** CodeLead never writes a password into a request, a file, a log or a record. So the part's tests make their own rows:

- A test inserts a row for a made-up account, its hash made by the port from a made-up password. That account signs in.
- A wrong password and an unknown account are refused, with the old refusal's words.
- A copy of the old check's own source, in the tests, verifies what the port makes for 20 made-up passwords.

Then the part's walk proves that a real account signs in. CodeLead starts the API in Development with `SignIn__Directory=real`, and gives the stand-in no account. Its own sign-in check then signs the walk's account in, with the password from the walk's credentials only. Only a 2xx answer passes. Two more checks follow, each with a password made at random and never written:

- the walk's account with a wrong password must be refused (401 or 403);
- an account that no database holds must be refused too.

So a directory that signs every account in fails its walk. The acceptance line says which directory the check went through:

<!-- described, not captured -->
```output
Acceptance (walk r0): accepted. … CodeLead's own sign-in of the walk's account <ACCOUNT> went through the real directory UserDirectory before this walk, and it refused a wrong password and an account no database holds (…).
```

Every other walk stays on the stand-in directory. Once the part is accepted, CodeLead records the directory in the manifest, with `builtBy: "user-directory"`. The note "a build for production signs nobody in" then goes. The plan, the run summary, the stage and the audit pack say what was built and how it was proved:

<!-- described, not captured -->
```output
User directory: the part user-directory is accepted (ev-<ID>): UserDirectory behind IUserDirectory, its password check ported from PasswordHasher.Verify (…). Proved by its contract tests … and by its walk (ev-<ID>): …
```

With `--identity drop`, or a directory a person recorded, there is no such part.

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

Sometimes nothing planned has its needs met, and the planned parts wait on a part that landed and was not accepted. Then the run judges that part again by itself first. It makes no model call and offers no repair round. The judgement's record says what keeps the part. For example, CodeLead's own sign-in check refuses the walk's account. The next plan reads that record and plans the sign-in's endpoint again, with no person. The run does this once for each landing.

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

What a part is held to is its **baseline**: the tree that its acceptance judged.

- A part accepted at its own walk is held to its last checkpoint.
- A part accepted when the run judged it again, on a tree that moved since its landing, is held to that tree. For example, another part's repair gave the Edit links of `clients.index:screen` their id, before the run accepted that screen. That changed test is what the screen is held to.
- So a part is never accepted and planned again in the same breath, for a change that came before its acceptance.
- A landing is named only when its own commit broke the work: it held at the commit before the landing, and not after it. When no landing of the rebuild did that, the plan says `a change made outside the rebuild's parts`.
- Two parts that share a file cannot plan each other again in a loop. Each is held to the tree that accepted it, and the check refuses a later patch that breaks the other.

### Parked parts

A parked part is a card on the rebuild's board, with the reason. It is your turn. Read the card, then fix the part by hand, or plan it again with `/modernize --transform T9 --reopen <PART_ID> --reason <WHY>`, and run again. See [`/modernize --transform T9`](../chat-commands/modernize.md#--transform-t9).

A part is parked after 3 attempts that did not land, by default. It is parked at once when the patch check refused two attempts in a row with the same problems, on the same request. A third attempt would meet the same check. The reason then says `the patch check refused the same change twice: <problem>`. See [Run profile](../reference/run-profile.md#the-keys-of-run).

Sometimes the tree already holds what a part asks for. An earlier part's repair built it, for example. The model then says so, and gives no diff, or a patch that changes nothing. CodeLead records that the attempt found **nothing to change**. The plan then judges the part by the stored walk of the tree as it stands, with no model call:

- When the part's own steps pass, it is **present by the walk**. The run says `satisfied: present by the walk of <TIME>: …`, and composes it no more.
- When they do not pass, the part is parked after that one attempt. The reason says what the walk says:

<!-- described, not captured -->
```output
parked after 1 attempt that did not land (the last: ev-<ID>, the model found nothing to change ("…both files already satisfy every requirement…"), and the walk of the tree as it stands (<TIME>) does not pass its own step: products.create-post-5 fails (on /products/index, expected ^/products/create$ (clicked "Add the product"))): a later rule or a person reopens it
```

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
