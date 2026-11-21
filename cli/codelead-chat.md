---
title: codelead chat
description: Send one message to the configured model and print its answer, to test a provider, a model or a role.
kind: reference
area: cli
date: 2026-11-20
---

# codelead chat

**Applies to:** CodeLead 0.1 and later

`codelead chat` sends one message to a model and prints the answer. It is a quick way to test that a provider, a model or a role works.

## Name

`codelead chat` - Sends a one-turn chat message to the configured model provider.

## Synopsis

```console
codelead chat [--message <TEXT>] [--provider <PROVIDER>] [--model <MODEL>] [--role <ROLE>]
codelead chat -h|--help
```

## Description

`codelead chat` sends exactly one message and prints exactly one answer. It sends no repository context: the model sees only your message. It keeps no history, and it writes no file.

Without `--message`, CodeLead asks you to type the message.

> [!IMPORTANT]
> `codelead chat` is not the chat of a session. To ask about your code, start a session with [`codelead interactive`](codelead-interactive.md), or use [the chat in Visual Studio Code](../extension/chat.md). There, text without a leading `/` goes to the chat, which reads the repository. See [`/chat`](../chat-commands/chat.md).

### Which model answers

CodeLead picks the provider and the model in this order:

1. `--provider` and `--model`, when you give them.
2. The provider and the model of `--role`, when you give it.
3. The default role of the configuration.

See [Models and roles](../concepts/models-and-roles.md). A paid provider bills the request.

> [!NOTE]
> **New to this?** A *role* is a job inside CodeLead, for example `preflight` or `repair`. Each role has a provider and a model in `.codelead/config.json`. `--role repair` tests the model that repairs use.

## Options

- **`--message <TEXT>`**

  The message to send. Without it, CodeLead asks for the message in the terminal.

- **`--provider <PROVIDER>`**

  The provider to use: a name under `providers` in `.codelead/config.json`. The default is the provider of the role.

- **`--model <MODEL>`**

  The model id to use. The default is the model of the role.

- **`--role <ROLE>`**

  The model role to use: `default`, `fast`, `preflight`, `architect`, `deep`, `critic` or `repair`. The default is `default`.

## Examples

- Send one message to the default model:

  ```console
  codelead chat --message "Reply with the single word: ready"
  ```

  <!-- described, not captured -->
  ```output
  ✔ Response received
  ready
  ```

- Test the model of the repair role:

  ```console
  codelead chat --role repair --message "Reply with the single word: ready"
  ```

  <!-- described, not captured -->
  ```output
  ✔ Response received
  ready
  ```

- See what happens when the key of a remote provider is not set:

  ```console
  codelead chat --provider anthropic --message "Say ready"
  ```

  ```output
  Error:
  Missing API key for provider "anthropic".
  Set ANTHROPIC_API_KEY or update .codelead/config.json.
  ```

- See what happens with a role that does not exist:

  ```console
  codelead chat --role nobody --message hi
  ```

  ```output
  Error:
  Unknown model role "nobody". Valid roles: default, fast, preflight, architect, deep, critic, repair.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The model answered, and the answer was printed. |
| `1` | The request failed: the server did not answer, a key is missing, a provider or a role is unknown. |
| `2` | The configuration was refused before the request (the [override rule](codelead-config.md#the-override-rule)). |

## Files

`codelead chat` writes nothing. It reads `.codelead/config.json`, `.env` and the environment.

## See also

- [`codelead models`](codelead-models.md): find a model id.
- [`codelead config`](codelead-config.md): see which model each role uses.
- [`codelead doctor`](codelead-doctor.md): `--probe-model` sends the same kind of test request.
- [`/chat`](../chat-commands/chat.md): the chat about the repository, in a session.
- [Models and roles](../concepts/models-and-roles.md)
