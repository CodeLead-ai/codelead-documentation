---
title: Configuration file
description: Every key of .codelead/config.json, with its type, its default, what reads it and an example, and the codelead config command that prints the result.
kind: reference
area: reference
date: 2026-11-20
---

# Configuration file

**Applies to:** CodeLead 0.1 and later

`.codelead/config.json` holds the settings of one project. It names the model providers, the model of each role and the strategies. It also holds the context limits, the privacy switches, and the settings of new apps and of a modernization. This page lists every key. The examples come from a folder initialized with [`codelead init`](../cli/codelead-init.md) for the Basic Store sample.

> [!NOTE]
> **New to this?** A *provider* is a service that runs models, such as a local LM Studio server or Anthropic. A *role* is a job inside CodeLead, such as `preflight` or `repair`, that gets a provider and a model. See [Models and roles](../concepts/models-and-roles.md).

## The file

| Fact | Value |
|---|---|
| Path | `.codelead/config.json` in the folder where you run CodeLead. |
| Schema | `codelead.config/1`. A file without `schema` reads as version 1. |
| Written by | [`codelead init`](../cli/codelead-init.md) (all of it, with the defaults), `codelead init --force` (all of it again), [`codelead modernize source`](../cli/codelead-modernize-source.md) (`modernization.source` only), and the [Settings page](../extension/settings.md) of the extension. |
| Read by | Every command that resolves the configuration. [`codelead config`](../cli/codelead-config.md) prints the result. |
| In git | Yes. The file holds no key. A key lives in `.env` or in the environment. |
| Encoding | JSON, UTF-8. |

No file is not an error. CodeLead then uses its built-in defaults, which are the same values that `init` writes.

## How CodeLead resolves the configuration

CodeLead reads the values in this order. A later step wins over an earlier one:

1. The built-in defaults.
2. `.codelead/config.json`, when the folder has one.
3. The [environment variables](environment-variables.md), such as `CODELEAD_BASE_URL` or `CODELEAD_MODEL_REPAIR`.
4. The model ladder's rung, only inside a rebuild run. See [Run profile](run-profile.md#runmodels).

Then CodeLead checks the result against the schema. A value of the wrong type stops every command with the key's path and the reason:

```output
Error:
Invalid config JSON at <your project>/.codelead/config.json.
- context.maxFiles: Number must be greater than 0
```

A file with a schema that this release does not read is refused in the same way:

```output
Error:
The config at <your project>/.codelead/config.json is in a format this CodeLead does not read (schema "codelead.config/2"; this release reads codelead.config/1).
```

Two older formats are still read: one with a single `provider` and `models.default`, and one with `routing` in place of `strategyModelRoles`. `codelead config` then prints a note that the file uses an older format.

## A whole file

This is the file that `codelead init` writes. The four strategies are shortened to one here:

```json
{
  "schema": "codelead.config/1",
  "providers": {
    "lmstudio": {
      "type": "openai-compatible",
      "baseUrl": "http://localhost:1234/v1",
      "apiKeyEnv": "CODELEAD_LMSTUDIO_API_KEY",
      "defaultApiKey": "lm-studio",
      "isRemote": false
    },
    "openrouter": {
      "type": "openai-compatible",
      "baseUrl": "https://openrouter.ai/api/v1",
      "apiKeyEnv": "OPENROUTER_API_KEY",
      "isRemote": true
    },
    "anthropic": {
      "type": "anthropic",
      "apiKeyEnv": "ANTHROPIC_API_KEY",
      "isRemote": true
    },
    "repair-local": {
      "type": "openai-compatible",
      "baseUrl": "http://localhost:8080/v1",
      "apiKeyEnv": "CODELEAD_REPAIR_API_KEY",
      "defaultApiKey": "local",
      "isRemote": false
    }
  },
  "modelRoles": {
    "default": { "provider": "lmstudio", "model": "" },
    "fast": { "provider": "lmstudio", "model": "" },
    "preflight": { "provider": "lmstudio", "model": "" },
    "architect": { "provider": "lmstudio", "model": "" },
    "deep": { "provider": "lmstudio", "model": "" },
    "critic": { "provider": "lmstudio", "model": "" }
  },
  "strategyModelRoles": {
    "fast": "fast",
    "clarify": "preflight",
    "architect": "architect",
    "deep": "deep"
  },
  "strategySettings": {
    "architect": {
      "temperature": 0.2,
      "maxTokens": 12000,
      "askQuestions": true,
      "maxQuestions": 5,
      "includeImplementationPlan": true,
      "includeRiskAnalysis": false,
      "includeValidationPlan": true,
      "includeNonGoals": true
    }
  },
  "context": {
    "maxFiles": 12,
    "maxSnippetCharsPerFile": 4000,
    "includeTests": true,
    "includePackageJson": true
  },
  "output": { "saveSessions": true },
  "privacy": {
    "allowRemoteModels": false,
    "confirmBeforeSendingRepoContextToRemote": true,
    "redactSensitiveFiles": true
  }
}
```

