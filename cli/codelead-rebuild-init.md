---
title: codelead rebuild init
description: Make the rebuild repository beside the old application, with its template, manifest, run profile, journeys, plan and a dry run.
kind: reference
area: cli
date: 2026-11-20
---

# codelead rebuild init

**Applies to:** CodeLead 0.1 and later

`codelead rebuild init` makes the rebuild repository beside the old application. It copies a template, writes the manifest and the run profile, merges the journeys, initializes CodeLead, writes the plan and runs a dry run. Then it commits what it made. Run it in the old application, after [`codelead model`](codelead-model.md).

## Name

`codelead rebuild init` - Makes the rebuild repository beside the old application.

## Synopsis

```console
codelead rebuild init --out <FOLDER> [--source <FOLDER>] [--template <NAME|PATH>]
                      [--framework <TFM>] [--identity keep|drop]
                      [--database <CONNECTION_NAME>|sample-seed] [--data-access port|same-library]
                      [--no-install] [--no-commit] [--json]
codelead rebuild init -h|--help
```

## Description

`init` refuses before it writes anything when something is missing:

- the old application has no System Model or feature map (run [`codelead model`](codelead-model.md));
- `--out` is not new or empty;
- an answer that the manifest needs is not given (each one is named, with the option to pass);
- the template cannot be read.

Then it makes the rebuild in ten steps:

