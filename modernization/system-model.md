---
title: The System Model
description: What the System Model of an old application holds, how the extraction gate checks it, how you explain a finding, and what the feature map is.
kind: concept
area: modernization
date: 2026-11-20
---

# The System Model

The System Model is CodeLead's map of an old application, read from its source code. It lists every entry point, screen, script and database table, and the links between them, each with its file and line. CodeLead reads it in the Model stage with no model call: the word "model" here means this map, not an AI model.

**In short**
- You know what the System Model holds, and what CodeLead reads to make it.
- You know what the extraction gate checks, and how to explain one of its findings.
- You know what the feature map is, and why the rebuild plans from it.
- You know how the journeys draft signs users in, and what to do when a stored model is older than CodeLead's reading.

## Why it exists

A rebuild must do everything that the old application did. Nobody can promise that from memory. The System Model is a complete list, read from the code. The extraction gate checks that the list has no hole that nobody explained. The rebuild's plan is made from it, part by part.

## What it reads

[`codelead model`](../cli/codelead-model.md) reads the application with **recipes**: one for each framework that it knows. A recipe switches on when the project's packages or references name its framework. For the Basic Store sample, the recipes are `aspnet-mvc` and `jquery-browser`. CodeLead also has recipes for Entity Framework 6, ADO.NET, Knockout and RequireJS.

Four passes read the code:

- **server**: the routes, the controllers and their actions, the redirects;
- **surfaces**: the views, their layouts and partials, the links and the forms in them;
- **client**: the script code of the pages and what calls it;
- **channel**: the messages that the server pushes to the pages, when there are any.

It also reads the database as the code uses it: the tables, the columns and the keys. You can give it the database's own definition: a T-SQL script with `--schema-dump`, or a live copy's catalog with `--schema-from` and `--consent`. CodeLead then confirms or contradicts each table and column. A column of a table that the definition does not hold is neither: it is "not in" the definition.

## How it works

