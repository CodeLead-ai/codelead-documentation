---
title: Environment variables
description: Every CODELEAD_* variable and every key variable that CodeLead reads, with its values, what it overrides, and which ones are for comparison runs only.
kind: reference
area: reference
date: 2026-11-20
---

# Environment variables

**Applies to:** CodeLead 0.1 and later

CodeLead reads some settings from the environment. A variable wins over the [configuration file](configuration.md) for one run, without a change to a file. This page lists every variable that the product reads, grouped by what it changes.

> [!NOTE]
> **New to this?** An environment variable is a name and a value that a shell gives to the programs it starts. You set one for one command with `CODELEAD_MODEL=qwen/qwen3.6-35b-a3b codelead config` on macOS and Linux, or with `$env:CODELEAD_MODEL="qwen/qwen3.6-35b-a3b"` in PowerShell.

## Where CodeLead reads them

- **The process environment**: the shell, the program that starts CodeLead, or the extension.
- **`.env`** in the folder where CodeLead runs. CodeLead loads it when it starts. A value that the environment already has wins over the file.
- **A rebuild's run profile**: `run.unattended.env` of the [rebuild manifest](rebuild-manifest.md) sets `CODELEAD_*` switches for an unattended run. It cannot set a key or a password. See [Run profile](run-profile.md#rununattended).

[`codelead doctor`](../cli/codelead-doctor.md) lists the `CODELEAD_*` switches in effect, and warns when a model setting comes from the environment. [`codelead config`](../cli/codelead-config.md) says, after each value, which variable set it. A key's value is never shown: CodeLead says only whether it is set.

> [!IMPORTANT]
> Keep keys in `.env` or in your shell, never in `.codelead/config.json`. [`codelead init`](../cli/codelead-init.md) adds `.env` to `.gitignore`, so that the key stays out of git.

## Models and providers