1. **The template**: copied into `--out`. With `--framework`, the API's target framework is set.
2. **The ports**: the API and the web application get ports that the old application does not answer on. The `Ports:` line says which, and why.
3. **Git**: the folder becomes a git repository.
4. **The install**: `npm install`, unless `--no-install`.
5. **The manifest**, `docs/codelead/rebuild/rebuild.json`. It names the old application, its feature map and its web project. It names the targets: the new API and the web application. It keeps the decisions: identity, database and, when the API has a database, the data access.
6. **The run profile** in the manifest: how to start the API and the web application, the clock, the caps of an unattended run. For a sign-in with a cookie across two origins, it also holds a check of the API's CORS policy. That policy is the API's rule for which origins may call it. See [The two answers](#the-two-answers).
7. **The journeys**, `docs/codelead/rebuild/journeys.json`: merged from the model's draft, the Analyst's recordings and the Explorer's stories. `init` makes this copy once. See [The journeys copy](#the-journeys-copy).
8. **CodeLead initialized** in the folder: the configuration, the repository map, a board, a conventions starter and the declared validation gate.
9. **The plan**, with no model call, and a dry run of [`codelead rebuild run`](codelead-rebuild-run.md). The dry run asks the model server for its list of models, and nothing more. The `Plan:` line counts the parts that the plan drops because nothing live reaches them. The plan's notes follow it, each on a `Note:` line. For example, a note says when the stored System Model is older than this CodeLead's reading: the plan's warning `model-reading-old`. See [The System Model](../modernization/system-model.md#when-codelead-reads-more-than-the-stored-model).
10. **One evidence record**, `rebuild-init`, for success or failure. Then **the commit** of what was made, with this machine's git user, unless `--no-commit`.

### The ports

The new application and the old one run side by side during the walk and the parity check. So `init` reads the ports that the old application answers on, from five places:

- the program's address, `application.baseUrl` in `.codelead/modernization.json`;
- the old application's run profile, `modernization.source.url` in `.codelead/config.json`;
- the base address of the Analyst's recordings: `base` in each `analyst-recordings/<time>/recording.json`, and in the Explorer's `explore-<role>/recording.json`. So a program started without `--base` and without a run profile still moves the new API off the port that the journeys were recorded on. The ports line then says `(the Analyst's recordings)`;
- the web project's file: `<IISUrl>` and `<DevelopmentServerPort>`;
- the web project's `Properties/launchSettings.json`.

When the template's port of the API or the web is one of them, `init` takes the next free port. It writes the new port into the API's launch settings and the web's dev script. When those files do not hold the port, it sets `ASPNETCORE_URLS` or `PORT` on the server in the run profile instead.

### The commit

`init` commits the folder with this machine's git user, as `CodeLead: the rebuild's scaffold from <template>`. The Rebuild stage then starts on a clean tree. With `--no-commit`, nothing is committed, and you commit it yourself before the first run. When git has no user, `init` makes no commit, and says so: `Commit: not made: no git user is configured (git config user.name and user.email): commit it yourself before the first run.`

> [!NOTE]
> **New to this?** A *template* is a starter application: an API and a web application with their build and test scripts. The default template is `dotnet-next-shadcn`: an ASP.NET Core API and a React web application.

### The two answers

`--identity` is needed when the old application signs users in. `--database` is needed when the old application reaches a database. The refusal names each, with both options and the line to run.

- `--identity keep`: the new API keeps the accounts, with ASP.NET Core Identity over the old user tables. `--identity drop`: no sign-in, and the account features are dropped.
- `--database <name>`: the name of the connection string that the new API reads, kept out of the repository. `--database sample-seed`: a sample database, reset before every journey.

The `database` answer gives the plan its part `data-access`. How the new API reaches that database is a decision of its own, `--data-access`:

- `--data-access port`, the default when the old application is on Entity Framework 6. The rebuild ports the data access to EF Core inside the new API. The old application is not changed for it.
- `--data-access same-library`. The new API references the old application's data library and reaches the data through it. The old application's Upgrade in place stage makes the library usable first.

The manifest records it as `decisions.dataAccess`. To change it later, answer `dataAccess=port` or `dataAccess=same-library` with `codelead modernize answer`. The program then writes it into the manifest. With `sample-seed`, the API makes the sample database at start-up when it is missing. It runs the old application's own schema and seed scripts, such as `database/schema.sql` and `database/seed.sql`. `init` copies them into the rebuild first, so the new application never reads the old repository beside it. The copies go into the API project's folder `SampleSeed`, with a note, `copied.json`, that keeps where each came from and its hash. The line says so:

<!-- described, not captured -->
```output
Sample database: the sample database's scripts copied into api/Api/SampleSeed: schema.sql (from the old application's database/schema.sql) and seed.sql (from the old application's database/seed.sql), the statements only (2 comment lines left out), with where each came from and its hash in copied.json.
```

The copies keep the statements and leave the comments out, because a seed may name a test password in a comment. A seed that writes a password in clear text in a statement is not copied, and the line says what to do. When the old application keeps no script that creates its tables, the part waits, and its note says what to give. See [The data access](../modernization/rebuild.md#the-data-access).

### The day reset

With `sample-seed`, `init` writes the day reset, `scripts/reset-sample-seed.mjs`. Before every journey of a walk, it puts the sample database back to the seed. It works in place, through SQLite, while the API runs. It drops each table and makes it again from the copies, so the ids start again too. It never deletes, moves or replaces the file, because the API keeps its connections to it open. It needs Node.js 22.13 or later. When it has run, it says one line:

<!-- described, not captured -->
```output
Sample database put back to the seed in place: api/Api/sample-seed.db (schema.sql and seed.sql).
```

An earlier `init` wrote a reset that deleted the file. CodeLead writes that one again by itself, at the next run. Before every walk, CodeLead also checks that the API still reads its data after the reset. See [The checks before a walk](../modernization/walk-and-parity.md#the-checks-before-a-walk).

With `--identity keep`, the plan builds the real user directory as a part of its own, `user-directory`. It checks a password against the old user table, through the rebuild's data access. Its password check is ported from the old code, such as `PasswordHasher.Verify`, never invented. The sign-in's stand-in directory stays for Development and the tests. Until the part is accepted, a build for production signs nobody in. When CodeLead cannot read the old check to its source, there is no such part. The note on `sign-in:endpoint` then says what was read and what was not, and a person adds the real directory. See [The user directory](../modernization/rebuild.md#the-user-directory).

When the System Model reads a sign-in action in the old code, the rebuild signs users in with a cookie. With `--identity keep`, and the API and the web application at two origins, `init` writes a check into the run profile. An origin is the scheme, the host and the port of an address. The check is an `OPTIONS` request to the API's caller-identity route, such as `/api/me`, from the web application's origin. It expects `Access-Control-Allow-Origin` with that origin and `Access-Control-Allow-Credentials: true`. The `Run profile:` line names it. See [Run profile](../reference/run-profile.md#checks).

### Where the template comes from

A template name is looked up in this order: `greenfield.templates` in the configuration, then `CODELEAD_TEMPLATES_DIR/<name>`, then `../codelead-templates/<name>` beside the repository. For `dotnet-next-shadcn`, CodeLead then clones the template's git repository. `--template` also takes a folder or a git address.

### The journeys copy

The rebuild walks its own copy of the journeys. `init` makes it once, from the sources as they are then. After that, the copy belongs to the rebuild repository: you commit it, and you can edit it. No stage of the program copies the journeys again.

The System Model's draft can change later. For example, `codelead model` runs again with a newer CodeLead, whose draft signs a site's users in. The plan then warns with `journeys-draft-moved`: the copy stands otherwise than the draft. The warning names the journeys that moved, and the start of each run prints it. To bring the new draft in, run [`codelead rebuild journeys --refresh`](codelead-rebuild-journeys.md) in the rebuild. It keeps the Analyst's and the Explorer's journeys, the journeys written by hand, and a drafted journey that a person marked.

### A second run

When `--out` already holds a rebuild that this command made, `init` says so and stops. It changes nothing, and it does not copy the journeys again.

> [!TIP]
> **For experienced users:** the unattended caps start at 12 parts, $20 and 480 minutes, with 8 repairs for each session. Raise them under `run.unattended` in the manifest. With `sample-seed`, the run profile's day reset is `node scripts/reset-sample-seed.mjs`.

The command is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, it is refused before anything runs, with exit code 2. See [Licence and plans](../concepts/licence-and-plans.md).

## Options

- **`--out <FOLDER>`**

  The folder that the rebuild is made in: new or empty, beside the old application.

- **`--source <FOLDER>`**

  The old application's root. The default is the current folder.

- **`--template <NAME|PATH>`**

  The template: a name (from `greenfield.templates`, `CODELEAD_TEMPLATES_DIR`, or `dotnet-next-shadcn`), a folder or a git address. The default is `dotnet-next-shadcn`.

- **`--framework <TFM>`**

  The API's target framework: `net10.0` or `net8.0`. The default is the template's.

- **`--identity <ANSWER>`**

  `keep` or `drop`. Needed when the old application signs users in.

- **`--database <ANSWER>`**

  The name of the connection string that the new API reads, or `sample-seed`. Needed when the old application reaches a database.

- **`--no-install`**

  Does not run `npm install` in the rebuild. Run it yourself before the first run.

- **`--no-commit`**

  Does not commit what was made. By default, `init` commits it with this machine's git user, so that the first run starts on a clean tree. With `--no-commit`, commit it yourself before the first run.

- **`--json`**

  Prints the result as one JSON object.

## Examples

- Run it without the answers. CodeLead refuses, names each answer, and gives the line to run. The exit code is 2:

  ```console
  codelead rebuild init --out ../basic-store-rebuild --no-install
  ```

  ```output
  Refused: Answers the manifest needs are missing: identity: the old application signs users in (20 account features). Pass --identity keep (the new API keeps the accounts, with ASP.NET Core Identity over the old user tables) or --identity drop (the rebuild has no sign-in; the account features are dropped); database: the old application's code reaches a database (4 tables). Pass --database <connection name> (the name of the connection string the new API reads, kept out of the repository) or --database sample-seed (the new API starts on a sample database, put back before every journey).
    The way out: Run the command again with the answers, for example: codelead rebuild init --out <folder> --identity keep --database sample-seed
  Nothing was made.
  ```

- Make the rebuild of the Basic Store sample, with the answers, after the program's Model stage. Here `CODELEAD_TEMPLATES_DIR` names the folder that holds the templates. The old application answers on 5080, the program's address, so the API moves to 5081:

  ```console
  CODELEAD_TEMPLATES_DIR=<your templates folder> codelead rebuild init --out ../basic-store-rebuild --identity keep --database sample-seed --no-install
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
    - The model of the default role, the preflight role, the architect role, the repair role does not answer: lmstudio (http://127.0.0.1:9/v1) answered no list of its models (Could not connect to lmstudio at http://127.0.0.1:9/v1. If this is LM Studio, make sure it is open and the Local Server is running.).
  Evidence: .codelead/evidence/ev-20261120082318-c5dj0f.json (rebuild-init · failure).
  Commit: efee766 "CodeLead: the rebuild's scaffold from dotnet-next-shadcn", with this machine's git user: the rebuild starts on a clean tree.
  Not ready to run: answer the dry run's problems above; codelead rebuild run --dry-run in the folder checks again.
  ```

  The rebuild is made and committed. It is not ready to run only because the model server did not answer. Start the model server, then check again with `codelead rebuild run --dry-run` in the folder. When the dry run passes in `init`, the last line reads `Ready: run codelead rebuild run in ../basic-store-rebuild.`

- Make the rebuild after `codelead model` alone. Nobody has recorded the reasons for the extraction gate's findings yet, so the plan has 36 blockers. No program runs here, so only the project file gives the old ports, and the API keeps 5080:

  ```console
  codelead rebuild init --out ../store-rebuild --identity keep --database sample-seed --no-install
  ```

  ```output
  Rebuild of webapp (.) from the template dotnet-next-shadcn (…/dotnet-next-shadcn), in ../store-rebuild.
  Template: 43 files copied.
  Ports: the API on 5080, the web on 3000; the old application answers on 44366 (webapp.csproj's IISUrl), 52895 (webapp.csproj's DevelopmentServerPort).
  Git: the folder is a git repository; what is made is committed at the end.
  …
  Plan: 80 parts: 78 planned, 2 dropped; blockers: 36 (the extraction gate 36). Written: docs/codelead/rebuild/rebuild-plan.json and docs/codelead/rebuild/rebuild-plan.md.
    - the extraction gate: unexplained-orphan: entry-point account.confirm-email is neither live nor dead: … Record why under "unexplained-orphan:GET /Account/ConfirmEmail" in the old repository's docs/codelead/rebuild/extraction-dispositions.json, then extract the model again (WebApplication1/Controllers/AccountController.cs:178)
  …
    … and 31 more, each with the key to record a reason or a decision under: docs/codelead/rebuild/rebuild-plan.md.
  Dry run: refused (the plan's blockers above, and what follows):
  …
  Evidence: .codelead/evidence/ev-20261120083559-g7h31i.json (rebuild-init · failure).
  Commit: 71da0a4 "CodeLead: the rebuild's scaffold from dotnet-next-shadcn", with this machine's git user: the rebuild starts on a clean tree.
  Not ready to run: resolve the plan's blockers (docs/codelead/rebuild/rebuild-plan.md says each, with the key to record a reason or a decision under), then answer the dry run's problems above; codelead rebuild run --dry-run in the folder checks again.
  ```

  The 36 blockers are ASP.NET Identity pages that nothing in the sample links to. Without their reasons, the plan cannot know that nothing live reaches them, so it drops none of them. The program's `model` stage records the reasons by itself. See [`codelead modernize run`](codelead-modernize-run.md).

- Leave the commit to yourself. The `Git:` line says so, no `Commit:` line follows, and the last line asks you to commit:

  ```console
  codelead rebuild init --out ../store-no-commit --identity keep --database sample-seed --no-install --no-commit
  ```

  ```output
  …
  Git: the folder is a git repository (nothing committed, --no-commit: commit what was made before the first run).
  …
  Evidence: .codelead/evidence/ev-20261120082321-ugao06.json (rebuild-init · failure).
  Not ready to run: answer the dry run's problems above; codelead rebuild run --dry-run in the folder checks again.
  ```

  When this run is ready, its last line reads `Ready: commit the folder, then run codelead rebuild run in ../store-no-commit.`

- Make a rebuild for .NET 8 from a template folder:

  ```console
  codelead rebuild init --out ../store-net8 --template ../codelead-templates/dotnet-next-shadcn --framework net8.0 --identity drop --database sample-seed
  ```

  <!-- described, not captured -->
  ```output
  Rebuild of webapp (.) from the template dotnet-next-shadcn (../codelead-templates/dotnet-next-shadcn), in ../store-net8.
  Template: 43 files copied.
  …
  Install: npm install …
  Manifest: docs/codelead/rebuild/rebuild.json: aspnet-mvc-to-api-and-spa; API api/Api/Api.csproj (net8.0), web web; identity drop, database sample-seed, channel none.
  …
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The rebuild is made, the plan has no blocker, and the dry run's checks passed. |
| `1` | The rebuild is made but not ready to run. The words say what to do. A commit that could not be made does not change the code. |
| `2` | A refusal: nothing was made. The words say why, and the way out. A folder without the Pro plan is refused too. |

## Files

In the old application, `init` reads `docs/codelead/rebuild/system-model.json`, `feature-map.json`, `journeys.draft.json` and the Analyst's recordings. In the rebuild (`--out`), it writes:

| File | What it holds |
|---|---|
| The template's files | The new API and web application. |
| `docs/codelead/rebuild/rebuild.json` | The manifest and its run profile. See [Rebuild manifest](../reference/rebuild-manifest.md). |
| `docs/codelead/rebuild/journeys.json` | The merged journeys, the rebuild's own copy. [`codelead rebuild journeys`](codelead-rebuild-journeys.md) refreshes it. See [Journeys format](../reference/journeys-format.md). |
| `docs/codelead/rebuild/rebuild-plan.json`, `.md` | The plan. |
| `.codelead/config.json`, `docs/codelead/repository-map.*`, `docs/codelead/board/` | CodeLead initialized. |
| `.codelead/CONVENTIONS.md` | A conventions starter, to fill in. |
| `.codelead/validation.json` | The declared gate, from the template's build and test scripts. |
| `scripts/reset-sample-seed.mjs` | With `sample-seed`, the day reset. |
| `docs/codelead/licence.json` | Copied from the old application, when it has one. |
| `.codelead/evidence/ev-*.json` | The `rebuild-init` record. |
| `.git/` | The rebuild's own git repository, with its first commit unless `--no-commit`. |

A moved API port is written into the API's `Properties/launchSettings.json`. A moved web port is written into the web's `package.json` dev script, and into the API's `appsettings*.json`, where the API allows the web's address.

## See also

- [The rebuild](../modernization/rebuild.md)
- [`codelead model`](codelead-model.md): the step before.
- [`codelead rebuild run`](codelead-rebuild-run.md): the step after.
- [`codelead rebuild`](codelead-rebuild.md): print the plan.
- [`codelead rebuild journeys`](codelead-rebuild-journeys.md): refresh the journeys copy from a new draft.
- [Rebuild manifest](../reference/rebuild-manifest.md)
- [Template manifest](../reference/template-manifest.md)
- [`codelead modernize answer`](codelead-modernize-answer.md): the same answers, for the program.
