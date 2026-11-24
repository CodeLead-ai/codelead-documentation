---
title: Files and folders
description: Every file and folder that CodeLead writes in a project and in your home folder, which command writes it, its format, and whether it belongs in git.
kind: reference
area: reference
date: 2026-11-24
---

# Files and folders

**Applies to:** CodeLead 0.1 and later

CodeLead keeps everything it knows about a project in plain files inside the project: settings, [evidence records](../concepts/evidence-records.md), session logs, documents and reports. This page lists each file, the command that writes it, its schema when it has one, and whether it belongs in git. The examples come from a folder of the Basic Store sample.

> [!NOTE]
> **New to this?** "In git: yes" means that you commit the file, so that your team and your reviewers see it. "In git: no" means that the file belongs to one machine or one run. "No, ignored" means that CodeLead adds the file to a `.gitignore` itself. For a file marked only "No", add it to your own `.gitignore`.

## After codelead init

[`codelead init`](../cli/codelead-init.md) in an empty folder that holds only `package.json` gives this tree:

```text
<your project>/
  .gitignore                      .env lines added by init
  .codelead/
    config.json                   the configuration
    knowledge.json                the knowledge base
    evidence/
      ev-20261120070542-2ss1d4.json   the record of the repository read
    sessions/                     empty: preflight sessions go here
  docs/codelead/
    repository-map.json           the repository map, as data
    repository-map.md             the repository map, for people
    CATALOG.md                    the documents CodeLead keeps
    catalog.json
```

The first chat session adds `.codeleadsessions/` and its line in `.gitignore`. Its end adds `docs/codelead/PINNED.md` and `docs/codelead/PROJECT-LOG.md`.

## The .codelead folder

| Path | Written by | Schema | In git | What it holds |
|---|---|---|---|---|
| `.codelead/config.json` | `codelead init`, `codelead modernize source`, the extension's Settings | `codelead.config/1` | Yes | The settings. See [Configuration file](configuration.md). |
| `.codelead/knowledge.json` | every run that learns a fact, and `init` | `codelead.knowledge/1` | Yes | The [knowledge base](../concepts/knowledge-base.md): the entries and their evidence. |
| `.codelead/evidence/ev-<time>-<id>.json` | every run | `codelead.evidence/1` | Yes | One [evidence record](evidence-record-format.md) per run. |
| `.codelead/disclosures.jsonl` | [`codelead pkb sync`](../cli/codelead-pkb-sync.md), [`codelead pkb embed`](../cli/codelead-pkb-embed.md), [`codelead pkb ask`](../cli/codelead-pkb-ask.md) | | Yes | One line for each sync that changed something in the PKB store, and for each send of texts to a remote embeddings or chat server: what left, as file names, ids, hashes and counts, never the text. |
| `.codelead/project-index.json` | the project library | `codelead.project-index/1` | No | The index of the [project library](../concepts/project-library.md). CodeLead writes it again when the repository changes. |
| `.codelead/validation.json` | you, and the pins and `rebuild init` when they add an entry | `codelead.validation/1` | Yes | The gate's commands. See [Validation file](validation-file.md). |
| `.codelead/run.json` | you | `codelead.run/1` | Yes | How to start the application for a spec walk. See [Run file](run-file.md). |
| `.codelead/CONVENTIONS.md` | you, from the starter that the extension or `rebuild init` writes | | Yes | Your team's rules, in plain words. See [Conventions](../concepts/conventions.md). |
| `.codelead/conventions.json` | [`codelead conventions`](../cli/codelead-conventions.md) | `codelead.conventions/1` | Yes | The rules that CodeLead read from the code. |
| `.codelead/template.json` | `/create-app --template` | | Yes | The template that the new app was built from. |
| `.codelead/sessions/<time>-preflight/` | [`codelead preflight`](../cli/codelead-preflight.md) | `codelead.preflight-*/1` | No | A saved preflight: `repo-context.json`, `strategy-classification.json`, `clarification-answers.json`, `implementation-spec.json`, `preflight-result.json`. |
| `.codelead/run-journal.json` | every increment build | `codelead.run-journal/1` | No, ignored in a rebuild | What happened to each increment. |
| `.codelead/run-report.html` | every increment build, and `codelead rebuild run` | | No, ignored in a rebuild | The live run report. It reloads itself while a build runs. |
| `.codelead/spec-report.html` | [`codelead spec`](../cli/codelead-spec.md) `report`, `walk` and `coverage` | | No | The tester's page of every spec and its last walk. |
| `.codelead/specs/<name>/*.walk.json` | [`/implement --spec`](../chat-commands/implement.md) | `codelead.walk/1` | No | The walks of a spec after a landing. |
| `.codelead/acceptance/` | a walk that starts servers | | No, ignored | `servers.json` (`codelead.acceptance-servers/1`): the servers that CodeLead started, so that it can stop them. `servers/<role>.log`: each server's output, for example `source.log` for the old application. `walks/*.json`: the walks that judged a rebuild's parts. The folder holds a `.gitignore` of `*`. |
| `.codelead/run.lock` | `codelead rebuild run`, `codelead implement`, the modernization program | `codelead.run-lock/1` | No, ignored | One run per clone. It names this machine's process. |
| `.codelead/STOP` | you, and `codelead modernize stop` | | No | Makes a rebuild run stop after the part in flight. |
| `.codelead/rebuild-run-<stamp>.json`, `.md` | `codelead rebuild run` | `codelead.rebuild-run/1` | No, ignored | The run's summary. See [Rebuild run summary](rebuild-run-summary.md). |
| `.codelead/rebuild-run-<stamp>/` | the same | | No, ignored | The console of each session of the run. |
| `.codelead/rebuild-report.html` | every record of a rebuild, and [`codelead rebuild-report`](../cli/codelead-rebuild-report.md) | | No, ignored | The rebuild's progress report. |
| `.codelead/modernization.json` | the [modernization program](../modernization/the-program.md) | `codelead.modernization/1` | No, ignored | The program's stages and answers. See [Modernization state file](modernization-state-file.md). |
| `.codelead/modernization.stop` | `codelead modernize stop` | | No, ignored | Asks the running program to stop. |
| `.codelead/.gitignore` | CodeLead, the first time that it writes one of the ignored files above | | No, ignored | Keeps those files out of every commit. It ignores itself. |

