---
title: Rebuild manifest
description: The format of docs/codelead/rebuild/rebuild.json, codelead.rebuild/1 - the old application, the targets, the decisions, the confirmed effects and the run profile, key by key.
kind: reference
area: reference
date: 2026-11-20
---

# Rebuild manifest

**Applies to:** CodeLead 0.1 and later

The **rebuild manifest** is the file that makes a folder a [rebuild](../modernization/rebuild.md): a new application, built part by part beside an old one. It names the old application, its feature map and the targets of the new code. It keeps the decisions that a person made about the new application. Its `run` block is the rebuild's [run profile](run-profile.md#the-rebuilds-run-profile).

## The file

| Fact | Value |
|---|---|
| Path | `docs/codelead/rebuild/rebuild.json` in the rebuild. |
| Schema | `codelead.rebuild/1`. |
| Written by | [`codelead rebuild init`](../cli/codelead-rebuild-init.md) and `/modernize --rebuild-init`, once. Then you, for the decisions and the run profile. |
| Read by | [`codelead rebuild`](../cli/codelead-rebuild.md), [`codelead rebuild run`](../cli/codelead-rebuild-run.md), `/modernize --rebuild-plan`, `/modernize --transform T9`, [`codelead parity --rebuild`](../cli/codelead-parity.md), [`codelead rebuild-report`](../cli/codelead-rebuild-report.md), [`codelead walk`](../cli/codelead-walk.md) and the [modernization program](../modernization/the-program.md). |
| In git | Yes. It is the rebuild's record of what was decided. |

`rebuild init` refuses to make a second rebuild in a folder whose manifest says that it made one.

