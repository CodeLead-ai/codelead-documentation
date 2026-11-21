---
title: codelead config
description: Print the configuration that CodeLead uses in this folder, and where each value that an environment variable set came from.
kind: reference
area: cli
date: 2026-11-20
---

# codelead config

**Applies to:** CodeLead 0.1 and later

`codelead config` prints the configuration that CodeLead resolves in the current folder. When an environment variable changes a value, the line says which variable, and what the file says.

## Name

`codelead config` - Prints the resolved CodeLead configuration and the source of each overridden value.

## Synopsis

```console
codelead config [--json]
codelead config -h|--help
```

## Description

CodeLead reads its settings from `.codelead/config.json` and then applies the environment variables, for example `CODELEAD_BASE_URL` or `CODELEAD_MODEL`. The result is the **resolved configuration**. `codelead config` prints it, so that you see what a session will really use.

Use it when a session talks to the wrong model or the wrong server. It writes nothing and calls no model.

> [!NOTE]
> **New to this?** A *provider* is a service that runs models (a local LM Studio server, Anthropic, OpenRouter). A *role* is a job inside CodeLead (default, preflight, architect, repair) that is given a provider and a model. See [Models and roles](../concepts/models-and-roles.md).

### What it prints

- **Providers**: each provider's type, base URL, the name of its key variable, and whether it is remote. A key is never printed: it shows as `(set)`, or as `(none)` when no key is set.
- **Model Roles**: each role's provider and model.
- **Strategy Model Roles** and **Strategy Settings**: which role each [strategy](../concepts/strategies.md) uses, with its temperature, its token limit and its questions.
- **Context**, **Output** and **Privacy**: how many files a request may carry, whether sessions are saved, and whether remote models are allowed.

After a value that an environment variable set, the line says so, for example `(from CODELEAD_BASE_URL; the file says http://localhost:1234/v1)`.

### The override rule

`CODELEAD_MODEL` sets the model of every role. When a role uses another provider than the default role, that model name cannot be right for it. CodeLead then refuses the configuration before any model call. `codelead config` still prints the configuration, then the refusal with three ways out, and ends with exit code 2.

> [!TIP]
> **For experienced users:** `--json` adds `sources`. For each role, it gives the `source` of the provider, the model, the base URL and the key: `file`, `default`, or `env:<VARIABLE>`. For a key, it gives only `present`. A refused configuration also carries `refusals`.

## Options

- **`--json`**

  Prints the configuration as JSON, with `sources`: where each role's provider, model, base URL and key came from. A key is never printed: a provider's `defaultApiKey` reads `(set)`.

## Examples

- Print the configuration of the current folder:

  ```console
  codelead config
  ```

  ```output
  Providers
    lmstudio
      Type: openai-compatible
      Base URL: http://localhost:1234/v1
      API key env: CODELEAD_LMSTUDIO_API_KEY
      Default API key: (set)
      Remote: false
    openrouter
      Type: openai-compatible
      Base URL: https://openrouter.ai/api/v1
      API key env: OPENROUTER_API_KEY
      Default API key: (none)
      Remote: true
  …
  Model Roles
    default: lmstudio / (provider default)
    fast: lmstudio / (provider default)
    preflight: lmstudio / (provider default)
  …
  Privacy
    Allow remote models: false
    Confirm before remote repo context: true
    Redact sensitive files: true (not used: whatever its value, CodeLead's ignore rules leave out of its scans, the chat's sources and the patch prompt the files they call sensitive: .env and .env.*, id_rsa, id_ed25519, *.pem, *.key, *.p12, *.pfx, *.sqlite, *.db, secrets.*, credentials.*; nothing inside a file that is sent is redacted, and what you type or paste is sent as written)
  ```

  The last line shows a switch that CodeLead does not use. The words in parentheses say what is true, whatever its value.

- See which values come from the environment. Here `CODELEAD_MODEL` names a model for every role:

  ```console
  CODELEAD_MODEL=qwen/qwen3.6-35b-a3b codelead config
  ```

  ```output
  …
  Model Roles
    default: lmstudio / qwen/qwen3.6-35b-a3b  (model from CODELEAD_MODEL; the file names none)
    fast: lmstudio / qwen/qwen3.6-35b-a3b  (model from CODELEAD_MODEL; the file names none)
    preflight: lmstudio / qwen/qwen3.6-35b-a3b  (model from CODELEAD_MODEL; the file names none)
  …
  ```

- See a refusal. The critic role uses Anthropic, and `CODELEAD_MODEL` names a local model:

  ```console
  CODELEAD_MODEL=qwen/qwen3.6-35b-a3b CODELEAD_PROVIDER_CRITIC=anthropic codelead config
  ```

  ```output
  …
    critic: anthropic / qwen/qwen3.6-35b-a3b  (provider from CODELEAD_PROVIDER_CRITIC; the file says lmstudio)  (model from CODELEAD_MODEL; the file names none)
  …
  Error:
  CODELEAD_MODEL names qwen/qwen3.6-35b-a3b, but the critic role's provider is anthropic (the default role's is lmstudio): a generic model variable does not apply across providers. Unset CODELEAD_MODEL, or set CODELEAD_MODEL_CRITIC for that role, or unset CODELEAD_PROVIDER_CRITIC.
  ```

- Read where each value came from, as JSON. Here `CODELEAD_BASE_URL` is set in the environment:

  ```console
  CODELEAD_BASE_URL=http://127.0.0.1:9/v1 codelead config --json
  ```

  ```output
  {
    "providers": { … },
    "modelRoles": { … },
  …
    "sources": {
      "file": true,
      "roles": {
        "default": {
          "provider": { "source": "file", "value": "lmstudio" },
          "model": { "source": "file", "value": "" },
          "baseUrl": {
            "source": "env:CODELEAD_BASE_URL",
            "value": "http://127.0.0.1:9/v1",
            "configured": "http://localhost:1234/v1"
          },
          "apiKey": { "source": "file", "present": true }
        },
  …
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The configuration was printed. |
| `2` | The override rule refuses the environment's model. The configuration and the refusal are printed. |
| `1` | The configuration file cannot be read. |

## Files

`codelead config` writes nothing. It reads `.codelead/config.json` and `.env` in the current folder, and the environment.

An older format of `.codelead/config.json` still works. `config` then prints a note that you can update the file.

## See also

- [Configuration file](../reference/configuration.md)
- [Environment variables](../reference/environment-variables.md)
- [Models and roles](../concepts/models-and-roles.md)
- [`codelead doctor`](codelead-doctor.md): check that the configured model answers.
- [`codelead models`](codelead-models.md): list the models of a provider.
- [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md)