The `.codelead/.gitignore` that a rebuild run writes, as an example:

```text
# CodeLead's unattended runs (codelead rebuild run): their summaries, consoles, journal and run report, rewritten at every session: kept out of every commit (checkpoints run git add -A)
/rebuild-run-*.json
/rebuild-run-*.md
/rebuild-run-*.tmp
/rebuild-run-*/
/run-journal.json
/run-journal-*.json
/run-report.html
/.gitignore
```

`CODELEAD_KNOWLEDGE_DIR` moves `knowledge.json`, `evidence/` and `project-index.json` to another folder. See [Environment variables](environment-variables.md#folders).

## The session folder

Each session of the chat writes a folder of its own. The folder is `.codeleadsessions/` at the repository root, and the first session adds the line `.codeleadsessions/` to `.gitignore`. `CODELEAD_SESSIONS_DIR` moves it. `CODELEAD_READ_ONLY=1` keeps the `.gitignore` as it is.

```text
.codeleadsessions/
  session-2026-11-20T07-18-21-503Z/
    session.jsonl          every event of the session, one JSON line each
    summary.json           the session's counts
    model/
      preflight-prompt-001.md   each prompt and each answer, by agent
    greenfield/            only for a new app
      state.json  spec.json  plan.json  target-folder.json  build.json  scaffold.json
    implement/
      state.json           what a later session resumes
  increment-ledger.json    what each increment of a build owns
  exports/
    session-2026-11-20T07-18-21-503Z.zip   an exported session
```

| Path | Schema | What it holds |
|---|---|---|
| `session.jsonl` | `codelead.session-log/1` on its first line | The events: input, model requests and answers, commands and their results. See [Sessions](../concepts/sessions.md). |
| `summary.json` | `codelead.session-summary/1` | The counts of the session. |
| `model/` | | The prompts and answers, in markdown. |
| `greenfield/*.json` | `codelead.greenfield-*/1` | The whole state of a new app's build. |
| `implement/state.json` | `codelead.implement-state/1` | The `/implement` that a later session can resume. |
| `increment-ledger.json` | `codelead.increment-ledger/1` | What each increment of a build owns and how it settled. |
| `exports/*.zip` | | The zips that `/session --export` and `codelead sessions export` write, when no `--out` is given. |

Session logs hold your prompts and the model's answers, which can hold code. They stay on the machine. [`codelead sessions export`](../cli/codelead-sessions.md) makes a zip of one in `exports/` when you want to share it. In git: no, ignored. [`codelead pr`](../cli/codelead-pr.md) never stages an exported zip, wherever it is.

## The docs/codelead folder

These are the project's documents. CodeLead writes them so that people read them in the repository. `CODELEAD_DOCS_DIR` moves the folder.

| Path | Written by | Schema | In git | What it holds |
|---|---|---|---|---|
| `repository-map.json`, `repository-map.md` | `codelead init`, [`codelead map`](../cli/codelead-map.md), [`/map`](../chat-commands/map.md), the end of a build | `codelead.repository-map/1` | Yes | The [repository map](../concepts/repository-map.md) and its freshness stamp. |
| `CATALOG.md`, `catalog.json` | every write of a document | `codelead.catalog/1` | Yes | Every document that CodeLead made or keeps, with its purpose. |
| `PINNED.md`, `pinned.json` | the end of a session | `codelead.pinned/1` | Yes | What waits on a person: decisions, answers, reviews. |
| `PROJECT-LOG.md`, `project-log.json` | the end of a session | `codelead.project-log/1` | Yes | What happened to the project, one entry per session. |
| `answers/<date>-<slug>.json` | [`/ask`](../chat-commands/ask.md) | `codelead.answer/1` | Yes | A recorded answer. |
| `licence.json` | the person who issues your licence | `codelead.licence/1` | Yes | The project's signed licence. See [Licence and plans](../concepts/licence-and-plans.md). |
| `board/` | [`codelead board init`](../pinboard/cli/board-init.md) | `codelead.board/1` | Yes, except `cursors/`, `.local/` and the lock | The [board](../pinboard/index.md). See [Files and events](../pinboard/files-and-events.md). |
| `specs/` | you, and [`codelead spec`](../cli/codelead-spec.md) | | Yes | The specs in plain words, and their drafts and walks. See [Specification files](spec-files.md). |
| `security-waivers.json` | [`/modernize --waive`](../chat-commands/modernize.md) | `codelead.security-waivers/1` | Yes | The security findings that you decided not to fix. See [Extraction dispositions and security waivers](dispositions-and-waivers.md). |
| `parity-waivers.json` | you | | Yes | The page differences that you accept. See [Parity waivers](parity-waivers.md). |
| `dossier/` | `/modernize --plan`, the Read stage | `codelead.dossier-*/1` and others | Yes | The [dossier](../modernization/dossier.md): `DOSSIER.md`, `options.md`, the inventories, `functional-narration.json`, `pin-levels.json`. |
| `rebuild/` | [`codelead model`](../cli/codelead-model.md), `/modernize --rebuild-map`, `--rebuild-plan` | several | Yes | The [System Model](../modernization/system-model.md), the gate report, the feature map, the journeys draft, the schema, the plan. |
| `parity/` | the walk and `codelead parity` | `codelead.walk/1`, `codelead.rebuild-parity/1` | Yes | The last walk's report and the rebuild's parity. |
| `audit-pack/` | `/modernize --audit-pack`, the Audit stage | `codelead.audit-pack-manifest/1` | Yes | The [audit pack](../modernization/audit-pack.md), each file with its SHA-256. |

### The rebuild folder of the old application

[`codelead model`](../cli/codelead-model.md) and the modernization program write these under `docs/codelead/rebuild/` of the old application:

| File | Schema | What it holds |
|---|---|---|
| `system-model.json` | `codelead.system-model/1` | The System Model. |
| `extraction-gate.md` | | The extraction gate's report: each finding with the key to record a reason under. |
| `extraction-dispositions.json` | `codelead.extraction-dispositions/1` | The reasons that a person, or the program, recorded for the gate. |
| `feature-map.json`, `feature-map.md` | `codelead.rebuild-map/1` | The features of the old application. |
| `journeys.draft.json` | `codelead.journeys/1` | The journeys that the model drafted. |
| `truth.draft.json` | `codelead.system-model-truth/1` | The truth set drafted from the model, for a person to confirm. |
| `schema.json`, `schema.md` | `codelead.schema-from-code/1` | The database schema read from the code. |

### The rebuild

[`codelead rebuild init`](../cli/codelead-rebuild-init.md) makes a new repository. Beside the template's files, it writes:

| Path | Schema | In git | What it holds |
|---|---|---|---|
| `docs/codelead/rebuild/rebuild.json` | `codelead.rebuild/1` | Yes | The manifest and its run profile. See [Rebuild manifest](rebuild-manifest.md). |
| `docs/codelead/rebuild/journeys.json` | `codelead.journeys/1` | Yes | The merged journeys. See [Journeys format](journeys-format.md). |
| `docs/codelead/rebuild/rebuild-plan.json`, `.md` | `codelead.rebuild-plan/1` | Yes | The plan, its parts and its lanes. |
| `docs/codelead/rebuild/layout-proposal.json` | `codelead.layout-proposal/1` | Yes | The split of the C# controllers that the model proposed. |
| `scripts/reset-sample-seed.mjs` | | Yes | The day reset of a sample database, with `--database sample-seed`. |
| `.codelead/config.json`, `.codelead/validation.json`, `.codelead/CONVENTIONS.md` | | Yes | CodeLead initialized in the rebuild. |

## Files at the project's root

| Path | Written by | In git | What it holds |
|---|---|---|---|
| `.gitignore` | `codelead init` (the `.env` lines), the first session (`.codeleadsessions/`) | Yes | CodeLead only adds lines. It never removes one. |
| `.env` | you | **No** | Your keys and passwords, such as `ANTHROPIC_API_KEY`. CodeLead reads it when it starts. |
| `.env.example` | [`codelead setup`](../cli/codelead-setup.md) | Yes | A starter, with every line commented. It holds no key. |

The lines that `init` adds to `.gitignore`:

```text
# CodeLead: the key in .env is never committed
.env
.env.local
.env.*.local
```

## Your home folder

| Path | Written by | What it holds |
|---|---|---|
| `<home>/.codelead/accepted.json` | the first run of a packaged release | That you accepted the evaluation terms on this machine. `codelead doctor` and `codelead licence` run without it. |
| `<home>/.codelead/locks/walk-<key>.lock` | a rebuild run, `codelead implement` | One run per walk database on this machine. `CODELEAD_LOCKS_DIR` moves the folder. |
| `<home>/.codelead/pkb/` | [`codelead pkb store up`](../cli/codelead-pkb-store.md) | The PKB store: `pkb.json`, the env file `.env` with its passwords (readable by you only), `compose.yaml`, `mongod.yml` and `mongot.yml`. `pkb.json` also names the embeddings server ([`codelead pkb embed`](../cli/codelead-pkb-embed.md#the-embeddings-server)) and its two task prefixes, `embeddings.documentPrefix` and `embeddings.queryPrefix` ([the task prefixes](../cli/codelead-pkb-embed.md#the-task-prefixes)). Its `chat` section names the chat model of [`codelead pkb ask`](../cli/codelead-pkb-ask.md#the-chat-model). Its `bind`, `port` and `hosts` say where [`codelead pkb serve`](../cli/codelead-pkb-serve.md) listens, and the `.env` file holds the service's shared key, `PKB_SHARED_KEY`, when you set one. `CODELEAD_PKB_HOME` moves the folder. |
CodeLead writes nothing else outside the project, except a folder that you name. [`codelead pkb export`](../cli/codelead-pkb-export.md) writes a project of the PKB service into the folder of its `--out`. The folder holds `entries/<id>.md`, `questions/<id>.md`, the JSON files and `manifest.json` (`codelead.pkb-export/1`). [`codelead pkb import`](../cli/codelead-pkb-import.md) reads it back. Docker keeps the PKB store's data in its own volumes. The walk's passwords file, when you use one, is a file that you make and name with `CODELEAD_WALK_CREDENTIALS`. Keep it outside every repository.

## Files that CodeLead never writes

- Your source files, except through a patch that passed the [patch check](../concepts/the-patch-check.md) and that you applied.
- `.env` and other sensitive files: the patch check refuses a patch that names one.
- Anything outside the repository: the patch check refuses a path that leaves it.

## See also

- [Configuration file](configuration.md)
- [Evidence record format](evidence-record-format.md)
- [Sessions](../concepts/sessions.md)
- [Files and events](../pinboard/files-and-events.md) of the board
- [Files the Analyst writes](../analyst/files.md)
- [`codelead init`](../cli/codelead-init.md) and [`codelead sessions`](../cli/codelead-sessions.md)