The `defaultApiKey` values `lm-studio` and `local` are placeholders that a local server accepts. They are not secrets.

## Top-level keys

| Key | Type | Required | What it holds |
|---|---|---|---|
| `schema` | string | No | `codelead.config/1`. |
| [`providers`](#providers) | object | Yes | The model servers, by name. |
| [`modelRoles`](#modelroles) | object | Yes | The provider and model of each role. |
| [`strategyModelRoles`](#strategymodelroles) | object | Yes | The role that each strategy uses. |
| [`strategySettings`](#strategysettings) | object | Yes | The temperature, the token cap and the questions of each strategy. |
| [`context`](#context) | object | Yes | How much of the repository a request carries. |
| [`privacy`](#privacy) | object | Yes | Whether code may go to a remote model. |
| [`output`](#output) | object | Yes | Whether a preflight saves its session. |
| [`incrementPlanner`](#incrementplanner) | object | No | The temperature of the increment planner of a new app. |
| [`callBudgets`](#callbudgets) | object | No | How long a model call may think. |
| [`greenfield`](#greenfield) | object | No | The switches and templates of new apps. |
| [`modernization`](#modernization) | object | No | The old application's run profile, the pins and the repair rounds. |

A key that the schema does not name is left out when CodeLead reads the file.

## providers

Each entry is one provider, by a name that you choose. A role names the provider by that name.

| Key | Type | Default | What it does |
|---|---|---|---|
| `type` | `openai-compatible` or `anthropic` | (required) | The protocol. `openai-compatible` covers LM Studio, llama-server, mlx_lm.server and OpenRouter. `anthropic` uses the Anthropic Messages API. |
| `baseUrl` | URL | (required for `openai-compatible`) | The server's address, for example `http://localhost:1234/v1`. An `anthropic` provider without it uses the SDK's endpoint. |
| `apiKeyEnv` | string | none | The environment variable that holds the key, for example `ANTHROPIC_API_KEY`. CodeLead reads it from the environment or from `.env`. |
| `defaultApiKey` | string | none | The key to send when the variable is not set. Use it only for a local server's placeholder. |
| `isRemote` | boolean | (required) | `true` for a server that is not on this machine. A remote provider is refused until `privacy.allowRemoteModels` is `true`. |

The four built-in providers:

| Name | Type | Address | Key variable | Remote |
|---|---|---|---|---|
| `lmstudio` | `openai-compatible` | `http://localhost:1234/v1` | `CODELEAD_LMSTUDIO_API_KEY` | No |
| `openrouter` | `openai-compatible` | `https://openrouter.ai/api/v1` | `OPENROUTER_API_KEY` | Yes |
| `anthropic` | `anthropic` | the SDK's endpoint | `ANTHROPIC_API_KEY` | Yes |
| `repair-local` | `openai-compatible` | `http://localhost:8080/v1` | `CODELEAD_REPAIR_API_KEY` | No |

`CODELEAD_BASE_URL` moves the address of `lmstudio` only. `CODELEAD_REPAIR_BASE_URL` moves the address of `repair-local`. `repair-local` is in every resolved configuration, even when the file leaves it out. See [Environment variables](environment-variables.md#models-and-providers).

## modelRoles

Each role has an entry with these keys:

| Key | Type | Default | What it does |
|---|---|---|---|
| `provider` | string | `lmstudio` | A name of `providers`. |
| `model` | string | `""` | The model's id, as the server names it. An empty string sends no model, and the server answers with the model that it has loaded. |
| `contextTokens` | whole number | none | The model's context window, in tokens. CodeLead sizes the patch, retry and repair prompts from it: `contextTokens × 3.5 − 24,000` characters, at least 20,000. |

The roles:

| Role | Required | Used for |
|---|---|---|
| `default` | Yes | The main model, and the chat. |
| `fast` | Yes | The `fast` strategy. |
| `preflight` | Yes | The `clarify` strategy and its questions. |
| `architect` | Yes | The `architect` strategy, the default of [`/implement`](../chat-commands/implement.md). |
| `deep` | Yes | The `deep` strategy. |
| `critic` | No | The plan critic of a new app. Without it, the critic uses the architect's provider and model. |
| `repair` | No | Every repair prompt: [`/repair`](../chat-commands/repair.md), the repair rounds of a rebuild, the validation repair of a new app. Each key that it leaves out is the strategy role's. |

Example: the default role on a local model, and the repairs on Anthropic. This configuration needs `privacy.allowRemoteModels: true` and `ANTHROPIC_API_KEY` in `.env`:

```json
"modelRoles": {
  "default": { "provider": "lmstudio", "model": "qwen/qwen3.6-35b-a3b" },
  "fast": { "provider": "lmstudio", "model": "" },
  "preflight": { "provider": "lmstudio", "model": "" },
  "architect": { "provider": "lmstudio", "model": "" },
  "deep": { "provider": "lmstudio", "model": "" },
  "critic": { "provider": "lmstudio", "model": "" },
  "repair": { "provider": "anthropic", "model": "claude-opus-5-5" }
}
```

`codelead config` prints it so:

```output
Model Roles
  default: lmstudio / qwen/qwen3.6-35b-a3b
  fast: lmstudio / (provider default)
  preflight: lmstudio / (provider default)
  architect: lmstudio / (provider default)
  deep: lmstudio / (provider default)
  critic: lmstudio / (provider default)
  repair: anthropic / claude-opus-5-5
```

See [The repair role](../concepts/models-and-roles.md#the-repair-role) and [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md).

## strategyModelRoles

Which role each [strategy](../concepts/strategies.md) uses. Each value is one of the seven roles.

| Strategy | Default role |
|---|---|
| `fast` | `fast` |
| `clarify` | `preflight` |
| `architect` | `architect` |
| `deep` | `deep` |

## strategySettings

One entry for each strategy, with these keys:

| Key | Type | What it does |
|---|---|---|
| `temperature` | number from 0 to 2 | The sampling temperature of the strategy's calls. |
| `maxTokens` | whole number | The reply cap of the strategy's calls. |
| `askQuestions` | boolean | Whether the preflight asks clarification questions. |
| `maxQuestions` | whole number | The most questions it asks. Optional. |
| `includeImplementationPlan` | boolean | Whether the preflight writes an implementation plan. |
| `includeRiskAnalysis` | boolean | Whether it writes a risk analysis. |
| `includeValidationPlan` | boolean | Whether it writes a validation plan. |
| `includeNonGoals` | boolean | Whether it lists what the change must not do. |

The defaults:

| Strategy | `temperature` | `maxTokens` | `askQuestions` | `maxQuestions` | Plan | Risks | Validation plan | Non-goals |
|---|---|---|---|---|---|---|---|---|
| `fast` | 0.1 | 3000 | `false` | none | `false` | `false` | `false` | `false` |
| `clarify` | 0.2 | 6000 | `true` | 5 | `false` | `false` | `false` | `false` |
| `architect` | 0.2 | 12000 | `true` | 5 | `true` | `false` | `true` | `true` |
| `deep` | 0.2 | 16000 | `true` | 7 | `true` | `true` | `true` | `true` |

## context

How much of the repository a request carries to the model. [`codelead preflight`](../cli/codelead-preflight.md), [`codelead scan`](../cli/codelead-scan.md), the chat, [`/ask`](../chat-commands/ask.md) and [`/lint`](../chat-commands/lint.md) read it.

| Key | Type | Default | What it does |
|---|---|---|---|
| `maxFiles` | whole number above 0 | 12 | The most files in one request's context. `--max-files` of a command overrides it. |
| `maxSnippetCharsPerFile` | whole number above 0 | 4000 | The most characters of one file in the context. |
| `includeTests` | boolean | `true` | Whether test files can be in the context. |
| `includePackageJson` | boolean | `true` | Whether `package.json` can be in the context. |

## privacy

See [Privacy and remote models](../concepts/privacy-and-remote-models.md).

| Key | Type | Default | What it does |
|---|---|---|---|
| `allowRemoteModels` | boolean | `false` | `false` refuses every call to a provider with `isRemote: true`, with the words `Remote model blocked by privacy settings.` `CODELEAD_ALLOW_REMOTE_MODELS=1` sets it to `true` for one run. |
| `confirmBeforeSendingRepoContextToRemote` | boolean | `true` | Asks before the first call that sends the repository's code to a remote provider. `CODELEAD_ALLOW_REMOTE_MODELS=1` turns it off. |
| `redactSensitiveFiles` | boolean | `true` | Not used. A file that sets it still loads, and `codelead config` prints it with `(not used: …)`. Whatever its value, CodeLead's ignore rules keep sensitive files, such as `.env`, out of its scans, the chat's sources and the patch prompt. The [patch check](../concepts/the-patch-check.md) refuses a patch that names one. Nothing inside a file that is sent is redacted. The extension's Settings page no longer shows it, from version 0.7.16, and its **Save** leaves the key as it is. |

## output

| Key | Type | Default | What it does |
|---|---|---|---|
| `saveSessions` | boolean | `true` | Whether [`codelead preflight`](../cli/codelead-preflight.md) saves its analysis under `.codelead/sessions/`. The session logs of the chat do not depend on it. |

## incrementPlanner

Optional. Read when a new app is planned as increments ([`/build-incremental`](../chat-commands/build-incremental.md)).

| Key | Type | Default | What it does |
|---|---|---|---|
| `temperature` | number from 0 to 2 | 0.2 | The temperature of the call that cuts the request into increments. `CODELEAD_INCREMENT_PLANNER_TEMP` wins over it. |

## callBudgets

Optional. How long one model call may run and think. The provider cuts a call that goes past its budget, and CodeLead asks once more with less thinking.

| Key | Type | Default | What it does |
|---|---|---|---|
| `profile` | `fast`, `balanced` or `quality` | `balanced` | The table of budgets below. `CODELEAD_BUDGET_PROFILE` wins over it. |
| `perRole.<role>.maxSeconds` | number above 0 | the profile's | The longest one call may run. |
| `perRole.<role>.maxThinkingTokens` | number above 0 | the profile's | The most thinking tokens of one call. |
| `perRole.<role>.softNotifySeconds` | number above 0 | the profile's | When the session says that a call takes long. |

The `balanced` profile, in seconds and thinking tokens:

| Role | `maxSeconds` | `maxThinkingTokens` | `softNotifySeconds` |
|---|---|---|---|
| `default` | 480 | 8000 | 300 |
| `fast` | 240 | 4000 | 120 |
| `preflight` | 360 | 6000 | 240 |
| `architect` | 900 | 10000 | 480 |
| `deep` | 480 | 8000 | 300 |
| `critic` | 900 | 10000 | 480 |
| `repair` | 900 | 10000 | 480 |

A patch call on the `architect` role always gets at least the `quality` profile's thinking cap, 16,000 tokens, unless `perRole.architect.maxThinkingTokens` is set.

## greenfield

Optional. The settings of new apps. See [New apps](../concepts/new-apps.md).

| Key | Type | Default | What it does |
|---|---|---|---|
| `polishOnRepairRole` | boolean | `false` | The final polish step of a new app runs on the repair role's model. It works only when the `repair` role is set. `CODELEAD_POLISH_ON_REPAIR` wins over it. |
| `reviewPassAfterBuild` | boolean | `false` | After the planned work, one more step reads the app for code quality and usability, on the repair role's model. It works only when the `repair` role is set. `CODELEAD_REVIEW_PASS_AFTER_BUILD` wins over it. |
| `templates` | object | none | Named templates for [`/create-app --template`](../chat-commands/create-app.md) and [`codelead rebuild init --template`](../cli/codelead-rebuild-init.md). |

Each entry of `templates` has these keys:

| Key | Type | What it does |
|---|---|---|
| `source` | string, required | An `npm create` package (`vite`, `@acme/create-app@2`), a folder on this machine, or a git address. |
| `args` | list of strings | For a package only: the arguments that go after `--`. |
| `note` | string | A note for people. CodeLead does not read it. |

Example: two named templates and the review pass:

```json
"greenfield": {
  "reviewPassAfterBuild": true,
  "templates": {
    "store-ui": { "source": "../codelead-templates/vite-react-shadcn", "note": "The team's React starter" },
    "vite-vue": { "source": "vite", "args": ["--template", "vue-ts"] }
  }
}
```

Then `/create-app --template store-ui …` starts from that folder. See [Template manifest](template-manifest.md).

## modernization

Optional. Read in the old application's folder by the [modernization program](../modernization/the-program.md) and by [`/modernize`](../chat-commands/modernize.md).

| Key | Type | Default | What it does |
|---|---|---|---|
| `source` | object | none | How CodeLead starts the old application. [`codelead modernize source`](../cli/codelead-modernize-source.md) writes it. See [Run profile](run-profile.md#the-old-applications-run-profile). |
| `pins.unitsPerIncrement` | whole number above 0 | 6 | The units that one pin request names when `--units` does not. |
| `pins.patchRole` | role | see below | The role whose model writes the pins' patch and its retries. |
| `pins.maxIncrementsPerRun` | whole number, 1 to 50 | 6 | How many increments one `/modernize --pin` runs before it stops and says the next. |
| `pins.maxIncrementsPerStage` | whole number, 1 to 500 | 24 | How many increments the program's Pin stage runs by itself, unattended, before it waits. |
| `repair.rounds` | whole number, 0 to 10 | 2 | The repair rounds that a pin or an in-place transform runs by itself after its gate refused the code. It counts the rounds that did not move the failure. A round whose failure differs from every earlier one earns one more, up to twice this number. `0` runs none: [`/repair`](../chat-commands/repair.md) stays yours. |

Without `pins.patchRole`, the pins' patch goes to the repair role when it is on a remote provider and remote models are allowed. Otherwise it goes to the patch strategy's role, as for any patch.

A `source` block with a problem never makes the configuration unreadable. CodeLead says every problem of it when a stage needs it. Example of the whole section:

```json
"modernization": {
  "source": {
    "command": "dotnet",
    "args": ["run", "--project", "WebApplication1/webapp.csproj", "--urls", "http://localhost:5080"],
    "url": "http://localhost:5080/",
    "allowed": true
  },
  "pins": { "unitsPerIncrement": 4, "maxIncrementsPerRun": 3 },
  "repair": { "rounds": 1 }
}
```

## The codelead config command

[`codelead config`](../cli/codelead-config.md) prints the resolved configuration. It reads the file and the environment, and it writes nothing. After a value that an environment variable set, the line names the variable and what the file says:

```console
CODELEAD_BASE_URL=http://127.0.0.1:9/v1 codelead config
```

```output
Providers
  lmstudio
    Type: openai-compatible
    Base URL: http://127.0.0.1:9/v1  (from CODELEAD_BASE_URL; the file says http://localhost:1234/v1)
    API key env: CODELEAD_LMSTUDIO_API_KEY
    Default API key: (set)
    Remote: false
…
```

`--json` prints the configuration and `sources`. For each role and each provider, `sources` says where the provider, the model, the address and the key came from. A source is `file`, `default`, `env:<VARIABLE>` or `ladder`. A key is never printed. `defaultApiKey` reads `(set)`, and a key's source says only `present`:

```json
"default": {
  "provider": { "source": "file", "value": "lmstudio" },
  "model": { "source": "env:CODELEAD_MODEL", "value": "qwen/qwen3.6-35b-a3b", "configured": "" },
  "baseUrl": { "source": "file", "value": "http://localhost:1234/v1" },
  "apiKey": { "source": "file", "present": true }
}
```

`CODELEAD_MODEL` sets the model of every role. When a role's provider differs from the default role's, CodeLead refuses the configuration and ends with exit code 2:

```output
CODELEAD_MODEL names qwen/qwen3.6-35b-a3b, but the repair role's provider is anthropic (the default role's is lmstudio): a generic model variable does not apply across providers. Unset CODELEAD_MODEL, or set CODELEAD_MODEL_REPAIR for that role, or remove the role from the file.
```

To change the file, edit it, use the extension's [Settings](../extension/settings.md) page, or run `codelead init --force` to write the defaults again.

## See also

- [`codelead config`](../cli/codelead-config.md) and [`codelead init`](../cli/codelead-init.md)
- [Environment variables](environment-variables.md)
- [Models and roles](../concepts/models-and-roles.md)
- [Strategies](../concepts/strategies.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
- [Settings](../extension/settings.md) in the extension
- [Files and folders](files-and-folders.md)
