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
- You know how the plan cuts the work into parts and lanes.
- You know what the unattended run does, where it stops, and what a parked part is.

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

### The run profile

The manifest's `run` block is the **run profile**. It says how CodeLead starts the new API and the new web application, where each one answers, and which page says that it is ready. It also holds the clock's time zone, the day reset that puts the database back before each journey, and the caps of an unattended run. See [Run profile](../reference/run-profile.md).

### The journeys, from three sources

A **journey** is a path that a person takes through the application, step by step, with what each step must show. CodeLead merges the rebuild's journeys, `docs/codelead/rebuild/journeys.json`, from three sources:

1. **The Analyst's recordings**, from the Observe stage. A recording counts when its `journeys.json` passes the walk's check as a whole.
2. **The System Model's draft**, `journeys.draft.json`.
3. **The Explorer's stories**, from the Read stage: one journey for each screen that it reached.

When two journeys are the same, CodeLead keeps one. It judges this by where the journey ends and what its last step expects. The Analyst's journey wins over the model's, and the model's over the Explorer's. See [Journeys format](../reference/journeys-format.md).

## The plan and its parts

The plan cuts the rebuild into **parts**. CodeLead computes it from the System Model, the feature map, the manifest and the journeys, with no model call. A part is one of these:

- a **prerequisite**, such as `identity-seam`, the place where the new code reads who the user is;
- a part of the **shell**: its start-up, its menu, its real-time channel;
- an **endpoint** of the new API, such as `clients.index:endpoint`;
- a **screen** of the new web application, such as `clients.index:screen`.

Each part has its needs: the parts that must land before it. Each part is `planned`, `dropped`, `accepted` or `parked`. A **blocker** is a reason that stops the whole plan, such as a finding of the extraction gate that nobody explained. Nothing is composed while a blocker is open.

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
4. **Judge**: the part is accepted, repaired, or parked.

When a check fails, the **model ladder** decides what comes next. First repair rounds on the compose model. Then the part moves to a stronger repair model. Then an intervention: an agent on a frontier model takes the part over once, within limits of turns, minutes and dollars. When that fails too, the part is **parked** for a person, and the run continues with the next part. See [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md).

The run stops at the end of the plan, at a cap, when the file `.codelead/STOP` appears, or on Ctrl+C. The part that is running finishes first.

> [!WARNING]
> The run calls models for every part, and answers `Apply this patch?` itself. With a paid provider, every part costs money. The caps of a new rebuild are 12 parts, 20 US dollars and 480 minutes, with 8 repair rounds for each session. In an earlier rebuild of another application, one part cost about 0.55 US dollars to compose and accept.

### Parked parts

A parked part is a card on the rebuild's board, with the reason. It is your turn. Read the card, then fix the part by hand, or plan it again with `/modernize --transform T9 --reopen <PART_ID> --reason <WHY>`, and run again. See [`/modernize --transform T9`](../chat-commands/modernize.md#--transform-t9).

## Details

- **Folders**: the rebuild is `../<folder>-rebuild` by default, the `rebuildFolder` answer of the program. It is a git repository of its own, with CodeLead initialized in it. Its first commit is the scaffold, `CodeLead: the rebuild's scaffold from <template>`, unless you give `--no-commit`.
- **The ports**: the API and the web application never take a port that the old application answers on, nor each other's. A moved port is written into the API's launch settings and the web's dev script, or set as `ASPNETCORE_URLS` or `PORT` in the run profile.
- **The answers**: `--identity keep|drop`, `--database <CONNECTION_NAME>|sample-seed`, `--framework net10.0|net8.0`, `--template <NAME|PATH>`. A second `init` into a folder that it made says so and changes nothing.
- **The plan's files**: `docs/codelead/rebuild/rebuild-plan.json` and `rebuild-plan.md`, written by `rebuild init` and by `/modernize --rebuild-plan`.
- **The caps**: `run.unattended` in the manifest (`maxParts`, `maxCostUsd`, `maxMinutes`, `repairsPerSession`), or `--max-parts`, `--max-cost-usd` and `--max-minutes` for one run.
- **The ladder**: `run.models` in the manifest (`compose`, `repair`, `intervene`, `escalate`). With no `models` block, the ladder is off.
- **The run summary**: `.codelead/rebuild-run-<stamp>.md` and `.json`: each part, its outcome and its cost. See [Rebuild run summary](../reference/rebuild-run-summary.md).
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
