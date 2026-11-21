---
title: codelead preflight
description: Analyze a coding request with a model before a coding agent works on it, and get a prompt that is ready for the agent.
kind: reference
area: cli
date: 2026-11-20
---

# codelead preflight

**Applies to:** CodeLead 0.1 and later

`codelead preflight` reads a coding request and the repository, asks a model to analyze them, and writes a prompt that a coding agent can follow. It changes no code.

## Name

`codelead preflight` - Analyzes a coding request before you hand it to an AI coding agent.

## Synopsis

```console
codelead preflight [<REQUEST>...] [--strategy <STRATEGY>] [--max-files <N>]
                   [--provider <PROVIDER>] [--model <MODEL>] [--role <ROLE>]
                   [--json] [--no-save] [--no-copy] [--no-interactive]
                   [--accept-defaults] [--from-clipboard]
codelead preflight -h|--help
```

## Description

A *preflight* is the first step of every change in CodeLead. It does four things:

1. It scans the repository for the files that the request is likely about, as [`codelead scan`](codelead-scan.md) does. It also reads the [repository map](../concepts/repository-map.md), and writes it again when it is stale.
2. It sends the request and those files to a model. The model returns a summary, the likely files, the clear requirements, the ambiguities, the assumptions and the questions.
3. It asks you the questions, one at a time, unless `--no-interactive` stops them.
4. It builds an implementation spec and a final prompt for a coding agent. It saves the session and copies the prompt to the clipboard.

In a session, [`/implement`](../chat-commands/implement.md) runs a preflight first, then the patch. `codelead preflight` runs only the first step, for a request that you give to another coding agent.

> [!NOTE]
> **New to this?** A *request* is what you want, in your own words: "add a discount field to products". The preflight turns it into a precise prompt, and asks you what is not clear. See [Requests, plans and patches](../concepts/requests-plans-and-patches.md).

### Strategies

The [strategy](../concepts/strategies.md) decides how deep the analysis goes. CodeLead picks one from the request; `--strategy` picks it for you.

| Strategy | What it adds |
|---|---|
| `fast` | A short analysis. No questions. |
| `clarify` | Up to 5 questions. |
| `architect` | Questions, an implementation plan, a validation plan and non-goals. |
| `deep` | Up to 7 questions, and a risk analysis as well. |

### Remote models

When the role's provider is remote (Anthropic, OpenRouter), the preflight sends parts of your code to it. CodeLead refuses unless `privacy.allowRemoteModels` is `true` in `.codelead/config.json`. See [Privacy and remote models](../concepts/privacy-and-remote-models.md).

### In a rebuild

In a rebuild repository (one that holds `docs/codelead/rebuild/rebuild.json`), the preflight also reads the old application, read-only. It checks what the request claims about the old code. See [The rebuild](../modernization/rebuild.md).

> [!TIP]
> **For experienced users:** the number of files is `context.maxFiles` in the configuration (12 by default), and `--max-files` overrides it. The strategies' settings are `strategySettings` in [the configuration file](../reference/configuration.md).

## Arguments

- **`<REQUEST>...`**

  The coding request, in words. Several words are joined with spaces. Without a request and without `--from-clipboard`, CodeLead asks you to type it.

## Options

- **`--strategy <STRATEGY>`**

  The preflight strategy: `fast`, `clarify`, `architect` or `deep`. The default is the one that CodeLead picks from the request.

- **`--max-files <N>`**

  The most relevant files to include. A whole number of at least 1. The default is `context.maxFiles` (12).

- **`--provider <PROVIDER>`**

  The provider to use. The default is the provider of the strategy's role.

- **`--model <MODEL>`**

  The model id to use. The default is the model of the strategy's role.

- **`--role <ROLE>`**

  The model role to use, in place of the strategy's role.

- **`--json`**

  Prints the result as JSON. Nothing is copied, and nothing else is printed.

- **`--no-save`**

  Does not save the preflight session under `.codelead/sessions/`.

- **`--no-copy`**

  Does not copy the final prompt to the clipboard.

- **`--no-interactive`**

  Does not ask the clarification questions. The final prompt then carries the open questions and safe defaults.

- **`--accept-defaults`**

  Uses the default answer for each clarification question, and records it as the answer.

- **`--from-clipboard`**

  Uses the text on the clipboard as the request.

## Examples

- Analyze a request on the Basic Store sample:

  ```console
  codelead preflight "add a discount field to products"
  ```

  <!-- described, not captured -->
  ```output
  ╭─ CodeLead Preflight ─╮
  …
  Request
  add a discount field to products

  CodeLead Context
  - Repository scanned.
  - Files scanned: 89.
  - Relevant files found: 12.
  - Session saved: .codelead/sessions/<the session>.

  LLM Analysis
  Summary
  Strategy Reasoning
  Likely Relevant Files
  Clear Requirements
  Ambiguities
  Assumptions
  Follow-Up Questions
  Suggested Validation
  …
  ╭─ Final Coding-Agent Prompt ─╮
  …
  ```

  The questions come one at a time, before the final prompt. Each one says why it matters and its default.

- Analyze a request in a script: no questions, no clipboard, JSON output:

  ```console
  codelead preflight "add a discount field to products" --no-interactive --no-copy --json
  ```

  <!-- described, not captured -->
  ```output
  {
    "summary": "…",
    "likelyRelevantFiles": [ { "path": "WebApplication1/Models/Product.cs", "reason": "…" } ],
    …
    "finalCodingAgentPrompt": "…"
  }
  ```

- See what happens when the model server does not answer. The scan and the map still run:

  ```console
  codelead preflight "add a discount field to products" --no-save --no-copy --no-interactive
  ```

  ```output
  CodeLead: Repository map was stale: the file count moved from 89 to 94, more than 2%. Rewritten: 3 modules, 94 files; docs/codelead/repository-map.md
  Error:
  Could not connect to lmstudio at http://127.0.0.1:9/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  ```

- See the refusal for a remote provider that the privacy settings block:

  ```console
  codelead preflight --provider anthropic "add a done flag to tasks"
  ```

  ```output
  Error:
  Remote model blocked by privacy settings.

  Provider:
  anthropic

  CodeLead may send repository snippets to this provider.
  To allow this, set privacy.allowRemoteModels to true in .codelead/config.json.
  ```

- Pick a strategy that does not exist:

  ```console
  codelead preflight "x" --strategy slow
  ```

  ```output
  Error:
  Unknown strategy "slow". Valid strategies: fast, clarify, architect, deep.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The analysis is done and the final prompt is printed. |
| `1` | The analysis failed: the model did not answer, its answer could not be read, a remote provider is blocked, an option is wrong. |
| `2` | The configuration was refused before the model call (the [override rule](codelead-config.md#the-override-rule)). |

## Files

| File | What `preflight` does with it |
|---|---|
| `.codelead/config.json`, `.env` | Reads the settings and the keys. |
| `.codelead/sessions/<preflight session>/` | Writes `request.txt`, `repo-context.json`, `preflight-result.json`, `implementation-spec.json`, `final-prompt.md` and the raw response. Not with `--no-save`. |
| `docs/codelead/repository-map.md`, `.json` | Reads the map for the request's context. Writes it again when it is stale. |

## See also

- [`codelead scan`](codelead-scan.md): the same scan, with no model.
- [`codelead lint`](codelead-lint.md): check a request before a run.
- [`/preflight`](../chat-commands/preflight.md) and [`/implement`](../chat-commands/implement.md): the same step inside a session.
- [Requests, plans and patches](../concepts/requests-plans-and-patches.md)
- [Strategies](../concepts/strategies.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
