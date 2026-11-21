---
title: Models and roles
description: Learn which model does each kind of work in CodeLead, how providers and roles fit together, and how the repair role gives repairs a stronger model.
kind: concept
area: concepts
date: 2026-11-20
---

# Models and roles

A model is the program that writes the plans and the code: a large language model. CodeLead contains no model. It sends each piece of work to a model server, and a **role** decides which model gets which piece. This page explains providers, roles and the repair role.

**In short**
- A **provider** is a model server that CodeLead can call. It is local (on your machine or your network) or remote (on the internet).
- A **role** names a kind of work, such as `preflight` (the analysis before a change) or `repair`. Each role points to a provider and a model.
- The **repair role** lets a stronger model fix what the gate (your project's build and tests) refuses. A local model does the rest.
- [`codelead config`](../cli/codelead-config.md) prints each role's provider and model, and where each value came from.

## Why it exists

Different work needs different strength. A rename needs little thought. A change to payment code needs a careful plan. A local model keeps your code on your machine and costs nothing per call, but it is often weaker. A remote model is often stronger, but your code leaves the machine for each call. Roles let you choose, for each kind of work, in one file.

CodeLead treats every model as a worker that can be replaced. The [patch check](the-patch-check.md) and [the gate](validation-and-the-gate.md) judge what a model writes, whatever the model is.

## How it works

### Providers

A provider is an entry under `providers` in `.codelead/config.json`. [`codelead init`](../cli/codelead-init.md) writes four:

| Provider | Kind | Address | Remote |
|---|---|---|---|
| `lmstudio` | OpenAI-compatible | `http://localhost:1234/v1`, a local server such as LM Studio | No |
| `openrouter` | OpenAI-compatible | `https://openrouter.ai/api/v1` | Yes |
| `anthropic` | The Anthropic API | The SDK's default address | Yes |
| `repair-local` | OpenAI-compatible | `http://localhost:8080/v1`, a second local server | No |

The `isRemote` field of a provider decides the privacy rules. CodeLead sends no code to a remote provider without your consent. See [Privacy and remote models](privacy-and-remote-models.md).

### Roles

A role is an entry under `modelRoles`: a provider and a model. An empty model means "the model that the server has loaded".

| Role | The work it does |
|---|---|
| `default` | Any call whose own role has no entry. |
| `fast`, `preflight`, `architect`, `deep` | The work of the four [strategies](strategies.md): the preflight and the patch of a change run on the role of the session's strategy. With the `auto` strategy, the patch runs on the `architect` strategy's role. |
| `critic` | The review of a new app's increment plan against the request. Without an entry, it takes the `architect` role's provider and model. |
| `repair` | Optional. The repairs, when it is set. See the next section. |

`strategyModelRoles` maps each strategy to its role. The default is `fast → fast`, `clarify → preflight`, `architect → architect` and `deep → deep`. The planners of a new app use the `architect` role, and the chat uses the `deep` role.

### The repair role

When the gate refuses a change, a [repair round](repair.md) asks a model for a fix. Without a repair role, that call goes where the patch calls go. With a repair role, it goes to the repair role's provider and model. A common setup is a local model for every role, and a stronger remote model for the repair role.

The repair role also does four other jobs when it is set:

- In a new app's build, the patch check can refuse a step's patch twice for the same problem. The step then gets one more attempt on the repair role.
- When `greenfield.polishOnRepairRole` is `true`, the final polish step or increment of a new app runs on the repair role.
- When `greenfield.reviewPassAfterBuild` is `true`, one more step reviews the app after the build, for code quality and ease of use, on the repair role.
- The [pins](../modernization/pins.md) of a modernization write their patch on the repair role when it is remote and remote models are allowed.

> [!NOTE]
> **New to this?** The Settings page of the [extension](../extension/settings.md) shows each role as a row, with its provider and its model. The repair role is the row named "repair".

### A real example

Here the file sets a remote repair role, and an environment variable changes its model. The model names are examples. `codelead config` says where each value came from:

```console
CODELEAD_MODEL_REPAIR=remote-model-xl codelead config
```

```output
…
Model Roles
  default: lmstudio / (provider default)
  fast: lmstudio / (provider default)
  preflight: lmstudio / (provider default)
  architect: lmstudio / (provider default)
  deep: lmstudio / (provider default)
  critic: lmstudio / (provider default)
  repair: anthropic / remote-model-xl  (model from CODELEAD_MODEL_REPAIR; the file says remote-model-large)
…
```

[`codelead doctor`](../cli/codelead-doctor.md) checks the same roles. Its line for this file is `Repair role  anthropic, model remote-model-large`. Without a repair role, it says `unset: the repairs go where the patch calls go`.

## What it is not

- **Not a model of its own**. CodeLead ships no model and trains none. It calls the servers that you name.
- **Not a promise that a stronger model fixes everything**. A failure of the environment, such as a missing tool, is not a model problem. CodeLead says so in the result, and offers a stronger model only when the model is the likely cause. See [Results and stops](results-and-stops.md).
- **Not a way around the checks**. A patch from the repair role goes through the same patch check, the same approval and the same gate.

## Details

- **Where the settings live**. `providers`, `modelRoles`, `strategyModelRoles`, `strategySettings` and `greenfield` in `.codelead/config.json`. See [Configuration file](../reference/configuration.md).
- **Environment variables**. They win over the file:
  - `CODELEAD_BASE_URL` sets the `lmstudio` provider's address.
  - `CODELEAD_MODEL` sets the model of every role. It is refused when a role uses another provider than the default role. See the override rule below.
  - `CODELEAD_MODEL_<ROLE>` sets one role's model, for example `CODELEAD_MODEL_REPAIR`.
  - `CODELEAD_PROVIDER_REPAIR` sets the repair role's provider. `CODELEAD_REPAIR_BASE_URL` sets the address of `repair-local`.
  - `CODELEAD_POLISH_ON_REPAIR` and `CODELEAD_REVIEW_PASS_AFTER_BUILD` (`1`, `true`, `yes` or `0`, `false`, `no`) win over the two `greenfield` switches.
- **The override rule**. A generic `CODELEAD_MODEL` that would replace the model of a role on another provider is refused before any model call. `codelead config` and [`codelead interactive`](../cli/codelead-interactive.md) then end with exit code 2. See [The override rule](../cli/codelead-config.md#the-override-rule). The words are:

  ```output
  CODELEAD_MODEL names local-model-small, but the repair role's provider is anthropic (the default role's is lmstudio): a generic model variable does not apply across providers. Unset CODELEAD_MODEL, or set CODELEAD_MODEL_REPAIR for that role, or remove the role from the file.
  ```

- **Keys**. A provider names the variable that holds its key in `apiKeyEnv`, for example `ANTHROPIC_API_KEY`. Put the key in the repository's `.env`. `codelead config` masks every key, and `--json` prints `(set)`.
- **What each call used**. The session log names the role, the provider, the model asked for and the model that answered, for every call. See [Sessions](sessions.md).
- **Reply limits**. `strategySettings.<strategy>.maxTokens` caps the length of a reply. The result names a reply that was cut at its cap. When most of that reply was the model's thinking, CodeLead tries once more and asks for less thinking.

> [!TIP]
> **For experienced users:** `codelead config --json` carries a `sources` object. For each role and provider, it says where each value came from: `file`, `default`, or `env:<NAME>`.

## Related content

- [`codelead config`](../cli/codelead-config.md), [`codelead models`](../cli/codelead-models.md) and [`codelead doctor`](../cli/codelead-doctor.md)
- [`/strategy`](../chat-commands/strategy.md) and [`/repair`](../chat-commands/repair.md)
- [Strategies](strategies.md)
- [Repair](repair.md)
- [Privacy and remote models](privacy-and-remote-models.md)
- [Set up local models](../how-to/set-up-local-models.md) and [Set up remote models](../how-to/set-up-remote-models.md)
- [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md)
- [Environment variables](../reference/environment-variables.md)
- [Glossary](glossary.md)
