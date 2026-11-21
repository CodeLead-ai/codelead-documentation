---
title: codelead pr
description: Open a pull request for the last landed change, with its evidence record as the body, through the GitHub CLI.
kind: reference
area: cli
date: 2026-11-20
---

# codelead pr

**Applies to:** CodeLead 0.1 and later

`codelead pr` puts the last landed change in front of a reviewer. It makes a branch, a commit and a push, and opens a pull request whose body is the change's evidence record.

## Name

`codelead pr` - Opens a pull request for the last landed increment, with its evidence record as the body.

## Synopsis

```console
codelead pr [--base <BRANCH>] [--draft] [--title <TEXT>] [--evidence <ID>] [--dry-run]
codelead pr -h|--help
```

## Description

When a change *lands*, the patch is applied and the validation passed. CodeLead then writes an [evidence record](../concepts/evidence-records.md): the request, the files changed, the patch check, each validation command with its result, and the repair rounds. `codelead pr` turns that record into a pull request.

It needs the GitHub CLI, `gh`, installed and signed in (`gh auth login`).

> [!NOTE]
> **New to this?** A *pull request* asks a reviewer to look at a change before it joins the main branch. The body of the pull request explains the change. Here the body is the evidence, so the reviewer sees what was asked, what changed and what was checked.

### What it runs

1. `git checkout -b codelead/<name>`, when you are on the base branch. On another branch, it stays there.
2. `git add -A`. The session logs (`.codeleadsessions` and `.codelead/sessions`) and any exported session zip are taken out again: they are never committed. CodeLead knows an exported zip by its contents, whatever its name and place.
3. `git commit` with a message made from the title and the record.
4. `git push -u origin <branch>`.
5. `gh pr create --base <base> --head <branch> --title <title> --body-file <body>`.

After that, the evidence record carries the pull request's address.

### What it refuses

`codelead pr` opens a pull request only for a change that landed. It stops, and says why, when the last record stopped, did not land, ended after its preflight, or reopened a part of a rebuild. Name an earlier record with `--evidence`.

> [!TIP]
> **For experienced users:** a repair record names its change through `follows`. `codelead pr` sends the whole chain, from the implement record to the last repair, whichever record you name. The body lists at most 12 files of the change's surface.

## Options

- **`--base <BRANCH>`**

  The branch that the pull request targets. The default is the remote's default branch, else `main`.

- **`--draft`**

  Opens the pull request as a draft.

- **`--dry-run`**

  Prints the body and the steps, and runs nothing that changes state.

- **`--evidence <ID>`**

  The evidence record to send, for example `ev-20261119203740-01s4bg`. The default is the latest implement record.

- **`--title <TEXT>`**

  The pull request's title. The default is the first sentence of the request.

## Examples

- See the pull request before you open it:

  ```console
  codelead pr --dry-run
  ```

  <!-- described, not captured -->
  ```output
  Dry run — nothing was changed. This would run:
    git checkout -b codelead/add-a-discount-field-to-products-ev1
    git add -A (then .codeleadsessions and .codelead/sessions and any exported session zip unstaged, if anything there was staged)
    git commit -m "Add a discount field to products …"
    git push -u origin codelead/add-a-discount-field-to-products-ev1
    gh pr create --base main --head codelead/add-a-discount-field-to-products-ev1 --title "Add a discount field to products" --body-file <body>

  Title: Add a discount field to products

  …
  ## Evidence (CodeLead)

  - Record: `.codelead/evidence/ev-….json` — outcome **landed**
  - Surface: 3 files
    - `WebApplication1/Models/Product.cs`
  …
  - Validation: **passed**
    - `dotnet build` — passed
  ```

- Open a draft pull request against the branch `develop`:

  ```console
  codelead pr --base develop --draft
  ```

  <!-- described, not captured -->
  ```output
  Pull request opened: https://github.com/<owner>/<repository>/pull/43
  Branch codelead/add-a-discount-field-to-products-ev1 → develop.
  Evidence record ev-… now carries the pull request.
  ```

- Run it in a repository where nothing has landed yet:

  ```console
  codelead pr --dry-run
  ```

  ```output
  No increment to hand off yet: no record of an increment /implement landed. Land one with /implement first.
  ```

- When `gh` is not installed, CodeLead says how to fix it:

  <!-- described, not captured -->
  ```output
  Pull request not opened: The GitHub CLI (`gh`) is not available: install it and run `gh auth login`, then /pr again.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The pull request was opened, or the dry run was printed. |
| `1` | Nothing to send, or a step failed. The words list the steps that ran. |

## Files

| File | What `pr` does with it |
|---|---|
| `.codelead/evidence/ev-*.json` | Reads the record and its chain. After the pull request opens, the record carries its address. |
| The working tree | Commits it, without the session logs. |

## See also

- [Evidence records](../concepts/evidence-records.md)
- [Evidence record format](../reference/evidence-record-format.md)
- [`/pr`](../chat-commands/pr.md): the same pull request from inside a session.
- [`codelead implement`](codelead-implement.md): land the changes that `pr` sends.
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
