---
title: codelead rebuild journeys
description: Compare the rebuild's copy of the journeys with the System Model's draft, and refresh the copy's drafted journeys from the draft.
kind: reference
area: cli
date: 2026-11-24
---

# codelead rebuild journeys

**Applies to:** CodeLead 0.1 and later

`codelead rebuild journeys` compares the rebuild's copy of the journeys with the System Model's draft in the old application. With `--refresh`, it replaces the copy's drafted journeys with the draft's, and keeps what a person made. Run it in the rebuild repository that [`codelead rebuild init`](codelead-rebuild-init.md) made.

## Name

`codelead rebuild journeys` - Compares the rebuild's copy of the journeys with the System Model's draft, and refreshes it.

## Synopsis

```console
codelead rebuild journeys [--target <REPO>] [--refresh]
codelead rebuild journeys -h|--help
```

## Description

A **journey** is a path that a person takes through the application, step by step, with what each step must show. [The walk](../modernization/walk-and-parity.md) follows the journeys to judge each part of the rebuild. See [Journeys format](../reference/journeys-format.md).

The rebuild walks its own copy of the journeys, `docs/codelead/rebuild/journeys.json`. `codelead rebuild init` makes this copy once. It merges three sources: the System Model's draft, the Analyst's recordings and the Explorer's stories. See [The journeys, from three sources](../modernization/rebuild.md#the-journeys-from-three-sources).

The draft is `docs/codelead/rebuild/journeys.draft.json` in the old application. [`codelead model`](codelead-model.md) writes it again each time it runs. A newer CodeLead can draft the journeys otherwise, for example with a sign-in that the earlier draft did not have. The copy does not change by itself. No stage of the program copies the journeys again, and a second `rebuild init` changes nothing.

> [!NOTE]
> **New to this?** The copy belongs to the rebuild repository, and you commit it there. CodeLead never writes over it without this command. So a journey that you recorded or wrote by hand is safe.

### Without --refresh: how the copy stands

The command reads the copy and the draft, and prints one line. It writes nothing. The line says one of two things:

- the copy stands as the draft writes it;
- the copy stands otherwise than the draft, and how.

The comparison reads the copy's drafted journeys, the journeys whose `source.kind` is `model`, against the draft. A drafted journey that a person marked, with a `note` or a `covers` list on a step, is not compared. It says:

- the drafted journeys that the draft now writes otherwise. CodeLead compares what each step does and expects, its user and its title. A step's `note` and `covers` are left out;
- the drafted journeys that the draft no longer writes;
- the draft's journeys that the copy does not hold;
- whether the draft signs its users in another way than the copy, such as `given/explicit` against `registered`.

Each user's account is left out of the comparison too: in `users`, and wherever a step quotes it, such as in a title. The account is yours. A test account in place of the draft's placeholder is no difference.

The exit code is 0 in both cases. Read the words, not the exit code.

