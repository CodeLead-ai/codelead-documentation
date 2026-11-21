---
title: codelead models
description: List the models that a configured provider offers, to choose a model for a role.
kind: reference
area: cli
date: 2026-11-20
---

# codelead models

**Applies to:** CodeLead 0.1 and later

`codelead models` asks a provider for the models it offers and prints one model id on each line. Use an id from this list in `.codelead/config.json` or in `CODELEAD_MODEL`.

## Name

`codelead models` - Lists the models of the configured provider.

## Synopsis

```console
codelead models [--provider <PROVIDER>]
codelead models -h|--help
```

## Description

By default, `codelead models` asks the provider of the default role. For an OpenAI-compatible server (LM Studio, OpenRouter, a local server), it calls the server's model list. This is not a model call: no model answers a prompt, and nothing is billed.

> [!NOTE]
> **New to this?** A model id is the exact name that a provider gives a model, for example `qwen/qwen3.6-35b-a3b` or `claude-sonnet-5-5`. CodeLead needs the exact id. Copy it from this list.

### An Anthropic provider without a key

When the provider is Anthropic and its key is not set, CodeLead cannot ask the API. It then prints the models that it knows, and a note on the error stream that says why. The Visual Studio Code extension uses this list for its model menu.

> [!TIP]
> **For experienced users:** the model ids go to standard output, one `- <id>` line each. Notes and the progress line go to standard error. A script can read standard output alone.

## Options

- **`--provider <PROVIDER>`**

  The provider to list models from: a name under `providers` in `.codelead/config.json`, such as `lmstudio`, `openrouter` or `anthropic`. The default is the default role's provider.

## Examples

- List the models of the default provider, a local LM Studio server. The list shows the models that the server offers:

  ```console
  codelead models
  ```

  <!-- described, not captured -->
  ```output
  - qwen/qwen3.6-35b-a3b
  - google/gemma-4-e4b
  ```

- List the models of Anthropic, when `ANTHROPIC_API_KEY` is not set:

  ```console
  codelead models --provider anthropic
  ```

  ```output
  The known Anthropic models: the API could not be asked (Missing API key for provider "anthropic".).
  - claude-sonnet-5-5
  - claude-opus-5-5
  - claude-haiku-5-5
  - claude-fable-5-1
  ```

- See what happens when no server answers at the configured address:

  ```console
  CODELEAD_BASE_URL=http://127.0.0.1:9/v1 codelead models
  ```

  ```output
  Error:
  Could not connect to lmstudio at http://127.0.0.1:9/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ```

  Start the server, or correct the base URL. See [Set up local models](../how-to/set-up-local-models.md).

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The list was printed, the known Anthropic models included. |
| `1` | The provider could not be reached, or the provider name is not in the configuration. |

## Files

`codelead models` writes nothing. It reads `.codelead/config.json`, `.env` and the environment.

## See also

- [`codelead config`](codelead-config.md): see which provider and model each role uses.
- [`codelead doctor`](codelead-doctor.md): check that the model server answers.
- [Models and roles](../concepts/models-and-roles.md)
- [Set up local models](../how-to/set-up-local-models.md)
- [Set up remote models](../how-to/set-up-remote-models.md)
