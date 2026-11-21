---
title: /pr
description: Hand the last change that landed to a reviewer as a branch, a commit, a push and a pull request whose body is the evidence record.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /pr

**Applies to:** CodeLead 0.1 and later

## Name

`/pr` - Opens a pull request for the last landed change, with its evidence record as the body.

## Synopsis

```console
/pr
/pr --draft
/pr --base <BRANCH>
/pr --evidence <RECORD_ID>
/pr --title <TEXT>
/pr --dry-run
```

## Description

`/pr` takes the last change that landed: the one of this session, or else the latest evidence record in `.codelead/evidence/`. It hands the change to a reviewer in five steps:

1. A branch: `git checkout -b codelead/<request>-<id>`, when you are on the base branch.
2. `git add -A`. The session logs and the exported session zips are never staged.
3. `git commit`, with a message made from the request.
4. `git push -u origin <branch>`.
5. `gh pr create`, with the evidence record as the body: the request, what changed, the gate and its verdict, the repair rounds.

After the pull request opens, the evidence record carries its address.

You use `/pr` after [`/validate`](validate.md) or [`/implement`](implement.md) ends with a landing. `/pr --dry-run` shows the steps and the body, and changes nothing.

`/pr` refuses a record whose change did not land, and says what to do: repair it first, or name an earlier record with `--evidence`.

> [!IMPORTANT]
> `/pr` needs git with a remote named `origin`, and the GitHub command-line tool `gh`, signed in with `gh auth login`.

## Options

- **`--draft`**

  Opens the pull request as a draft.

- **`--base <BRANCH>`**

  The branch that the pull request goes into. The default is the remote's default branch, or `main` when the remote names none. A value such as `../x` is refused.

- **`--evidence <RECORD_ID>`**

  Hands off this evidence record, for example `ev-20261120043124-90r192`. The default is the record of this session's last change, or the latest record.

- **`--title <TEXT>`**

  The title of the pull request. The words go up to the next option. The default is the first sentence of the request.

- **`--dry-run`**

  Prints the steps, the title and the body, and runs nothing.

## What you see

| Situation | Headline |
|---|---|
| The pull request opened | `Pull request opened: <address>`, then the branch and `Evidence record <id> now carries the pull request.` |
| A dry run | `Dry run — nothing was changed. This would run:`, then each step, the title and the body |
| Nothing landed | `No increment to hand off yet: no record of an increment /implement landed. Land one with /implement first.` |
| The change did not land | `The increment did not land (<stop reason>); nothing to hand off. …` |
| A step failed | `Pull request not opened: <the step and its error>`, then `Done so far:` and the steps that ran |
| `gh` is missing | `Pull request not opened: The GitHub CLI (gh) is not available: install it and run gh auth login, then /pr again.` |

## Examples

- See what `/pr` would do, and change nothing:

  ```console
  /pr --dry-run
  ```

  <!-- described, not captured -->
  ```output
  Dry run — nothing was changed. This would run:
    git checkout -b codelead/<request>-<id>
    git add -A (then .codeleadsessions and .codelead/sessions and any exported session zip unstaged, if anything there was staged)
    git commit -m "…"
    git push -u origin codelead/<request>-<id>
    gh pr create --base main --head codelead/<request>-<id> --title "…" --body-file <body>

  Title: …
  …
  ```

- Send the last record for review, when its change did not land:

  ```console
  /pr --dry-run
  ```

  ```output
  The increment did not land (preflight_failed); nothing to hand off. Repair it first (/repair), or hand off an earlier record with --evidence <id>.
  ```

- Run `/pr` in a project where nothing landed yet:

  ```console
  /pr
  ```

  ```output
  No increment to hand off yet: no record of an increment /implement landed. Land one with /implement first.
  ```

- Open a draft into another branch, with your own title:

  ```console
  /pr --draft --base develop --title Add clearCompleted to the task list
  ```

- A base branch that leaves the repository's names is refused:

  ```console
  /pr --base ../x
  ```

  ```output
  Invalid base branch: ../x
  ```

## Files

- Reads the evidence records in `.codelead/evidence/` and the declared gate in `.codelead/validation.json`.
- Makes a branch, a commit and a push in git, and a pull request on GitHub.
- Writes the pull request's address into the evidence record.

## See also

- [`/implement`](implement.md)
- [`/validate`](validate.md)
- [Evidence records](../concepts/evidence-records.md)
- [Evidence record format](../reference/evidence-record-format.md)
- [`codelead pr`](../cli/codelead-pr.md)
