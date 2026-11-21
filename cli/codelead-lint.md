---
title: codelead lint
description: Check a request that a person wrote, before a run, with no model call.
kind: reference
area: cli
date: 2026-11-20
---

# codelead lint

**Applies to:** CodeLead 0.1 and later

`codelead lint` checks a written request against the repository before a run. It says which files the request names, which tests are near them, and what the request should also say. It calls no model.

## Name

`codelead lint` - Lints a request written by a person before a run.

## Synopsis

```console
codelead lint --request <FILE> [--root <DIR>] [--json]
codelead lint -h|--help
```

## Description

A request that names the wrong file, or assumes a library that the project does not have, leads to a wrong change. `codelead lint` finds these problems before any model is called. It reads the request in the file, the repository and, when they are there, the [System Model](../modernization/system-model.md) and the journey spec. Then it prints:

- **Files the request names**: each one, and whether it exists. For a name that does not exist, the nearest existing name.
- **Test files**: the test files that the request names, and those beside the files it names, with their test counts.
- **Libraries it assumes**: present or absent in the project. For .NET, the lint knows common libraries by name, such as FluentValidation or NLog, and reads a `using` line that the request quotes. It looks for them in the project files, `packages.config` and `Directory.Packages.props`.
- **Model nodes it touches**: the parts of the System Model that the request reaches, in a modernization.
- **Spec steps that express its ask**: the journey steps that would show that the change works, or that none does.
- **Lines the request should carry**: the standing rules to add to the request.

The lint is advisory: it never blocks a run. In a session, [`/implement --lint`](../chat-commands/implement.md) runs it first and keeps it on the [evidence record](../concepts/evidence-records.md). [`/lint`](../chat-commands/lint.md) runs it in a session.

> [!NOTE]
> **New to this?** *Lint* means a quick check of a text for common mistakes. Here the text is your request.

> [!TIP]
> **For experienced users:** the System Model is read from `docs/codelead/rebuild/system-model.json` and the journeys from `docs/codelead/rebuild/journeys.json`. Without them, those two sections say so.

## Options

- **`--request <FILE>`**

  Required. The file that holds the request.

- **`--root <DIR>`**

  The repository the request is for. The default is the current folder.

- **`--json`**

  Prints the lint as JSON. The default is the report in words.

## Examples

- Lint a request for the Basic Store sample:

  ```console
  codelead lint --request request.md
  ```

  The file `request.md` holds: "Add a Discount field to Product in WebApplication1/Models/Product.cs and show it on the Products index page."

  ```output
  Request lint (the request, the repository, the System Model and the journey spec; no model call)

  Files the request names:
  - WebApplication1/Models/Product.cs: exists

  Test files it names, and those beside the files it names:
  - none

  Libraries it assumes:
  - none named

  Model nodes it touches: no System Model (docs/codelead/rebuild/system-model.json).

  Spec steps that express its ask: no journey spec (docs/codelead/rebuild/journeys.json); the ask can be judged by observation only.

  Lines the request should carry:
  - none
  ```

- Lint a request with two problems: a file name that is wrong, and a library that the project does not have. This repository has a System Model, so the lint also reads it:

  ```console
  codelead lint --request request2.md
  ```

  The file `request2.md` holds: "Fix the total in WebApplication1/Models/Orders.cs using lodash."

  ```output
  Request lint (the request, the repository, the System Model and the journey spec; no model call)

  Files the request names:
  - WebApplication1/Models/Orders.cs: not found (nearest: WebApplication1/Models/Order.cs)

  Test files it names, and those beside the files it names:
  - none

  Libraries it assumes:
  - lodash: absent (no manifest of the project declares it)

  Model nodes it touches (docs/codelead/rebuild/system-model.json):
  - none: the request names no path, symbol, route or label of the model

  Spec steps that express its ask: no journey spec (docs/codelead/rebuild/journeys.json); the ask can be judged by observation only.

  Lines the request should carry:
  - none
  ```

- Lint from the folder above the repository. `--root` names the repository:

  ```console
  codelead lint --request basic-store/request.md --root basic-store
  ```

  ```output
  Request lint (the request, the repository, the System Model and the journey spec; no model call)

  Files the request names:
  - WebApplication1/Models/Product.cs: exists
  …
  ```

- Name a request file that is not there. CodeLead stops with exit code 2:

  ```console
  codelead lint --request missing.md
  ```

  ```output
  The request cannot be read from <your project>/missing.md: ENOENT: no such file or directory, open '<your project>/missing.md'
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The lint ran, whatever it found. |
| `2` | `--request` was not given, the request cannot be read, the file is empty, or the root is not a folder. |

## Files

`codelead lint` writes nothing. It reads the request file, the repository, `docs/codelead/rebuild/system-model.json` and `docs/codelead/rebuild/journeys.json`.

## See also

- [`/lint`](../chat-commands/lint.md) and [`/implement`](../chat-commands/implement.md): the lint inside a session.
- [`codelead preflight`](codelead-preflight.md): the next step, with a model.
- [`codelead scan`](codelead-scan.md): the files that a request is about.
- [Requests, plans and patches](../concepts/requests-plans-and-patches.md)
- [The System Model](../modernization/system-model.md)
