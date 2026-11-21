---
title: /lint
description: Check a request against the repository before you run it, with no model call, and keep the lint on the next evidence record.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /lint

**Applies to:** CodeLead 0.1 and later

## Name

`/lint` - Checks a request that you wrote against the repository, with no model call.

## Synopsis

```console
/lint <REQUEST>
/lint --from-clipboard
```

## Description

`/lint` reads your request the way a reviewer would, before a model sees it. It looks for the files, tests and libraries that the request names, and says whether each one exists. It also reads the System Model and the journey spec of a rebuild, when they exist. At the end, it suggests lines that the request should carry. It calls no model, so it is fast and free.

You use `/lint` before [`/implement`](implement.md), to make a request clearer. The next `/implement` keeps the lint on its evidence record. `/implement --lint <request>` runs both in one command.

`/lint` never changes a file and never changes your request.

> [!NOTE]
> **New to this?** A request that names its files and its test is easier to do right. `/lint` shows what your request names. When it names nothing, add the file and the test that you mean.

## Arguments

- **`<REQUEST>`**

  The request to check, in plain words.

## Options

- **`--from-clipboard`**

  Takes the request from the clipboard text.

## What you see

| Outcome | Headline | The way on |
|---|---|---|
| `done` | `Request lint: 2 files and 1 test named, 0 lines to add; the next /implement keeps it` | `/implement <request>` |
| `stopped` | The command's own words, when no lint ran. | As the words say. |

The facts list up to three lines to add (`Add: …`) and up to two warnings (`Warning: …`).

## Examples

- Lint a request that names its files:

  ```console
  /lint add a clearCompleted function to src/tasks.js and a test in test/tasks.test.js
  ```

  ```output
  Request lint (the request, the repository, the System Model and the journey spec; no model call)

  Files the request names:
  - src/tasks.js: exists
  - test/tasks.test.js: exists

  Test files it names, and those beside the files it names:
  - test/tasks.test.js: 2 tests

  Libraries it assumes:
  - none named

  Model nodes it touches: no System Model (docs/codelead/rebuild/system-model.json).

  Spec steps that express its ask: no journey spec (docs/codelead/rebuild/journeys.json); the ask can be judged by observation only.

  Lines the request should carry:
  - none
  The next /implement keeps this lint on its evidence record.
  ```

  The result is `Request lint: 2 files and 1 test named, 0 lines to add; the next /implement keeps it`.

- Lint a request on the Basic Store sample that names a class but no file:

  ```console
  /lint add a discount to the order total in OrdersController
  ```

  ```output
  Request lint (the request, the repository, the System Model and the journey spec; no model call)

  Files the request names:
  - none

  Test files it names, and those beside the files it names:
  - none
  …
  The next /implement keeps this lint on its evidence record.
  ```

  The lint found no file. Add the path, for example `WebApplication1/Controllers/OrdersController.cs`, and a test.

- Lint and implement in one command:

  ```console
  /implement --lint add a clearCompleted function to src/tasks.js, with a test in test/tasks.test.js
  ```

## Files

- Reads the repository's files, `docs/codelead/rebuild/system-model.json` and `docs/codelead/rebuild/journeys.json` when they exist.
- Keeps the lint in the session for the next `/implement`. It writes no file of the project.

## See also

- [`/implement`](implement.md)
- [`/scan`](scan.md)
- [`/preflight`](preflight.md)
- [Requests, plans and patches](../concepts/requests-plans-and-patches.md)
- [`codelead lint`](../cli/codelead-lint.md)