The [Model stage](the-eight-stages.md#5-model) runs `codelead model --source . --project <web project>`, then writes the feature map. On the Basic Store sample, the command prints:

```output
System Model of webapp (basic-store/WebApplication1, inventory: project-file): 289 nodes, 189 edges; 63 entry points, 154 caller records; 10 not resolved, 0 leading nowhere.
Recipes: aspnet-mvc, jquery-browser. Passes: server, surfaces, client, channel ran.
Gate: not passed: 36 findings to resolve (36 unexplained-orphan).
Schema read from the code (schema.md): 4 tables, 28 columns (0 confirmed, 28 inferred), 30 keys, 0 procedures called by name; 4 data accesses not read whole (0 schema-unread warnings).
Written all the same: the model is the evidence the findings are read against (extraction-gate.md names each, with the key to record a reason under).
Written: docs/codelead/rebuild/system-model.json
…
```

- `289 nodes, 189 edges`: the items of the map, and the links between them.
- `63 entry points`: the controller actions that a request can reach.
- `10 not resolved`: places that CodeLead could not read to the end, for example a redirect to an address that the request supplies. Each one is listed in the gate's report.
- `Gate: not passed`: the model is written all the same. It is the evidence that you read the findings against.

## The extraction gate

The extraction gate is the check of the System Model. It reads the model and asks, for example:

- Does every entry point have a live trigger (a link, a form, a redirect), or a recorded reason?
- Does every rendered view have its layout, and every partial a page that includes it?
- Is no **orphan** left unexplained? An orphan is an item that nothing live reaches, and that CodeLead cannot call dead either.
- Did the extraction read the application, and does every pass account for what it read?
- Does the site's root reach an action?

Eight such rules block the gate. Some others only warn, for example a posted field that no parameter reads. The report, `docs/codelead/rebuild/extraction-gate.md`, lists each finding with its place and its key. On the Basic Store sample, the first finding reads:

```output
- entry-point account.confirm-email is neither live nor dead: what could not be resolved at WebApplication1/Controllers/AccountController.cs:450, … could still reach it (`WebApplication1/Controllers/AccountController.cs:178`). Key: `unexplained-orphan:GET /Account/ConfirmEmail`
```

The page `/Account/ConfirmEmail` is part of the ASP.NET Identity template. Nothing in the sample links to it. CodeLead cannot be sure that nothing reaches it, so it asks for a reason.

## Explaining a finding

You explain a finding by writing a reason under its key in `docs/codelead/rebuild/extraction-dispositions.json`, then you read the model again:

```json
{
  "schema": "codelead.extraction-dispositions/1",
  "dispositions": {
    "unexplained-orphan:GET /Account/ConfirmEmail": "ASP.NET Identity scaffold route; nothing live reaches it; answer identity=keep"
  }
}
```

A reason explains a finding. It never changes what the model says.

The program does this for one class of finding by itself. Sometimes every open finding is a page of the ASP.NET Identity template that nothing reaches. Then the Model stage records one reason for each, and reads the model again. On the Basic Store sample:

```output
The extraction gate: 36 findings, every one the ASP.NET Identity scaffold that nothing live reaches (Account/ConfirmEmail, Account/ForgotPassword, Account/ForgotPasswordConfirmation, Account/ResetPassword, Account/ResetPasswordConfirmation, Account/ExternalLoginFailure, Manage/AddPhoneNumber, Manage/EnableTwoFactorAuthentication, Manage/DisableTwoFactorAuthentication, Manage/VerifyPhoneNumber, Manage/RemovePhoneNumber): 36 reasons recorded in docs/codelead/rebuild/extraction-dispositions.json. The model is read again.
…
Gate: passed.
```

Any other finding is yours. The stage then waits, with the title `The extraction gate needs your reasons` and the keys to write.

> [!TIP]
> **For experienced users:** a finding that is held open only by an unresolved place can also be explained on that place: `unexplained-orphan:<file>:<line>`. One reason then covers every orphan that the place keeps open.

## The feature map

After the gate passes, CodeLead writes the **feature map**: `docs/codelead/rebuild/feature-map.md` and `.json`. It lists every controller action as a feature, in the order of the rebuild's increments. For each feature it gives the old route, the view, the live triggers, and the facts and pins that anchor it. It also says what replaces the feature: an endpoint of the new API, a screen of the new web application, or both.

On the Basic Store sample the map holds `63 features — 22 screens, 0 fragments, 16 commands, 5 content screens, 20 authentication actions`. One row reads:

```output
| 7 | clients.details | `GET /Clients/Details (int? id)` | screen | Clients/Details (0 / 0 / 0) | ClientsController.cs:74, ClientsController.cs:119, Delete.cshtml:12, +8 | 0 (0) | 0 | `GET /api/clients/details` + screen `/clients/details` |
```

The map also names the decisions that come before some features. On the sample: `Authentication: the old app's membership and external logins are not ported action by action; the new API needs an identity scheme …`. That is the `identity` answer of [Set up the rebuild](rebuild.md).

## The journeys draft

`codelead model` also drafts journeys from the code: `docs/codelead/rebuild/journeys.draft.json`. A journey is a path that a person takes through the application, step by step. Each step asserts the first outcome that the model knows of it. [`codelead rebuild init`](../cli/codelead-rebuild-init.md) merges the draft with the Analyst's recordings and the Explorer's stories. See [Journeys format](../reference/journeys-format.md).

How a drafted journey signs its user in depends on how the old application signs users in:

- **Windows sign-in, a token, or a directory's accounts**: the users are accounts that the walk is given. Nobody is registered.
- **A sign-in form of the application's own, with its own accounts**: the users are accounts that the walk is given too. The model reads the form's sign-in action, its account field and its hidden fields. Often the old application sent a visitor who opened `/` to its sign-in page. Then a journey of a signed-in user starts as that visitor: it opens `/`, and expects the sign-in page. The next step signs in with the walk's account, and expects the page where the old sign-in sent it. Then come the feature's own steps. When the root is open to a visitor, the journey signs in once its steps reach the sign-in form. When the model knows no way there, the step's title says that a person adds it.
- **Any other site**: the walk registers new users through the screens, in every run.

The draft never holds a password. A sign-in step types `{user.account}` and `{user.password}`. The walk reads the password when it runs, from `CODELEAD_WALK_PASSWORD_<USER>` or from its credentials file. Each account is a placeholder, such as `walk-user-1@example.com`. For a walk of the old application, replace it with a test account. See [What a given identity does](../reference/journeys-format.md#what-a-given-identity-does-and-what-it-does-not).

> [!NOTE]
> **New to this?** Many old applications send a visitor who is not signed in to their sign-in page. There, a journey that opened `/` as a visitor and expected to stay at `/` would fail, on the old application and on the rebuild. So the draft signs in first, as a person does.

## When CodeLead reads more than the stored model

The System Model records which reading of the code extracted it: `extractedBy` in `system-model.json`. It holds the reading's number and the CodeLead build that ran `codelead model`, for example `"extractedBy": { "reading": 1, "build": "codelead 0.1.0 (<COMMIT>)" }`. CodeLead raises the reading's number when a reader of the code changes what a model holds. The feature map records the same stamp under `source.systemModel`.

A model with an older reading, or with no `extractedBy`, lacks what the newer readers read. CodeLead never extracts the model again by itself, because an extraction writes files in the old application's folder. It says so, with the commands to run:

- **The rebuild's plan** warns with `model-reading-old`. The start of `codelead rebuild run`, its dry run and `codelead rebuild init` print the warning as a note.
- **`codelead modernize status`** prints an `Older reading:` line under the Model stage.

The plan's note reads, for a model stored before CodeLead numbered its readings:

<!-- described, not captured -->
```output
the System Model of webapp (extracted <TIME>) was extracted by an older reading (none recorded: it was extracted before CodeLead numbered its readings) than this build's (1): what the readers since then read is not in it: run `codelead model --source . --project webapp` in the old application's folder (../basic-store), then `codelead rebuild journeys --refresh` here to bring its journeys draft into the rebuild's copy; the plan reads the new model and feature map at its next turn: a part whose endpoint's proposal they move is planned again (proposal-moved), and a part accepted already stays accepted when only its request's words would change (reopen it to compose it anew)
```

What to do:

1. In the old application's folder, run `codelead model --source . --project <WEB_PROJECT>`, or the program's stage: `codelead modernize run model`.
2. In the rebuild's folder, run [`codelead rebuild journeys --refresh`](../cli/codelead-rebuild-journeys.md). The rebuild's copy of the journeys then takes the new draft.
3. Review the copy with `git diff`, and commit it.

An accepted part stays accepted. To compose it again from the new model, reopen it with `/modernize --transform T9 --reopen <PART_ID> --reason "<WHY>"`.

## Details

- **The files** are all in `docs/codelead/rebuild/` of the old repository. `system-model.json` is the model, and `model.md` shows its identity and rules for people. The others are `extraction-gate.md`, the drafts `journeys.draft.json` and `truth.draft.json`, `schema.json`, `schema.md`, `feature-map.json` and `feature-map.md`.
- **The journeys draft** cannot carry the reading's stamp, because the journeys format refuses other keys. The model written beside it carries the stamp for both.
- **Exit codes** of `codelead model`: `0` the gate passed, `1` it did not, `2` nothing could be read or written.
- **Several projects**: `--project <NAME>` names the web project. `--project-root <FOLDER>` says which of several projects that build one assembly is read.
- **Nothing half-written**: CodeLead makes every file in memory first. An extraction that read nothing writes nothing, and the model stored before stays.

## What it is not

- **The System Model is not an AI model**. Its reading is deterministic. The same code gives the same model.
- **The System Model does not read Web Forms pages**. A Web Forms project has no server recipe, so its model holds only the database (schema only). Its pages are not mapped, and they are not rebuilt.
- **The System Model is not a test**. It says what the code can do. The pins and [the walk](walk-and-parity.md) say what it does.

## Related content

- [`codelead model`](../cli/codelead-model.md)
- [`codelead rebuild journeys`](../cli/codelead-rebuild-journeys.md): bring a new draft into the rebuild's journeys
- [The eight stages](the-eight-stages.md#5-model)
- [The rebuild](rebuild.md): the plan is made from the model and its feature map
- [`codelead register`](../cli/codelead-register.md) and [`codelead seed`](../cli/codelead-seed.md): read the model's files
- [The behaviour register and seed data](../concepts/register-and-seed.md)
- [`/modernize --rebuild-map`](../chat-commands/modernize.md#--rebuild-map)
- [Glossary](../concepts/glossary.md)
