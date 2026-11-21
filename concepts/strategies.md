---
title: Strategies
description: Learn how a strategy decides the depth of a change's preflight, how the auto strategy picks one from your words, and when to set one yourself.
kind: concept
area: concepts
date: 2026-11-20
---

# Strategies

A strategy decides how much thinking CodeLead asks for before it writes a patch (the change to your files). The [preflight](requests-plans-and-patches.md) is the analysis of a request before any code is written. The strategy sets how many questions the preflight can ask, which parts of a plan it writes, and which model role it uses. A session starts with the `auto` strategy, which picks one of four strategies from the words of your request.

**In short**
- There are four strategies: `fast`, `clarify`, `architect` and `deep`. `auto` picks one of them for each request.
- A deeper strategy asks more questions, plans more, and takes longer.
- [`/strategy`](../chat-commands/strategy.md) shows or sets the strategy of the session.
- Each strategy has its own settings and its own model role in `.codelead/config.json`.

## Why it exists

A small change and a risky change need different care. "Rename the label" needs no questions. "Change how payments are refunded" needs questions, a plan of the risks and a plan of the tests. One fixed depth would be too slow for the first and too thin for the second. A strategy matches the depth to the request.

## How it works

### The four strategies

| Strategy | Questions | What the preflight plans | Model role | Reply cap (`maxTokens`) |
|---|---|---|---|---|
| `fast` | None | A clear prompt for the patch, and no plan | `fast` | 3,000 tokens |
| `clarify` | Up to 5 | A clear prompt for the patch, and no plan | `preflight` | 6,000 tokens |
| `architect` | Up to 5 | The implementation, the validation and the non-goals | `architect` | 12,000 tokens |
| `deep` | Up to 7 | The implementation, the risks, the validation and the non-goals | `deep` | 16,000 tokens |

A non-goal is something the change must not do, for example "do not change the public API". The values are the defaults that [`codelead init`](../cli/codelead-init.md) writes. See [Models and roles](models-and-roles.md) for what a role is.

### How `auto` picks

`auto` reads the request's words with no model call. It checks these lists in this order, and the first match wins:

1. **`deep`** when the request touches a risky area. The words: auth, authentication, permission, security, secret, payment, billing, destructive, delete all, hard delete, data deletion, database, schema, migration, privacy, pii, large refactor, cross-cutting.
2. **`clarify`** when the request mentions a command in five words or fewer, and does not say what the command does.
3. **`architect`** when the request likely spans several code areas. The words: command, page, endpoint, api, integration, workflow, refactor, reusable, component, approve, approval, actions, test, tests, multiple files, feature.
4. **`clarify`** when the request is broad and says little. It holds one of the words add, support, implement, manage, management, moderation, dashboard or admin. It also has fewer than 12 words, or no punctuation.
5. **`fast`** when the request is small. It holds one of the words link, label, text, copy, rename, display or config, and it is shorter than 90 characters.
6. **`clarify`** in every other case.

### A real example

In a new session, the strategy is `auto`:

```console
/strategy
```

```output
Current strategy: auto
```

To ask for the deepest analysis before a risky change, set it for the rest of the session:

```console
/strategy --set deep
```

```output
✓ Strategy set to deep.
```

> [!NOTE]
> **New to this?** You do not need to set a strategy. Write a clear request, and let `auto` pick. Set one only when you want more questions, or fewer.

## What it is not

- **Not a choice of how much the checks check**. The [patch check](the-patch-check.md) and [the gate](validation-and-the-gate.md) are the same for every strategy.
- **Not saved between sessions**. `/strategy --set` lasts until the session ends. A new session starts with `auto`.
- **Not a model choice by itself**. The strategy names a role. The role names the provider and the model.

## Details

- **Settings**. `strategySettings.<strategy>` holds `temperature`, `maxTokens`, `askQuestions`, `maxQuestions`, `includeImplementationPlan`, `includeRiskAnalysis`, `includeValidationPlan` and `includeNonGoals`. `strategyModelRoles.<strategy>` names the role. See [Configuration file](../reference/configuration.md).
- **The patch**. The patch of a change runs on the role of the session's strategy. With `auto`, it runs on the `architect` strategy's role.
- **The record**. The [evidence record](evidence-records.md) keeps the strategy that the preflight really used, in `preflight.strategy`. It also keeps the number of questions, and how many took their default.
- **A composed request**. A pin or a transform of a modernization answers its own preflight questions with their defaults. CodeLead shows the list once as "CodeLead assumed", and the record keeps it.

> [!TIP]
> **For experienced users:** raise `strategySettings.<strategy>.maxTokens` when a result says that the model's reply was cut at its cap. The cap of the provider is the upper limit.

## Related content

- [`/strategy`](../chat-commands/strategy.md)
- [`/preflight`](../chat-commands/preflight.md) and [`/implement`](../chat-commands/implement.md)
- [`codelead preflight`](../cli/codelead-preflight.md) and [`codelead config`](../cli/codelead-config.md)
- [Requests, plans and patches](requests-plans-and-patches.md)
- [Models and roles](models-and-roles.md)
- [Prompts and recommended answers](prompts-and-recommended-answers.md)
- [Configuration file](../reference/configuration.md)
- [Glossary](glossary.md)
