---
title: /strategy
description: Show or set how much planning the preflight does in this session, from a fast prompt to a deep analysis.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /strategy

**Applies to:** CodeLead 0.1 and later

## Name

`/strategy` - Shows or sets the preflight strategy of this session.

## Synopsis

```console
/strategy
/strategy --set auto
/strategy --set fast
/strategy --set clarify
/strategy --set architect
/strategy --set deep
```

## Description

A strategy decides how much the preflight of [`/preflight`](preflight.md) and [`/implement`](implement.md) does. It sets how many questions the preflight can ask, what it plans, and which model role it uses. See [Strategies](../concepts/strategies.md).

`/strategy` shows the strategy of this session. `/strategy --set <STRATEGY>` changes it until the session ends. A new session starts with `auto`.

| Strategy | Questions | Plans | Model role |
|---|---|---|---|
| `auto` | As the chosen strategy. | CodeLead picks one of the four below from the words of the request. | As the chosen strategy. |
| `fast` | None. | The prompt only. | `fast` |
| `clarify` | Up to 5. | The prompt only. | `preflight` |
| `architect` | Up to 5. | The implementation, the validation and the non-goals. | `architect` |
| `deep` | Up to 7. | The implementation, the risks, the validation and the non-goals. | `deep` |

With `auto`, words such as "security", "payment" or "migration" choose `deep`. Words such as "endpoint", "refactor" or "test" choose `architect`. Words such as "add" or "dashboard" choose `clarify`, and words such as "rename" or "label" choose `fast`.

> [!TIP]
> **For experienced users:** each strategy's settings are in `strategySettings` of `.codelead/config.json`: `maxTokens`, `maxQuestions`, `askQuestions` and the plan parts. The role of each strategy is in `strategyModelRoles`. See [Configuration file](../reference/configuration.md).

## Options

- **`--set <STRATEGY>`**

  Sets the strategy: `auto`, `fast`, `clarify`, `architect` or `deep`. The default is `auto`.

The form without `--set`, such as `/strategy deep`, is refused with `Use: /strategy --set deep`.

## What you see

| Outcome | Headline |
|---|---|
| `done` | `Current strategy: auto` |
| `done` | `✓ Strategy set to deep.` |
| `failed` | `Strategy not changed: slow is no strategy (auto, fast, clarify, architect or deep); it stays auto`, after the list of supported strategies |
| `failed` | `Missing value for --set` |

## Examples

- Show the strategy of this session:

  ```console
  /strategy
  ```

  ```output
  Current strategy: auto
  ```

- Ask for a deep analysis before a risky change:

  ```console
  /strategy --set deep
  ```

  ```output
  ✓ Strategy set to deep.
  ```

- A strategy that does not exist:

  ```console
  /strategy --set slow
  ```

  ```output
  Unknown strategy: slow

  Supported strategies:
  - auto
  - fast
  - clarify
  - architect
  - deep
  ✖ Strategy not changed: slow is no strategy (auto, fast, clarify, architect or deep); it stays auto
  ```

- The form without `--set`:

  ```console
  /strategy deep
  ```

  ```output
  Unsupported command form: /strategy deep

  Use:
    /strategy --set deep
  ```

## Files

- Reads `strategySettings` and `strategyModelRoles` from `.codelead/config.json`. It writes no file: the strategy lasts for this session.

## See also

- [`/preflight`](preflight.md)
- [`/implement`](implement.md)
- [Strategies](../concepts/strategies.md)
- [Models and roles](../concepts/models-and-roles.md)
- [Configuration file](../reference/configuration.md)
