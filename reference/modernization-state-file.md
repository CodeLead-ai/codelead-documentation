---
title: Modernization state file
description: The format of .codelead/modernization.json, codelead.modernization/1 - the application, the mode, the answers, the eight stages with their statuses, problems and coverage, and the fields of a skipped stage.
kind: reference
area: reference
date: 2026-11-20
---

# Modernization state file

**Applies to:** CodeLead 0.1 and later

`.codelead/modernization.json` holds the state of the [modernization program](../modernization/the-program.md) of one old application. It lists the eight stages and the status of each. It keeps the problem of a stage that failed or waits, and the answers that you gave. CodeLead writes it after every change of a stage. The extension's [Modernize wizard](../extension/modernize-wizard.md) reads it, and runs the commands that it names.

## The file

| Fact | Value |
|---|---|
| Path | `.codelead/modernization.json` in the old application's folder. |
| Schema | `codelead.modernization/1`. Required. |
| Written by | [`codelead modernize`](../cli/codelead-modernize.md) `start`, `next`, `run`, `continue`, `answer`, `skip` and `stop`, and the same commands of [`/modernize`](../chat-commands/modernize.md) in the chat. |
| Read by | `codelead modernize status`, every program command, and the wizard. |
| In git | No, ignored. It changes while a stage runs, and it says this machine's run. CodeLead adds it to `.codelead/.gitignore`. |
| Encoding | JSON, two spaces, its keys always in the same order, a newline at the end. |

CodeLead writes the file whole: first to `.codelead/.modernization.json.<pid>.tmp`, then it renames it. A reader never sees half a file. Each writer reads the file again, changes only what it owns, and writes it back. A long stage never writes an old copy over an answer that you gave meanwhile.

## A state file

The Basic Store sample after the Read and Observe stages. The Pin stage was stopped, then skipped by a person. The program is at the next stage:

<!-- described, not captured -->
```json
{
  "schema": "codelead.modernization/1",
  "application": {
    "root": ".",
    "webProject": "webapp",
    "kind": "aspnet-mvc",
    "baseUrl": "http://localhost:5080/"
  },
  "mode": "approve",
  "answers": {
    "rebuildFolder": "../basic-store-rebuild",
    "template": "dotnet-next-shadcn",
    "framework": "net10.0",
    "identity": "keep",
    "database": "sample-seed",
    "security": "fix-none"
  },
  "current": "in-place",
  "stages": [
    {
      "id": "read",
      "title": "Read the application",
      "status": "done",
      "command": "codelead modernize run read",
      "chat": "/modernize --run read",
      "startedAt": "2026-11-20T09:00:01.000Z",
      "finishedAt": "2026-11-20T09:04:12.000Z",
      "outputs": ["docs/codelead/dossier/DOSSIER.md", "docs/codelead/dossier/options.md"],
      "evidence": ["ev-20261120090412-k2m9qa"],
      "problem": null
    },
    {
      "id": "recordings",
      "title": "Record the journeys",
      "status": "done",
      "command": "codelead modernize run recordings",
      "chat": "/modernize --run recordings",
      "startedAt": null,
      "finishedAt": "2026-11-20T09:30:02.000Z",
      "outputs": [],
      "evidence": ["ev-20261120090413-p0x7rb", "ev-20261120093002-c4n8vd"],
      "problem": null
    },
    {
      "id": "pins",
      "title": "Pin the contracts",
      "status": "skipped",
      "command": "codelead modernize run pins",
      "chat": "/modernize --run pins",
      "startedAt": "2026-11-20T09:30:03.000Z",
      "finishedAt": "2026-11-20T10:15:00.000Z",
      "outputs": [],
      "evidence": ["ev-20261120094510-h3t6ma", "ev-20261120101500-s8d1wc"],
      "problem": null,
      "skippedBy": "person",
      "skipReason": "The pins wait for a copy of the database; the team goes on with the transforms first.",
      "skippedAt": "2026-11-20T10:15:00.000Z"
    },
    {
      "id": "in-place",
      "title": "Transform in place",
      "status": "pending",
      "command": "codelead modernize run in-place",
      "chat": "/modernize --run in-place",
      "startedAt": null,
      "finishedAt": null,
      "outputs": [],
      "evidence": [],
      "problem": null
    },
    …
  ],
  "updatedAt": "2026-11-20T10:15:00.000Z"
}
```

[`codelead modernize status`](../cli/codelead-modernize-status.md) prints the same state in words:

```output
The modernization program of webapp (aspnet-mvc), approve mode, at http://localhost:5080/; updated 2026-11-20T10:15:00.000Z.
  1. Read the application: done (09:00Z–09:04Z)
  2. Record the journeys: done
  3. Pin the contracts: skipped (09:30Z–10:15Z)
       Skipped by the person at 2026-11-20T10:15:00.000Z: The pins wait for a copy of the database; the team goes on with the transforms first.
  4. Transform in place: pending  ← the program is here
  5. Extract the model: pending
  6. Initialize the rebuild: pending
  7. Build and verify: pending
  8. Write the audit pack: pending
Answers: rebuildFolder=../basic-store-rebuild, template=dotnet-next-shadcn, framework=net10.0, identity=keep, database=sample-seed, security=fix-none.
Next: codelead modernize next (Transform in place)
```

The optional answers, here `security`, follow the five that the file always holds.

`codelead modernize status --json` prints the file as it is.

## Top-level keys

| Key | Type | What it holds |
|---|---|---|
| `schema` | string | `codelead.modernization/1`. |
| `application` | object | The old application. See [application](#application). |
| `mode` | `approve` or `unattended` | `approve`: CodeLead asks before it applies a change. `unattended`: it takes the recommended answer itself. `codelead modernize start --unattended` sets it. |
| `answers` | object | Your answers. See [Answers](#answers). |
| `current` | stage id | The stage that the program is at: the first stage that is not done or skipped, or the stage that failed or waits. |
| `stages` | list | The eight stages, always in the program's order. See [Stages](#stages). |
| `updatedAt` | string | When the file last changed, as an ISO time in UTC. |

## application

| Key | Type | What it holds |
|---|---|---|
| `root` | string | Always `"."`: the folder that holds `.codelead/`. |
| `webProject` | string or `null` | The old web project's name. |
| `kind` | string | The kind of application, such as `aspnet-mvc`, or `unknown`. |
| `baseUrl` | string or `null` | The address where the old application answers, from `--base` or the [run profile](run-profile.md#the-old-applications-run-profile). |
| `accounts` | list, optional | The roles that the read stage's Explorer signs in as, from `codelead modernize start --as` or `/modernize --run --as`. Each entry is `{ "role": "clerk", "account": "clerk@example.test" }`. It never holds a password: the stage reads each password when it runs, from `CODELEAD_WALK_PASSWORD_<ROLE>`, else from the file that `CODELEAD_WALK_CREDENTIALS` names. Absent when no `--as` was given. A later `--as` replaces the list. |

## Stages

The stages, in their fixed order:

| # | `id` | `title` | `command` | `chat` |
|---|---|---|---|---|
| 1 | `read` | Read the application | `codelead modernize run read` | `/modernize --run read` |
| 2 | `recordings` | Record the journeys | `codelead modernize run recordings` | `/modernize --run recordings` |
| 3 | `pins` | Pin the contracts | `codelead modernize run pins` | `/modernize --run pins` |
| 4 | `in-place` | Transform in place | `codelead modernize run in-place` | `/modernize --run in-place` |
| 5 | `model` | Extract the model | `codelead modernize run model` | `/modernize --run model` |
| 6 | `rebuild-init` | Initialize the rebuild | `codelead modernize run rebuild-init` | `/modernize --run rebuild-init` |
| 7 | `rebuild` | Build and verify | `codelead modernize run rebuild` | `/modernize --run rebuild` |
| 8 | `audit` | Write the audit pack | `codelead modernize run audit` | `/modernize --run audit` |

See [The eight stages](../modernization/the-eight-stages.md) for what each one does.

### The fields of a stage

| Key | Type | What it holds |
|---|---|---|
| `id` | string | The stage's id. |
| `title` | string | The stage's title. |
| `status` | string | See [Statuses](#statuses). |
| `command` | string | The line that runs the stage again in a terminal. |
| `chat` | string | The same line in the chat. |
| `startedAt` | string or `null` | When the stage last started. |
| `finishedAt` | string or `null` | When it last ended. |
| `outputs` | list of strings | The files that it wrote, relative to the folder, from all of its runs, each once. |
| `evidence` | list of strings | The ids of its [evidence records](evidence-record-format.md), in order. A stage that runs increments, such as the pins, lists theirs too. |
| `problem` | object or `null` | For a stage that failed or waits: what to do. See [The problem](#the-problem). |
| `coverage` | object | The Pin stage only, once it measured what the pins cover. See [The coverage of the pins](#the-coverage-of-the-pins). |
| `skippedBy` | `"person"` | Only on a stage that you skipped. See [A skipped stage](#a-skipped-stage). |
| `skipReason` | string | Only on a stage that you skipped: your reason. |
| `skippedAt` | string | Only on a stage that you skipped: when. |

`coverage`, `skippedBy`, `skipReason` and `skippedAt` are written only when they apply, after `problem`. A file without them stays the same, byte for byte.

### Statuses

| `status` | Meaning | What follows |
|---|---|---|
| `pending` | The stage has not run. | `codelead modernize next` runs it when its turn comes. |
| `running` | The stage runs now. | Wait, or stop it with `codelead modernize stop`. |
| `waiting` | The stage waits for you: the recordings, an answer, a reason for the extraction gate, a security decision. Your turn is not a failure. | Do what `problem.fix` says, then run `problem.retry`, or skip the stage. |
| `done` | The stage did its job. | The program goes on. |
| `failed` | The stage could not do its job. | Fix what `problem.reason` names, then run `problem.retry`, or skip the stage. |
| `skipped` | The program went past the stage: you skipped it, or a stub stood in for it. | The program goes on. |

A stage that starts again becomes `running`: its times reset, its problem and its skip fields go. Its earlier `outputs` and `evidence` are kept.

## The problem

Every stage that failed or waits carries a problem with four texts:

| Key | What it holds |
|---|---|
| `title` | What happened, in a few words. |
| `reason` | Why, in the stage's own words. |
| `fix` | What you can do. |
| `retry` | The one line that goes on after the fix. |

`retry` is always one of three forms, and the wizard runs it by itself: `codelead modernize next`, `codelead modernize continue`, or `codelead modernize run <stage>`.

The problem of the Observe stage, which waits for your recordings:

```json
{
  "title": "Record the journeys",
  "reason": "The Analyst's recordings are a person's: someone who knows the application walks its journeys while the Analyst records them",
  "fix": "Record each journey with the Analyst view's \"Record a journey\", or run codelead analyst record --base http://localhost:5080/; then continue",
  "retry": "codelead modernize continue"
}
```

The problem of a stage that you stopped:

```json
{
  "title": "Pin the contracts: stopped",
  "reason": "stopped by the person",
  "fix": "Run the stage again when you are ready; what it finished before the stop is kept",
  "retry": "codelead modernize run pins"
}
```

Other titles that you can meet: `<stage>: did not finish` (the process ended before the stage did), `An answer is needed`, `Answers are needed`, `The extraction gate needs your reasons`, and `<n> security findings are decisions for a person`.

## The coverage of the pins

The Pin stage writes `coverage` once it has measured the pins:

| Key | What it holds |
|---|---|
| `project` | The web project that the pins cover. |
| `counted` | `true` when the project has a narration, so that its facts are counted. |
| `memory`, `database` | For each pin level: `units` (`withTests` of `withFacts`), `facts` (`pinned` of `provable`) and `rules` (`pinned` of `provable`). |
| `walk` | The facts that no pin level proves, such as authorization attributes. The walk proves them. |
| `tests` | The tests of both characterization projects. |
| `units` | Without a narration: `withTests` of `all` units. |
| `pinsDatabase` | The database that the pins ran against, in words, or `null`. Never a password. |
| `increments` | The evidence ids of the landed pin increments. |
| `accepted` | When you went on with the pins as they stood (`pins=enough`). |

Example:

<!-- described, not captured -->
```json
"coverage": {
  "project": "webapp",
  "counted": true,
  "memory": { "units": { "withTests": 4, "withFacts": 4 }, "facts": { "pinned": 9, "provable": 9 }, "rules": { "pinned": 3, "provable": 3 } },
  "database": { "units": { "withTests": 4, "withFacts": 4 }, "facts": { "pinned": 10, "provable": 10 }, "rules": { "pinned": 4, "provable": 4 } },
  "walk": 8,
  "tests": 19,
  "pinsDatabase": "DefaultConnection: the file webapp.Characterization.Db/data/Store.mdf (attached to (LocalDb)\\MSSQLLocalDB)",
  "increments": ["ev-20261120094510-h3t6ma", "ev-20261120101204-q1w9ze", "ev-20261120102233-x7c2lp"]
}
```

See [Pins](../modernization/pins.md) and [Pin levels file](pin-levels-file.md).

## Answers

[`codelead modernize answer`](../cli/codelead-modernize-answer.md) records answers as `key=value` pairs. Nothing runs. There are two kinds:

- **Stored answers** are kept in `answers`. You can give one at any time, whatever stage waits. A stage reads it when it runs.
- **Acts** act on a stage, and are never kept. You can give one only while its stage waits or failed.

| Key | Kind | Values | What it decides |
|---|---|---|---|
| `rebuildFolder` | stored, always written | a folder beside the old application | Where the rebuild is made. The default is `../<folder name>-rebuild`. |
| `template` | stored, always written | a template's name or folder | The rebuild's template. The default is `dotnet-next-shadcn`. |
| `framework` | stored, always written | `net10.0` or `net8.0` | The new API's framework, and the target of the in-place upgrade. |
| `identity` | stored, always written | `keep` or `drop` | Whether the rebuild keeps the old accounts. |
| `database` | stored, always written | a connection string's name, or `sample-seed` | What the new API reads. |
| `pinsDatabase` | stored, once given | `done`, `env:<NAME>`, or the path of a copy of the database file | Where the database pins' own database is. `done` means that you wrote the connection in the pins' git-ignored connections file. Never a connection string. |
| `security` | stored, once given | `fix-none`, `fix-all` or `decided` | What the In place stage does with the security findings whose fix changes who may do what. `decided` means that you waived or fixed each one yourself. |
| `pins` | act | `enough` | Marks the Pin stage done as it stands, and you go on. |

An answer that the program always writes is `null` until you give it. An optional one appears only once given.

```console
codelead modernize answer security=fix-none
```

```output
Answers recorded: security=fix-none.
```

A value that the program does not take is refused, and nothing is recorded. The exit code is 2:

```output
Refused: security=maybe: fix-none (the findings that change who may do what stay open), fix-all (the in-place stage fixes them too) or decided (you waived or fixed each yourself). Nothing was recorded.
```

The answers `framework`, `identity` and `database` reach the rebuild's manifest. See [Rebuild manifest](rebuild-manifest.md).

## A skipped stage

[`codelead modernize skip`](../cli/codelead-modernize-skip.md) `<stage> --reason <WHY>` goes past a stage that failed or waits. In the chat, it is `/modernize --skip <stage> --reason <why>`. The stage then has:

| Key | Value |
|---|---|
| `status` | `skipped` |
| `finishedAt` | the time of the skip |
| `problem` | `null`: a skipped stage has no problem. |
| `skippedBy` | `"person"` |
| `skipReason` | your reason |
| `skippedAt` | the time of the skip |
| `evidence` | the stage's earlier records, and the skip's own record |

The skip writes a `modernize` evidence record with `status: "skipped"`, your reason, the status that the stage had and its problem. See [Evidence record format](evidence-record-format.md#the-fields-of-a-repository-read-and-of-the-program). The [audit pack](../modernization/audit-pack.md) states every skipped stage first in its README: who, when and why.

A stage that the program skipped itself, such as a stage that `CODELEAD_MODERNIZE_STUB` names, has none of the three skip fields.

The skip is refused, and nothing is recorded, when:

| The stage | The words |
|---|---|
| is `recordings` | `Refused: Record the journeys is not skipped: say the recordings are made (or that none will be): codelead modernize continue. Nothing was recorded.` |
| is `pending` | `Refused: Transform in place has not run (it is pending): a stage is skipped only when it failed or waits for you; run it first (codelead modernize run in-place). Nothing was recorded.` |
| is `running` | `… is running: wait for it to end, or stop it (codelead modernize stop); a stage that failed or waits can be skipped` |
| is `done` | `… is done: nothing to skip` |
| is skipped already | `Refused: Pin the contracts is skipped already (by the person: The pins wait for a copy of the database; the team goes on with the transforms first.): codelead modernize next goes on. Nothing was recorded.` |
| has an empty reason | `Refused: skipping <title> needs a reason: --reason <why> …` |

Without the `--reason` option at all, the command line itself is refused, with exit code 2: `error: required option '--reason <text...>' not specified`.

## How CodeLead reads the file

- **A stage that the file lacks** reads as `pending`. A file of an older program keeps working.
- **A `current` that is not a stage id** is computed again from the statuses.
- **A `mode` that is not `unattended`** reads as `approve`.
- **The file is refused** when it is not JSON, or when its `schema` is not `codelead.modernization/1`. It is refused too when a stage has a status that is not one of the six. Every program command then ends with exit code 2:

```output
Refused: .codelead/modernization.json: the stage pins has the status "paused", not one of pending, running, waiting, done, failed, skipped.
```

The wizard says: "The modernization state could not be read. Check .codelead/modernization.json, then refresh this page." Fix the file, or remove it to start the program again with `codelead modernize start`. The stages' evidence records stay in `.codelead/evidence/`.

## The stop file

`codelead modernize stop` writes `.codelead/modernization.stop`, with one line: `Stopped by the person at <time> (codelead modernize stop).` The running stage ends `failed` with the problem `<title>: stopped`, and no further stage runs. While the rebuild runs, `stop` also writes the rebuild run's own `.codelead/STOP` in the rebuild. Nothing is killed.

## See also

- [The modernization program](../modernization/the-program.md)
- [The eight stages](../modernization/the-eight-stages.md)
- [`codelead modernize`](../cli/codelead-modernize.md), [`codelead modernize status`](../cli/codelead-modernize-status.md) and [`codelead modernize answer`](../cli/codelead-modernize-answer.md)
- [The Modernize wizard](../extension/modernize-wizard.md)
- [Evidence record format](evidence-record-format.md)
- [Rebuild manifest](rebuild-manifest.md)