When the copy signs in as accounts that the walk is given, the lines of [the walk's passwords](#the-walks-passwords) follow, as after a refresh.

### With --refresh: what is kept and what is replaced

`--refresh` writes the copy anew from the draft as it is now. It **keeps**, as they stand:

- every journey that did not come from the draft: the Analyst's recordings, the Explorer's stories, and the journeys written by hand (with no `source`);
- a drafted journey that a person marked: one with a step that has a `note` or a `covers` list.

It **replaces** every other drafted journey with the draft's journey of the same id. It **adds** the draft's journeys that the copy does not hold. It **removes** the drafted journeys that the draft no longer writes. The users and the identity of the kept journeys stay. The draft's users and identity come with the draft's journeys.

What you wrote in a user that the copy holds too is kept, such as a test account in place of the placeholder. The command says so on a line such as `Kept: the account of user user.` The draft's steps that quote the placeholder then quote your account. A placeholder that the draft wrote is not kept as yours.

The command then prints what it did: the journeys replaced, added, removed and kept, each by id. When the draft's journey is the same as a kept journey, CodeLead keeps the kept one. A draft's journey that the walk's check refuses in the new file is left out, on a `Left out:` line with the reason.

> [!IMPORTANT]
> CodeLead cannot tell your edits of a drafted journey from the draft's own changes. A drafted journey that you changed without a `note` or a `covers` list is replaced. Add a `note` to a step of the journey before you refresh, to keep it. After the refresh, `git diff` shows every change.

### When the refresh writes nothing

`--refresh` refuses, and leaves the copy as it was, when:

- the draft signs its users in another way than a kept journey does. One journeys file signs its users in one way only;
- the merged journeys are not a file that the walk takes;
- there is no draft in the old application, or the draft cannot be read.

The words say why. A refused refresh has the exit code 1.

### The walk's passwords

When the journeys sign in as accounts that the walk is given, the command ends with what you must do before a walk. It prints one line for each user:

- the variable that holds the user's password for the walk, such as `CODELEAD_WALK_PASSWORD_USER`, and whether it is `set` or `not set`;
- when a file of passwords is named, whether it holds a password for the account. The file is the one that `CODELEAD_WALK_CREDENTIALS` names, else `run.unattended.credentials` of the run profile.

The lines never show a password or the file's path. Journeys that register their users print none of these lines.

When an account is still the draft's placeholder, such as `walk-user-1@example.com`, one more line says so. A walk of the old application needs a real test account there. A walk of the rebuild can keep the placeholder when the rebuilt API holds a fake directory for Development. The walk gives its accounts to that directory. The line says whether the API's code holds one.

### After the refresh

1. Read the changes: `git diff docs/codelead/rebuild/journeys.json`.
2. Read the command's last lines, and do what they say. Set each password variable that is `not set`, or name a file of passwords outside the repository in `run.unattended.credentials`. See [Passwords](../analyst/passwords.md).
3. For a walk of the old application, write a real test account in place of each placeholder. The next refresh keeps it.
4. Commit the file before the next run.

The plan reads the new copy at its next turn. A part that a walk accepted stays accepted while that walk judged its own steps as the new copy writes them. When its own steps changed, the run judges it again by itself, with no model call. When they fail, the plan plans it again, to be revised in place. See [When a part's steps change](../modernization/rebuild.md#when-a-parts-steps-change).

A newer draft can give a step an action that an earlier draft could not name. For example, the earlier draft wrote `orders.cancel-4` with no action, `Act on the link written as "#" + order.Id`. The newer draft clicks the first link to `/Orders/Details`. Run [`codelead model`](codelead-model.md) in the old application first, so that the draft is the newer one, then refresh. Until then, the plan judges no part on such a step. See [Steps a person completes](../reference/journeys-format.md#steps-a-person-completes).

Without a password for each account, [`codelead rebuild run`](codelead-rebuild-run.md) refuses at its checks, with the line `The walk's passwords: …`.

### Where the difference is said

The plan says the same as this command, without `--refresh`. It warns with `journeys-draft-moved` when the copy stands otherwise than the draft. The same words are one of the plan's notes. The start of `codelead rebuild run`, its dry run and `codelead rebuild init` print the plan's notes.

The command is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, it is refused before anything runs, with exit code 1. A rebuild counts as on the Pro plan when its own folder has the plan, or the old application that its manifest names has it. See [Licence and plans](../concepts/licence-and-plans.md).

## Options

- **`--target <REPO>`**

  The rebuild repository. The default is the current folder.

- **`--refresh`**

  Writes the copy anew from the draft, and prints what it kept and what it replaced. Without it, nothing is written.

## Examples

The examples come from CodeLead's tests. In them, the rebuild was made before the draft signed its users in, and the old application is in `../old`.

- See how the copy stands against the draft. Here one drafted journey is drafted otherwise now, and the draft signs its users in another way:

  ```console
  codelead rebuild journeys
  ```

  <!-- described, not captured -->
  ```output
  The journeys copy docs/codelead/rebuild/journeys.json stands otherwise than the System Model's draft ../old/docs/codelead/rebuild/journeys.draft.json: 1 drafted journey is drafted otherwise now (orders.edit-post); the draft signs its users in as given/explicit, the copy as registered. The copy is the rebuild's own and is not rewritten by itself: `codelead rebuild journeys --refresh` replaces its drafted journeys with the draft's, and keeps the Analyst's, the Explorer's, those written by hand and a drafted journey a person marked with a note or a covers; after it, a part a walk accepted whose own steps changed is judged again by the run itself, and planned again when they fail.
  ```

  Nothing is written, and the exit code is 0.

- Refresh the copy from the draft:

  ```console
  codelead rebuild journeys --refresh
  ```

  <!-- described, not captured -->
  ```output
  Written: docs/codelead/rebuild/journeys.json, from the System Model's draft ../old/docs/codelead/rebuild/journeys.draft.json.
  Replaced with the draft's: orders.edit-post.
  Added from the draft: none.
  Removed (drafted no more): none.
  Kept as they were: none.
  The file is the repository's: git diff shows what changed; commit it before the next run. The plan reads it at its next turn. A part a walk accepted stays accepted while that walk judged its own steps as the journeys write them now. Where they changed, the run judges it again by itself, with no model call, and plans it again, to be revised in place, when they fail.
  The walk signs in as the accounts the journeys give, each with its password, which is never written in the journeys:
  - user (walk-user-1@example.com): CODELEAD_WALK_PASSWORD_USER is not set, and no file of passwords is named (CODELEAD_WALK_CREDENTIALS, or run.unattended.credentials of the run profile).
  The account of user, walk-user-1@example.com, is the draft's placeholder. A walk of the old application needs a real test account there: write it in users.user.account of docs/codelead/rebuild/journeys.json. A walk of this rebuild can keep it when the rebuilt API holds a fake directory for Development, which the walk gives its accounts to: none is in the API's code yet (the sign-in's endpoint builds it).
  ```

  The copy now signs its users in as the draft does: `identity` is `{ "mode": "given", "credentials": "explicit" }`, and the one user is `user`, with the placeholder account `walk-user-1@example.com`. The journey's second step types `{user.account}` and `{user.password}`, then clicks **Sign in**.

- Run it again after the refresh. The copy stands as the draft writes it, and the plan's warning is gone:

  ```console
  codelead rebuild journeys
  ```

  <!-- described, not captured -->
  ```output
  The journeys copy docs/codelead/rebuild/journeys.json stands as the System Model's draft ../old/docs/codelead/rebuild/journeys.draft.json writes it.
  The walk signs in as the accounts the journeys give, each with its password, which is never written in the journeys:
  - user (walk-user-1@example.com): CODELEAD_WALK_PASSWORD_USER is not set, and no file of passwords is named (CODELEAD_WALK_CREDENTIALS, or run.unattended.credentials of the run profile).
  The account of user, walk-user-1@example.com, is the draft's placeholder. A walk of the old application needs a real test account there: write it in users.user.account of docs/codelead/rebuild/journeys.json. A walk of this rebuild can keep it when the rebuilt API holds a fake directory for Development, which the walk gives its accounts to: none is in the API's code yet (the sign-in's endpoint builds it).
  ```

- Refresh a copy where you wrote a test account in place of the placeholder. The refresh keeps the account, and says so:

  ```console
  codelead rebuild journeys --refresh
  ```

  <!-- described, not captured -->
  ```output
  Written: docs/codelead/rebuild/journeys.json, from the System Model's draft ../old/docs/codelead/rebuild/journeys.draft.json.
  Replaced with the draft's: orders.edit-post.
  Added from the draft: none.
  Removed (drafted no more): none.
  Kept as they were: none.
  Kept: the account of user user.
  The file is the repository's: git diff shows what changed; commit it before the next run. The plan reads it at its next turn. A part a walk accepted stays accepted while that walk judged its own steps as the journeys write them now. Where they changed, the run judges it again by itself, with no model call, and plans it again, to be revised in place, when they fail.
  The walk signs in as the accounts the journeys give, each with its password, which is never written in the journeys:
  - user (desk-tester@example.org): CODELEAD_WALK_PASSWORD_USER is set.
  ```

  The account is a test account now, so no line names a placeholder. The password is never shown.

- Refresh a copy where a person wrote a `note` on a step of the drafted journey. The journey is kept, so its users sign in as before, and the draft signs them in another way. Nothing is written, and the exit code is 1:

  ```console
  codelead rebuild journeys --refresh
  ```

  <!-- described, not captured -->
  ```output
  Refused: the draft signs its users in as given/explicit, and the journeys kept (orders.edit-post) as registered: one spec signs its users in one way, so nothing was replaced. docs/codelead/rebuild/journeys.json is as it was.
  ```

  To go on, remove the `note`, so that the draft replaces the journey, and refresh again. A journey that you keep must sign in as the draft does: change the copy's `identity` and the journey's user first.

- Run it in a folder that is not on the Pro plan. The exit code is 1:

  ```console
  codelead rebuild journeys
  ```

  ```output
  The modernization program is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.
  ```

A journey that the refresh keeps is listed with its reason, in the form `Kept as they were: <JOURNEY_ID> (from the Analyst's recordings)`. The other reasons are `from the Explorer's stories`, `written by hand`, and `a drafted journey a person marked (a step's note or covers)`.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The copy was compared with the draft, whether it stands as the draft or otherwise. With `--refresh`: the copy was written. |
| `1` | No rebuild manifest, a copy that cannot be read, or a draft that is missing or cannot be read. With `--refresh`: the refresh was refused, and nothing was written. A folder without the Pro plan is refused too. |

## Files

| File | What the command does with it |
|---|---|
| `docs/codelead/rebuild/rebuild.json` | Reads the manifest: the old application's folder (`source.repository`), and the copy's file when `run.journeys` names another one. |
| `docs/codelead/rebuild/journeys.json` | Reads the copy. `--refresh` writes it. |
| `docs/codelead/rebuild/journeys.draft.json` in the old application | Reads the draft. Never writes it. |

## See also

- [`codelead rebuild init`](codelead-rebuild-init.md): makes the copy.
- [`codelead model`](codelead-model.md): writes the draft in the old application.
- [`codelead rebuild run`](codelead-rebuild-run.md): the run that walks the copy.
- [The rebuild](../modernization/rebuild.md#the-journeys-from-three-sources)
- [The System Model](../modernization/system-model.md)
- [Journeys format](../reference/journeys-format.md)
- [Passwords](../analyst/passwords.md)
