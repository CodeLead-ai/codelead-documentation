---
title: /implement
description: Make one change to existing code in one command, from the request to the evidence record, and read how the run ended.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /implement

**Applies to:** CodeLead 0.1 and later

## Name

`/implement` - Plans a change, writes and checks its patch, applies it, validates it and records the evidence.

## Synopsis

```console
/implement <REQUEST>
/implement --from-clipboard
/implement --force <REQUEST>
/implement --lint <REQUEST>
/implement --card <ID>
/implement #<ID>
/implement --spec <FILE> <REQUEST>
/implement --spec <FILE> --card <ID>
```

## Description

`/implement` is the main command for a change to existing code. It runs the whole path in one command and stops at each step that changes something, to ask you. Every run ends with an evidence record, whether the change landed or stopped. An evidence record is a file that says what was asked, what changed and what the checks said. See [Evidence records](../concepts/evidence-records.md).

The steps:

1. **Preflight.** A model reads the request and the code that it is about, and writes a plan. It can ask you clarifying questions.
2. **Patch.** A model writes the patch from the plan.
3. **Patch check.** CodeLead checks the patch against the files, with no model. See [The patch check](../concepts/the-patch-check.md).
4. **Retries.** When the check finds a problem that a new patch can fix, CodeLead asks for a new patch, at most 2 times.
5. **Apply.** CodeLead asks `Apply this patch?`. Only a yes changes files.
6. **Validation.** CodeLead asks `Run validation now?`, then runs the gate (the project's validation commands). A pass is a [landing](../concepts/glossary.md): the change is done.

After a landing, CodeLead can make a checkpoint commit when the configuration asks for one. With `--spec`, it also walks the spec on the running application.

When the request asks to fix a bug and the preflight does not find its cause, the run ends after the preflight. CodeLead makes no patch, and says what it could not find. Name the cause in the request, or use `--force`.

> [!NOTE]
> **New to this?** Write the request for a colleague. Say what should change, where, and how you will know it works. For example: `add a clearCompleted function to src/tasks.js that removes the done tasks, with a test`. [`/lint`](lint.md) checks a request before you run it.

> [!TIP]
> **For experienced users:** options count only before the request. In `/implement make the deploy script stop using git push --force`, the words `--force` are part of the request. The preflight's depth follows [`/strategy`](strategy.md). When the words are the request of a new app whose build is in progress, `/implement` continues that build.

## Arguments

- **`<REQUEST>`**

  The change, in plain words.

- **`#<ID>`**

  A board card's number, for example `#42`. The same as `--card 42`.

## Options

- **`--from-clipboard`**

  Takes the request from the clipboard text.

- **`--force`**

  Makes the patch even when the preflight says that it has not found the cause.

- **`--lint`**

  Runs [`/lint`](lint.md) on the request first, and keeps the lint on the evidence record. A `/lint` that you run just before `/implement` is kept in the same way.

- **`--card <ID>`**

  Takes the request from a card of the board (`docs/codelead/board`). CodeLead writes `Started`, then `Landed` or `Stopped`, on the card as the participant `codelead`. A card and request text together are refused, and so are a card and the clipboard. See [Pinboard](../pinboard/index.md).

- **`--spec <FILE>`**

  A spec to walk on the running application after the landing, for example `docs/codelead/specs/sign-in.md`. See [Specification files](../reference/spec-files.md). You can give it more than once. CodeLead starts the application from `.codelead/run.json`, or from the rebuild's run block. With `--card`, this option wins over the card's spec attachments. A spec alone is refused: it needs a request or a card.

## What CodeLead asks

| Question | CodeLead recommends | Enter gives |
|---|---|---|
| The preflight's clarifying questions, each ending `Answer, press Enter to use default:` | The default that the question shows. | The default. |
| `Apply this patch? [Y/n]` | Yes, when the check passed with no blocking finding. | Yes. |
| `Run validation now? [Y/n]` | Yes. | Yes. |
| `Choose validation scope [1-3], or press Enter for 1:` | 1, the widest scope. | 1. |
| `Continue?` before code goes to a remote model, when your privacy settings ask | Nothing: the step is hard to undo. | Nothing: type `y` or `n`. |

## What you see

| Outcome | Headline | The way on |
|---|---|---|
| `passed` | `The change landed: applied to 2 files and validation passed` | None. |
| `failed` | `The change was applied, but validation failed: npm test exited 1` | `/repair`, `Or fix it by hand, then /validate` |
| `done` | `The change was applied (2 files); not validated yet` | `/validate`, `/validate --override` |
| `stopped` | `The change was not applied: the answer was no; nothing is changed` | `/edit`, `/patch --show` |
| `stopped` | `The run ended after the preflight: the cause is not found, so no patch was made` | `Name the cause with its file and line in the request, and run /implement again` |
| `stopped` | `The patch passed its check and was not applied` | `/edit` |
| `failed` | `The change did not land: unable to complete preflight` | `Review with /status`, `Adjust the request and run /implement again` |
| `failed` | `The change did not land: the patch check refused the patch` | The stop report's lines, such as `Review with /patch --show` |
| `failed` | `The change did not land: the patch did not apply: <the reason>` | `/patch --check`, `/patch --retry` |

The facts name the applied files, the validation, and the evidence record: `Applied: src/tasks.js, test/tasks.test.js`, `Validation: 2 commands passed`, `Evidence record: ev-20261120043124-90r192`. When the model's reply was cut at its cap, the facts give each attempt with its counts.

## Examples

- Make a change, and accept each recommended answer with Enter:

  ```console
  /implement add a clearCompleted function to src/tasks.js that removes the done tasks, with a test
  ```

  <!-- described, not captured -->
  ```output
  Implement
  Request: add a clearCompleted function to src/tasks.js that removes the done tasks, with a test
  Step 1/6: Running preflight...
  Step 1/6: Preflight complete.
  Step 2/6: Generating patch from preflight...
  Step 2/6: Patch generated.
  Step 3/6: Checking patch...
  Step 5/6: Patch check passed.
  Patch check passed.
  Apply checked patch
  …
  The patch check passed. CodeLead recommends applying.

  Apply this patch? [Y/n]
  Patch applied successfully.
  …
  Run validation now? [Y/n]
  …
  Implementation complete.
  Validation: automated passed.
  Implementation summary

  Patch: applied
  Patch check: passed
  Retries used: 0
  Validation: automated passed

  Done.
  Evidence record: <your project>/.codelead/evidence/ev-….json
  ✔ The change landed: applied to 2 files and validation passed
  ```

- The model server does not run. The run stops after step 1, and it still writes an evidence record:

  ```output
  Implement
  Request: add a clearCompleted function that removes the done tasks
  Step 1/6: Running preflight...
  Preflight
  Generating an implementation-ready coding-agent prompt.
  Failed to run preflight
  Could not connect to lmstudio at http://localhost:1234/v1.
  If this is LM Studio, make sure it is open and the Local Server is running.
  Implementation stopped.

  Reason:
  Unable to complete preflight.

  Next:
  - Review with /status
  - Adjust the request and run /implement again
  Implementation summary

  Patch: not applied
  Patch check: not run
  Retries used: 0
  Validation: not run
  Evidence record: <your project>/.codelead/evidence/ev-20261120043124-90r192.json
  ✖ The change did not land: unable to complete preflight; next: Review with /status
  ```

  Start the model server, check it with [`codelead doctor`](../cli/codelead-doctor.md), then run the same `/implement` again.

- Implement a board card by its number, in a repository without a board:

  ```console
  /implement #42
  ```

  ```output
  No board in this repository (docs/codelead/board); /implement <request> works without one.
  ```

- A card and request text together are refused, before any model call:

  ```console
  /implement --card 42 add a button
  ```

  ```output
  /implement --card 42 takes its request from the card: write the request on the card, or run /implement <request> without --card.
  ```

- A spec without a request is refused:

  ```console
  /implement --spec docs/codelead/specs/tasks.md
  ```

  ```output
  /implement --spec names the spec walked after the landing, not the request: give the request too, as in /implement --spec docs/codelead/specs/sign-in.md <request>, or a card (--card <id>).
  ```

## Files

- Writes one evidence record per run to `.codelead/evidence/<id>.json`, landed or stopped.
- Writes the task's state to `.codeleadsessions/<session>/implement/state.json` at every step, so that [`/session --resume`](session.md) can continue it.
- Writes the model's prompts and answers to the session folder.
- With `--card`, writes comments on the card's file under `docs/codelead/board/`.
- With `--spec`, reads the spec file and `.codelead/run.json`.

## See also

- [`/lint`](lint.md) and [`/preflight`](preflight.md)
- [`/patch`](patch.md), [`/edit`](edit.md) and [`/validate`](validate.md)
- [`/repair`](repair.md)
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
- [Requests, plans and patches](../concepts/requests-plans-and-patches.md)
- [Evidence records](../concepts/evidence-records.md)
- [`codelead implement`](../cli/codelead-implement.md)