The answers of the [modernization program](modernization-state-file.md#answers) end up here. The Initialize the rebuild stage passes `framework`, `identity` and `database` to `rebuild init`. `framework` becomes `targets.api.framework`, and the other two become `decisions.identity` and `decisions.database`.

## A manifest

`codelead rebuild init --out ../basic-store-rebuild --identity keep --database sample-seed` writes this manifest for the Basic Store sample. The `run` block is shortened:

<!-- described, not captured -->
```json
{
  "schema": "codelead.rebuild/1",
  "kind": "aspnet-mvc-to-api-and-spa",
  "createdBy": {
    "command": "codelead rebuild init",
    "at": "2026-11-20T04:16:14.000Z",
    "template": "dotnet-next-shadcn"
  },
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
  },
  "run": {
    "schema": "codelead.run-profile/1",
    "api": { "command": "npm", "args": ["run", "api:dev"], "url": "http://localhost:5080", "ready": "/health", "startMs": 180000 },
    "web": { "command": "npm", "args": ["run", "dev"], "url": "http://localhost:3000", "ready": "/", "env": { "NEXT_PUBLIC_API_URL": "http://localhost:5080" } },
    …
  }
}
```

`rebuild init` says it in one line:

```output
Manifest: docs/codelead/rebuild/rebuild.json: aspnet-mvc-to-api-and-spa; API api/Api/Api.csproj (net10.0), web web; identity keep, database sample-seed, channel none.
```

## Top-level keys

| Key | Type | Required | What it holds |
|---|---|---|---|
| `schema` | string | Yes | `codelead.rebuild/1`. |
| `kind` | string | Yes | What is rebuilt into what, such as `aspnet-mvc-to-api-and-spa`. |
| `createdBy` | object | No | `command`, `at` (UTC time) and `template`: what made the rebuild. |
| `source` | object | Yes | The old application. |
| `targets` | object | Yes | Where the new code goes. |
| `decisions` | object | No | What a person decided about the new application. |
| `confirmedEffects` | object | No | For each command of the old application that returns nothing, by feature id: the effects that a person confirmed for its contract test. |
| `run` | object | No | The [run profile](run-profile.md#the-rebuilds-run-profile). Without it, acceptance by walking is off. |

## source

| Key | Type | What it holds |
|---|---|---|
| `repository` | string | The old application's folder, relative to the rebuild. |
| `featureMap` | string | The old application's feature map, `docs/codelead/rebuild/feature-map.json`, relative to the rebuild. |
| `webProject` | string | The old web project's name. |
| `libraries` | list of strings | The libraries of the old application that the System Model read. |

The plan reads the old application's files through these paths. A rebuild whose old application is not at `repository` cannot be planned: [`codelead rebuild --plan`](../cli/codelead-rebuild.md) ends with exit code 1.

## targets

| Key | Type | What it holds |
|---|---|---|
| `api.project` | string | The new API's project file, relative to the rebuild. |
| `api.tests` | string | The API's test project. |
| `api.stack` | string | `aspnetcore`. |
| `api.framework` | string | `net10.0` or `net8.0`, from `--framework`. |
| `api.routePrefix` | string | The prefix of the API's routes, `/api`. |
| `web.path` | string | The web application's folder. |
| `web.stack` | string | The web stack's names, from the [template manifest](template-manifest.md). |
| `web.status` | string | Optional. |

## decisions

What the new application does, as a person decided it. The reasons why a finding of the old application is intended are not here: they stay in the old repository. See [Extraction dispositions and security waivers](dispositions-and-waivers.md).

| Key | Type | What it holds |
|---|---|---|
| `identity` | string | The identity answer and what it means: `keep` (the accounts stay, with ASP.NET Core Identity over the old user tables), `drop` (no sign-in), or `none` (the old application signs nobody in). |
| `database` | string | The database answer: `sample-seed`, `connection <NAME>` (the connection string that the new API reads, kept out of the repository), or `none`. |
| `channel` | string or object | `"none"` when the old application has no real-time channel. Otherwise the channel translation table, `codelead.channel-translation/1`. |
| `dropped` | object | Features that the rebuild does not carry, by feature id, each with its reason. Parity counts them as `dropped by decision`. |
| `replacedBy` | object | Features that something else replaces, by feature id: `{ endpoint, screen, client, note }`. |
| `model` | object | What the rebuild does about a node of the System Model, keyed as the plan prints it. |
| `dispositions` | object | Reasons for findings of the rebuild's wiring check, by the check's key. |

`codelead rebuild init --identity drop` drops every account feature, with the reason `the rebuild has no sign-in (codelead rebuild init --identity drop)`.

With `keep`, no part of the plan builds the directory that checks a password outside Development and the tests. The plan says so in a note on the part `sign-in:endpoint`, and a person adds the real directory.

### decisions.model

Each key is one form that the plan prints: `reveal:<entry point>`, a key of the extraction gate's report, `unknown:<file>:<line>` or `route:<path>`. Each value:

| Key | Type | What it holds |
|---|---|---|
| `decision` | string, required | `rebuild`, `drop`, `replace`, `enforce`, `keep-as-old` or `follow-intent`. |
| `says` | string, required | What the decision means, in words. |
| `by` | string | Who decided. |
| `at` | string | When, as `YYYY-MM-DD`. |
| `ref` | string | The decision's name in a person's list. |

Example:

```json
"decisions": {
  "identity": "keep: …",
  "database": "sample-seed: …",
  "channel": "none",
  "dropped": {
    "manage.add-phone-number": "The team does not send text messages; the rebuild has no phone numbers."
  },
  "model": {
    "route:/Home/About": { "decision": "drop", "says": "The About page is static text; the new home page holds it.", "by": "sam", "at": "2026-11-20" }
  }
}
```

Where nothing is decided, each kind of node takes a default, and the plan shows it as a warning. A decision that is malformed is left out, and the plan says why in plain words, for example `decisions.dropped["orders.export"] gives no reason; a dropped feature says why.`

### decisions.channel

A table of the old real-time channel, one row per piece:

| Key of a row | What it holds |
|---|---|
| `id` | The row's id. |
| `old` | The piece of the old channel. |
| `evidence` | The old lines it rests on, as `file:line`. |
| `new` | What it becomes in the new application. |
| `owner` | `shell`, or the feature id of the screen that registers it. |
| `status` | `draft` or `reviewed`. |
| `by`, `at` | Who reviewed it, and when. |
| `moments`, `note` | When a start-up runs, and a note that the merge adds. |

`/modernize --review channel` shows the table for review.

## See also

- [The rebuild](../modernization/rebuild.md)
- [`codelead rebuild init`](../cli/codelead-rebuild-init.md), [`codelead rebuild`](../cli/codelead-rebuild.md) and [`codelead rebuild run`](../cli/codelead-rebuild-run.md)
- [Run profile](run-profile.md)
- [Journeys format](journeys-format.md)
- [Rebuild run summary](rebuild-run-summary.md)
- [Template manifest](template-manifest.md)
- [`codelead model`](../cli/codelead-model.md) and [`codelead modernize answer`](../cli/codelead-modernize-answer.md)