| Variable | Values | What it does | What it overrides |
|---|---|---|---|
| `CODELEAD_MODEL` | a model id | The model of every role. | `modelRoles.<role>.model`. Refused when a role's provider is not the default role's: see [the override rule](#the-override-rule). |
| `CODELEAD_MODEL_<ROLE>` | a model id | The model of one role. `<ROLE>` is `DEFAULT`, `FAST`, `PREFLIGHT`, `ARCHITECT`, `DEEP`, `CRITIC` or `REPAIR`. | `CODELEAD_MODEL` and the file, for that role. |
| `CODELEAD_PROVIDER` | a provider's name | Moves every role to one provider, for example `anthropic`. | `modelRoles.<role>.provider`. It does not create the `repair` role. |
| `CODELEAD_PROVIDER_<ROLE>` | a provider's name | The provider of one role. `CODELEAD_PROVIDER_REPAIR=repair-local` puts the repairs on the second local server. | `CODELEAD_PROVIDER` and the file, for that role. |
| `CODELEAD_BASE_URL` | a URL | The address of the `lmstudio` provider only, for example `http://192.168.1.20:1234/v1`. | `providers.lmstudio.baseUrl`. |
| `CODELEAD_REPAIR_BASE_URL` | a URL | The address of the `repair-local` provider. | `providers.repair-local.baseUrl`. |
| `ANTHROPIC_BASE_URL` | a URL | The Anthropic SDK's endpoint, for an `anthropic` provider without `baseUrl`. The SDK reads it. | The SDK's default endpoint. |

`CODELEAD_MODEL_REPAIR` and `CODELEAD_PROVIDER_REPAIR` make a `repair` role when the file has none. Each one sets only its own field. The other field stays the strategy role's.

### The override rule

`CODELEAD_MODEL` names one model for every role. A role on another provider than the default role cannot know that model. CodeLead then refuses before any model call, and ends with exit code 2:

```output
CODELEAD_MODEL names qwen/qwen3.6-35b-a3b, but the repair role's provider is anthropic (the default role's is lmstudio): a generic model variable does not apply across providers. Unset CODELEAD_MODEL, or set CODELEAD_MODEL_REPAIR for that role, or remove the role from the file.
```

A per-role variable, such as `CODELEAD_MODEL_REPAIR`, is never refused.

## Keys

A provider names its key variable in `apiKeyEnv`. CodeLead sends that variable's value. When the variable is not set, CodeLead sends the provider's `defaultApiKey`, when it has one.

| Variable | Read for | Notes |
|---|---|---|
| `ANTHROPIC_API_KEY` | the `anthropic` provider | Needed for `CODELEAD_PROVIDER=anthropic` or any role on Anthropic. |
| `OPENROUTER_API_KEY` | the `openrouter` provider | |
| `CODELEAD_LMSTUDIO_API_KEY` | the `lmstudio` provider | A local server takes any key. The default is the placeholder `lm-studio`. |
| `CODELEAD_API_KEY` | the `lmstudio` provider | Replaces the provider's default key. `CODELEAD_LMSTUDIO_API_KEY` still comes first. |
| `CODELEAD_REPAIR_API_KEY` | the `repair-local` provider | The default is the placeholder `local`. |
| any name in `apiKeyEnv` | a provider that you added | For example `MY_GATEWAY_KEY`. |

A key is never written to a session log, an evidence record or the session's parameters. `codelead doctor` shows `set` or `not set`.

## Privacy and consent

| Variable | Values | What it does |
|---|---|---|
| `CODELEAD_ALLOW_REMOTE_MODELS` | `1`, `true`, `yes` | Your consent to send code to a remote provider in this run. It sets `privacy.allowRemoteModels` to `true` and turns off the question before the first remote call. The session's parameters record it. |
| `CODELEAD_ACCEPT_TERMS` | `yes` | Answers yes to the evaluation terms of a packaged release, for a run that nobody types into, such as a scheduled job. |

Set `CODELEAD_ALLOW_REMOTE_MODELS` only when the code's owner agrees that the code may go to that provider. See [Privacy and remote models](../concepts/privacy-and-remote-models.md).

## Thinking, effort and call budgets

| Variable | Values | What it does | What it overrides |
|---|---|---|---|
| `CODELEAD_REASONING_EFFORT_<ROLE>` | `low`, `medium`, `high`, `xhigh`, `max` | The reasoning effort that the calls of one role ask for. `<ROLE>` is `DEFAULT`, `FAST`, `PREFLIGHT`, `ARCHITECT`, `DEEP`, `CRITIC` or `REPAIR`. Unset: the server's own default. | The server's setting. |
| `CODELEAD_REASONING_EFFORT_PATCH` | the same | The effort of every call that returns a patch, whatever its role. | The role's effort. |
| `CODELEAD_REASONING_EFFORT_INNER_LOOP` | the same | The effort of the turns of the governed inner loop of a new app. | The patch effort. |
| `CODELEAD_REASONING_EFFORT_REPAIR` | the same | The effort of every repair call. | The patch effort. |
| `CODELEAD_BUDGET_PROFILE` | `fast`, `balanced`, `quality` | The table of call budgets. | `callBudgets.profile`. |
| `CODELEAD_MAX_THINKING_<ROLE>` | a whole number | The thinking cap of one role's calls, in tokens. | The profile's value. |
| `CODELEAD_MAX_SECONDS_<ROLE>` | a whole number | The longest one call of the role may run, in seconds, at most 1800. | `callBudgets.perRole.<role>.maxSeconds`. |
| `CODELEAD_EXTRA_BODY_JSON` | a JSON object | Fields added to the body of every request to the provider. A value that is not JSON stops the call with `CODELEAD_EXTRA_BODY_JSON is not valid JSON`. | Nothing. |
| `CODELEAD_MODEL_WINDOW_LOOKUP` | `0` | Stops CodeLead from asking the server for the model's context window. | |

An OpenAI-compatible provider sends `high` for `xhigh` and `max`. `CODELEAD_REASONING_EFFORT_PREFLIGHT` is both a role and a stage: it also sets the effort of every preflight call.

## Prompt sizes

| Variable | Values | What it does | Default |
|---|---|---|---|
| `CODELEAD_PROMPT_MAX_CHARS` | a whole number, 20000 or more | The total size of a patch or repair prompt, in characters. | From `contextTokens` of the role, else 200,000. |
| `CODELEAD_RETRY_PROMPT_MAX_CHARS` | a whole number, 20000 or more | The total size of a retry prompt after the patch check refused a patch. | `CODELEAD_PROMPT_MAX_CHARS`, else from `contextTokens` of the role, else from the window that the provider reports for the model, else 90,000. |
| `CODELEAD_PATCH_CONTEXT_BUDGET` | a number of tokens | A cap on the repository context of the patch prompt. | Off. |

A window gives a budget of `contextTokens × 3.5 − 24,000` characters, and never less than 20,000. Only the retry asks the provider for the model's window, and only when no variable and no `contextTokens` gives the budget. `CODELEAD_MODEL_WINDOW_LOOKUP=0` stops that question. Under `CODELEAD_PATCH_CONTEXT_BUDGET`, the retry takes the smaller of the two budgets.

## Folders

| Variable | Default | What moves |
|---|---|---|
| `CODELEAD_KNOWLEDGE_DIR` | `.codelead` | The evidence records (`<dir>/evidence/`), the knowledge base and the project library's index. |
| `CODELEAD_SESSIONS_DIR` | `.codeleadsessions` | The session logs and the increment ledger. |
| `CODELEAD_DOCS_DIR` | `docs/codelead` | The project's documents: the repository map, the catalog and the project log. |
| `CODELEAD_LOCKS_DIR` | `<home>/.codelead/locks` | The machine's locks of walk databases. |
| `CODELEAD_TEMPLATES_DIR` | none | A folder of templates by name, for `codelead rebuild init --template <name>`. |
| `CODELEAD_PKB_HOME` | `<home>/.codelead/pkb` | The folder of the PKB store: `pkb.json`, the env file with its passwords, and its compose file. `--dir` of [`codelead pkb store`](../cli/codelead-pkb-store.md) comes first. |

A relative path is read from the project's folder. See [Files and folders](files-and-folders.md).

## Sessions and logs

| Variable | Values | What it does |
|---|---|---|
| `CODELEAD_DISABLE_SESSION_LOGGING` | `1` | Writes no session log. |
| `CODELEAD_SESSION_LOGGING` | `0` | The same. |
| `CODELEAD_READ_ONLY` | `1`, `true`, `yes` | Keeps the session log, and never edits the repository's `.gitignore`. The session says once that the sessions folder is not ignored. Use it in a checkout that you must not change. |
| `CODELEAD_PROJECT_DOCS` | `0` | Stops the refresh of `docs/codelead/CATALOG.md`, `PINNED.md` and `PROJECT-LOG.md` at the end of a run. |
| `CODELEAD_DEBUG_REPL` | `1` | Prints the input loop's own debug lines. |

## Patches, validation and checkpoints

| Variable | Values | What it does |
|---|---|---|
| `CODELEAD_SKIP_PREFLIGHT` | `1` | `/implement` goes from the request straight to the patch call. |
| `CODELEAD_CHECKPOINT_COMMITS` | `1` | Commits each landed increment. The commit message starts with `codelead: /implement`. |
| `CODELEAD_CHECKPOINT_BRANCH` | `0` | With checkpoint commits on, commits on the current branch. By default the commit lands on a `codelead/…` work branch. |
| `CODELEAD_EDIT_FORMAT` | `search_replace` | Asks the model for search and replace blocks in place of a unified diff. |
| `CODELEAD_WHOLE_FILE_SMALL` | `1` | Lets the model write a small file whole. |
| `CODELEAD_FORMAT_ON_APPLY` | `0` | Stops the repository's formatter from running on the files that a patch changed. |
| `CODELEAD_CONVENTIONS_CARD` | `0` | Leaves the conventions card out of the patch prompt. |
| `CODELEAD_TYPE_CONTEXT` | `0` | Leaves type declarations out of the context. |
| `CODELEAD_INSTALL_TIMEOUT_MS` | milliseconds | How long a dependency install of the gate may run. The default is 15 minutes. |

## Repository map, specs and modernization

| Variable | Values | What it does |
|---|---|---|
| `CODELEAD_REPOSITORY_MAP` | `0` | Keeps the [repository map](../concepts/repository-map.md) out of every reader for a whole run: the preflight, the reuse of the library, the chat and the request lint. |
| `CODELEAD_SPEC_WALK` | `0`, `off`, `false`, `no`, `disabled` | Stops the walk of an increment's spec in a new app. |
| `CODELEAD_SPEC_COVERAGE` | the same words | Stops the spec coverage check of a new app. |
| `CODELEAD_ANALYST` | a command line | The Analyst to run, split at spaces. By default CodeLead runs the Analyst that ships with it. |
| `CODELEAD_SOURCE_LINK` | `0` | Stops a rebuild's requests from carrying what the old application's code does. |
| `CODELEAD_MODERNIZE_STUB` | stage ids, comma-separated | The program ends those stages as skipped, with no model call, for a test of the program. It takes only the stages that call a model: `pins`, `in-place` and `rebuild`. |

## The walk's passwords

| Variable | Values | What it does |
|---|---|---|
| `CODELEAD_WALK_PASSWORD_<USER>` | a password | The password of one account of a [journey](journeys-format.md) under a given identity. `<USER>` is the user's key in capitals, for example `CODELEAD_WALK_PASSWORD_CLERK` for the user `clerk`. |
| `CODELEAD_WALK_CREDENTIALS` | a file path | A JSON file, outside the repository, that holds the passwords by account, for example `{ "STORE\\walk-clerk": "<PASSWORD>" }`. A `CODELEAD_WALK_PASSWORD_<USER>` wins over it. |

`run.unattended.credentials` of a rebuild's run profile sets `CODELEAD_WALK_CREDENTIALS` for an unattended run. The walk's report says the placeholder, never a password. See [Passwords](../analyst/passwords.md).

## The model ladder

The ladder of a rebuild run climbs from the compose model to a repair model, then to an intervention. These variables override the run profile's `run.models` for one run. See [Run profile](run-profile.md#runmodels).

| Variable | What it overrides |
|---|---|
| `CODELEAD_LADDER_REPAIR_MODEL` | `run.models.repair.model`. Turns the ladder on when the profile has no `models`. |
| `CODELEAD_LADDER_REPAIR_PROVIDER` | `run.models.repair.provider`. |
| `CODELEAD_LADDER_INTERVENE_MODEL` | `run.models.intervene.model`. |
| `CODELEAD_LADDER_INTERVENE_PROVIDER` | `run.models.intervene.provider`. |
| `CODELEAD_LADDER_INTERVENE_EFFORT` | `run.models.intervene.effort`. |

These are not `CODELEAD_MODEL_REPAIR` and `CODELEAD_PROVIDER_REPAIR`. Those set the repair role, which answers every repair prompt.

## New apps

| Variable | Values | What it does | What it overrides |
|---|---|---|---|
| `CODELEAD_POLISH_ON_REPAIR` | `1`, `true`, `yes`, or `0`, `false`, `no` | The final polish step runs on the repair role's model. | `greenfield.polishOnRepairRole`. |
| `CODELEAD_REVIEW_PASS_AFTER_BUILD` | the same | One more step reviews the built app on the repair role's model. | `greenfield.reviewPassAfterBuild`. |
| `CODELEAD_INCREMENT_PLANNER_TEMP` | a number from 0 to 2 | The temperature of the increment planner. | `incrementPlanner.temperature`. |

### Switches for comparison runs

These switches exist to compare two ways of building a new app. Leave them unset for normal work.

| Variable | Values | What it does |
|---|---|---|
| `CODELEAD_SKELETON` | `deterministic` | Writes the first skeleton of the app without a model call. |
| `CODELEAD_INCREMENT_ACCEPTANCE_PROBE` | `0`, `off`, `false`, `no`, `disabled` | Turns off the probe that opens each increment in a browser. |
| `CODELEAD_CUMULATIVE_PROBES` | the same words | Turns off the probe of earlier increments after each new one. |
| `CODELEAD_GREENFIELD_AUTO_REVISE` | the same words | Turns off the revise-and-continue cycle of `/create-app --auto`. |
| `CODELEAD_PROBE_AUTHORITY` | `gate` | Every finding of the acceptance probe and the regression probe refuses the attempt. The increment fails when its 3 attempts are spent. `/build-incremental` and the increment run driver, `scripts/incrementalBuildRun.ts`, record it as failed, put its changes back to the last checkpoint, and go on. Any other value means the default, `evidence`: a probe warns with its reason, and keeps work that builds and passes its tests. A verified page that now renders nothing is refused either way. The variable wins over `greenfield.probeAuthority`. Both read the setting once, at the start of the run. |
| `CODELEAD_NORTH_STAR` | `1` | Carries the whole request into every increment's prompt. |
| `CODELEAD_INNER_LOOP` | `1` | Builds each increment in a loop of model turns that act, look and fix. |
| `CODELEAD_BUILD_THEN_VERIFY` | `1` | Builds the whole app in one increment, then audits it. `CODELEAD_BUILD_TURNS` and `CODELEAD_BUILD_MINUTES` bound the loop (40 turns and 100 minutes by default). It needs `CODELEAD_INNER_LOOP=1` and `CODELEAD_NORTH_STAR=1`. |
| `CODELEAD_PLAN_GRANULARITY` | `coarse` | Plans fewer, larger increments: 4 to 7. |
| `CODELEAD_DESIGN_DIRECTION` | `1` | Asks the planner for a design direction. |
| `CODELEAD_RULE_ORACLE` | `1` | Derives the request's business rules from its words alone, tests the app against them, and reports a failing rule as evidence. |
| `CODELEAD_PROFILE` | `none`, `auto`, `capable`, `constrained` | Sets several of the switches above at once, for a class of model. `auto` picks by `CODELEAD_MODEL`. |
| `CODELEAD_VERBOSE_RECENT` | a whole number from 1 to 20 | How many recent increments the ledger describes in full. The default is 3. |

The session's parameters record the switches that change what a run does, so that a reader of the session sees them. Keys and machine paths are never recorded.

## Variables that CodeLead sets

- **`CODELEAD_EOL_REFRESH`**: set by CodeLead for its own survey when you run `/modernize --plan --eol-refresh`. Do not set it yourself.
- **`FORCE_COLOR=0` and `NO_COLOR=1`**: set by the extension when it starts `codelead interactive --json-io`. See [Session protocol](session-io-protocol.md).

The project's own test and benchmark scripts read a few more `CODELEAD_*` variables. They change nothing in the product, and this page leaves them out.

## Examples

- Point the default role at another LM Studio server for one command:

  ```console
  CODELEAD_BASE_URL=http://127.0.0.1:9/v1 codelead doctor
  ```

  ```output
  warn     Model settings overridden by the environment          lmstudio's base URL is http://127.0.0.1:9/v1 from CODELEAD_BASE_URL (the file says http://localhost:1234/v1)  → If it is not meant for this folder, remove CODELEAD_BASE_URL from the shell's profile or the program that starts CodeLead, then open a new terminal. codelead config shows every value with its source.
  FAIL     Model server at http://127.0.0.1:9/v1                 Could not connect to lmstudio at http://127.0.0.1:9/v1.  → Start LM Studio's server (Developer tab); for a server on another machine turn on "Serve on Local Network" there and set CODELEAD_BASE_URL=http://<host>:1234/v1 here.
  …
  ok       CODELEAD_* switches in effect                         CODELEAD_BASE_URL=http://127.0.0.1:9/v1
  ```

- Run every role on Anthropic, with the key in `.env`:

  ```console
  CODELEAD_PROVIDER=anthropic CODELEAD_MODEL=claude-sonnet-5-5 CODELEAD_ALLOW_REMOTE_MODELS=1 codelead interactive
  ```

- Keep the patches on the local model and send the repairs to the second local server:

  ```console
  CODELEAD_PROVIDER_REPAIR=repair-local CODELEAD_REPAIR_BASE_URL=http://localhost:8080/v1 codelead interactive
  ```

- A `.env` file for a project that uses Anthropic for repairs. The key's value is yours:

  ```text
  ANTHROPIC_API_KEY=<YOUR_API_KEY>
  CODELEAD_MODEL_REPAIR=claude-opus-5-5
  CODELEAD_PROVIDER_REPAIR=anthropic
  CODELEAD_ALLOW_REMOTE_MODELS=1
  ```

[`codelead setup`](../cli/codelead-setup.md) writes a starter `.env.example`, with every line commented.

## See also

- [Configuration file](configuration.md)
- [`codelead config`](../cli/codelead-config.md), [`codelead doctor`](../cli/codelead-doctor.md) and [`codelead setup`](../cli/codelead-setup.md)
- [Models and roles](../concepts/models-and-roles.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
- [Set up remote models](../how-to/set-up-remote-models.md)
- [Unattended runs and the run lock](../concepts/unattended-runs.md)
