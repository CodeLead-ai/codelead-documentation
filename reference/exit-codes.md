---
title: Exit codes
description: The exit codes of every CodeLead command family in one place - what 0, 1, 2 and 130 mean for each - and what a script or a CI job can rely on.
kind: reference
area: reference
date: 2026-11-20
---

# Exit codes

**Applies to:** CodeLead 0.1 and later

Every `codelead` command ends with an exit code. A script, a scheduled job or a CI pipeline reads it to know what happened, without parsing the words. This page gives the rule that most commands follow, the exceptions, and the codes of every command family. Each command's page has its own table too.

> [!NOTE]
> **New to this?** In a shell, `$?` holds the exit code of the last command: `codelead doctor; echo $?`. In PowerShell, it is `$LASTEXITCODE`. `0` means success.

## The rule

| Code | Meaning |
|---|---|
| `0` | The command did what it was asked. A check found nothing to fix. A program that stops at your turn is `0` too: your turn is not a failure. |
| `1` | The command ran and found a failure: a check failed, a gate did not pass, a step failed. An error that is not a refusal is `1` too, and so is a wrong option or an unknown command. |
| `2` | The command refused before it did anything: a file is missing or cannot be read, an answer is refused, the configuration is refused. Nothing was written. The words say why and what to do. |
| `130` | You pressed Ctrl+C during a long run. The step in flight finished first. |

`codelead --version` and `--help` always end with `0`.

## What a script can rely on

- **`0` and not `0`** is the one rule that holds for every command: `0` is success.
- **`2` means that nothing changed.** Every command that ends with `2` refused before it wrote a file. You can fix the input and run it again.
- **`1` can follow a change.** A gate that failed after a patch applied is `1`, and the patch is in the tree. Read the [evidence record](evidence-record-format.md) to know what changed.
- **The words go to the terminal; the code is for the script.** Never parse the words: they can change. For data, use `--json` where the command has it.
- **The configuration refusal is `2`.** A `CODELEAD_MODEL` across providers ends a command with `2`, before any model call. A session over JSON ends with `2` too, after its last line. See [the override rule](environment-variables.md#the-override-rule).
- **A long run stops between its steps.** On Ctrl+C and on a cap, `codelead implement` and `codelead rebuild run` finish the card or the part in flight first.

### The exceptions

| Command | What differs |
|---|---|
| [`codelead lint`](../cli/codelead-lint.md), [`codelead walk`](../cli/codelead-walk.md), [`codelead model`](../cli/codelead-model.md) and [`codelead modernize`](../cli/codelead-modernize.md) | A missing required option, a wrong option or an unknown word is `2`, not `1`: the command refused before anything ran. |
| [`codelead licence`](../cli/codelead-licence.md) | `0` in every state of the licence: none, valid or invalid. |
| [`codelead modernize status`](../cli/codelead-modernize-status.md) | `1` when there is no program in this folder. |
| [`codelead rebuild run`](../cli/codelead-rebuild-run.md) and [`codelead rebuild --plan`](../cli/codelead-rebuild.md) | `1`, not `2`, on a folder without the Pro plan. Every other command of the modernization program refuses it with `2`. |
| [`codelead scan`](../cli/codelead-scan.md) | `1` with `--expect-absent` when an expected-absent key is still there. |
| [`codelead lint`](../cli/codelead-lint.md) | `0` whatever the lint finds. |

## The codes of every command

### Setup and configuration

| Command | `0` | `1` | `2` |
|---|---|---|---|
| [`codelead init`](../cli/codelead-init.md) | The configuration was written. A map that could not be made is a warning. | The configuration exists and `--force` was not given, or the file could not be written. | |
| [`codelead config`](../cli/codelead-config.md) | The configuration was printed. | The configuration file cannot be read. | The override rule refuses the environment's model. |
| [`codelead doctor`](../cli/codelead-doctor.md) | Every required check passed. | A required check is `FAIL` or `missing`. | |
| [`codelead setup`](../cli/codelead-setup.md) | The steps were done, or nothing was asked, or the answer was no. | A step failed. | |
| [`codelead licence`](../cli/codelead-licence.md) | The licence was read, in any state. | The report could not be written. | |
| [`codelead models`](../cli/codelead-models.md) | The list was printed. | The provider could not be reached, or is not in the configuration. | |

### Sessions and changes

| Command | `0` | `1` | `2` | `130` |
|---|---|---|---|---|
| [`codelead interactive`](../cli/codelead-interactive.md) | The session ended with `/exit` or the end of the input. | The session ended on an error. | The start was refused, at a terminal or over JSON: the configuration, or a remote model that the privacy settings block. | |
| [`codelead chat`](../cli/codelead-chat.md) | The model answered. | The request failed. | The configuration was refused. | |
| [`codelead preflight`](../cli/codelead-preflight.md) | The analysis is done. | The analysis failed. | The configuration was refused. | |
| [`codelead implement`](../cli/codelead-implement.md) | Every card landed. An empty queue too. | A card did not land. | Nothing could run: no board, no card, a lock held, a remote model not allowed. A dry run whose model does not answer. | Ctrl+C. |
| [`codelead pr`](../cli/codelead-pr.md) | The pull request was opened, or the dry run printed. | Nothing to send, or a step failed. | | |
| [`codelead sessions`](../cli/codelead-sessions.md) | The list or the archive. | `export` found no session. | | |

### The repository

| Command | `0` | `1` | `2` |
|---|---|---|---|
| [`codelead map`](../cli/codelead-map.md) | The map is there: fresh, or written. | The map could not be read or written. | |
| [`codelead scan`](../cli/codelead-scan.md) | The scan ran. | An expected-absent key is present, or an error. | |
| [`codelead library`](../cli/codelead-library.md) | The library was printed. | The repository could not be indexed. | |
| [`codelead conventions`](../cli/codelead-conventions.md) | The conventions were extracted. | | |
| [`codelead lint`](../cli/codelead-lint.md) | The lint ran. | | `--request` was not given, the request cannot be read, or the root is not a folder. |

### Specs and walks

| Command | `0` | `1` | `2` |
|---|---|---|---|
| [`codelead spec check`](../cli/codelead-spec.md) | Every spec is usable. | A spec has a gap. | A path is missing, or a file cannot be read. |
| `codelead spec new` | Written. | Refused: the file is there, or the name is not valid. | |
| `codelead spec draft` | Every spec was drafted. | A spec was refused or failed. | No spec found, or a remote provider is blocked. |
| `codelead spec walk` | Every step passed. | A step failed or was blocked, or there was a finding. | Nothing was walked, or a journey was refused. |
| [`codelead walk`](../cli/codelead-walk.md) | Every step passed, with no finding. | A step failed or was blocked, a finding, or no step walked. | Nothing honest could be recorded: a refused spec, nothing that answers at `--base`, a browser that is not installed, a wrong option, no `--base`, `--rebuild` on the free plan. |
| [`codelead analyst`](../cli/codelead-analyst.md) | Each subcommand's. | Each subcommand's, and the free plan. | Each subcommand's. |

### Modernization

Every command of this table but `codelead modernize stop` needs the Pro plan. On the free plan, it ends with `2` before anything runs: `The modernization program is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.` The commands of the rebuild below need it too. See [Licence and plans](../concepts/licence-and-plans.md).

| Command | `0` | `1` | `2` |
|---|---|---|---|
| [`codelead modernize start`](../cli/codelead-modernize-start.md), [`next`](../cli/codelead-modernize-next.md), [`run`](../cli/codelead-modernize-run.md), [`continue`](../cli/codelead-modernize-continue.md) | The run ended with every stage done, or at a stage that waits for you. | A stage failed. | Refused: the free plan, an unknown stage, another process runs a program here, the state file cannot be read. `start` also in a folder where CodeLead is not initialized. |
| [`codelead modernize answer`](../cli/codelead-modernize-answer.md) | The answers were recorded. | | The free plan, or a key or a value was refused. |
| [`codelead modernize skip`](../cli/codelead-modernize-skip.md) | The stage was skipped. | | Refused: the free plan, no `--reason` option, an empty reason, a word that is no stage, the recordings, a stage that is pending, running, done or skipped already. |
| [`codelead modernize status`](../cli/codelead-modernize-status.md) | The state was printed. | No program in this folder. | The free plan, or the state file cannot be read. |
| [`codelead modernize stop`](../cli/codelead-modernize-stop.md) (free) | The stop was asked, or nothing ran. | | The state file cannot be read. |
| [`codelead modernize source`](../cli/codelead-modernize-source.md) | The profile was recorded, shown or taken out. | | The free plan, or the profile was refused. |
| [`codelead model`](../cli/codelead-model.md) | The extraction gate passed. | The gate did not pass. | The free plan, nothing could be extracted or written, or `--source` was not given. |
| [`codelead parity`](../cli/codelead-parity.md) | The capture fetched every route, the check passed, or the rebuild is finished. | A route could not be fetched, an unwaived difference, or a rebuild blocker. | The free plan, or nothing to compare: no mode, no routes, no baseline, no manifest. |
| [`codelead seed`](../cli/codelead-seed.md) | Written, and everything seeded. | Written, and something not seeded. | The free plan, or nothing written. |
| [`codelead register`](../cli/codelead-register.md) | Both files written. | | The free plan, or nothing written. |

### Rebuilds

| Command | `0` | `1` | `2` | `130` |
|---|---|---|---|---|
| [`codelead rebuild init`](../cli/codelead-rebuild-init.md) | Made, no blocker, and the dry run passed. | Made and not ready to run. | Refused, the free plan included: nothing was made. | |
| [`codelead rebuild`](../cli/codelead-rebuild.md) | The plan was printed. | The rebuild cannot be read, or the free plan. | | |
| [`codelead rebuild run`](../cli/codelead-rebuild-run.md) | The run ended: the queue's end, `--until`, a cap, the stop file. A dry run whose checks passed. | Refused (the free plan included), or stopped on a blocker, a held port or no progress. | | Ctrl+C. |
| [`codelead rebuild-report`](../cli/codelead-rebuild-report.md) | The page was written. | The page was not written. | Refused: the free plan, or the folder is not a rebuild. | |

### The board

| Code | Meaning for [`codelead board`](../cli/codelead-board.md) |
|---|---|
| `0` | Done. Also when there was nothing to change, and for a dry run. |
| `1` | A rule of the board refused the write, and nothing was written. `validate` found problems. |
| `2` | The command cannot run as written: no board, no such card, an unknown option, a missing or malformed argument. |

See [Board commands](../pinboard/cli/index.md) for each subcommand.

## Examples

- Stop a CI job when the machine lacks something:

  ```console
  codelead doctor || exit 1
  ```

- Tell a refusal from a failure in a script:

  ```bash
  codelead modernize next
  case $? in
    0) echo "done, or waiting for you: codelead modernize status" ;;
    1) echo "a stage failed: read its problem" ;;
    2) echo "refused: nothing changed" ;;
  esac
  ```

- Check a doctor run with an unreachable model server. A required check fails, so the code is `1`:

  ```console
  CODELEAD_BASE_URL=http://127.0.0.1:9/v1 codelead doctor; echo $?
  ```

  ```output
  …
  FAIL     Model server at http://127.0.0.1:9/v1                 Could not connect to lmstudio at http://127.0.0.1:9/v1.  → Start LM Studio's server (Developer tab); …
  …
  1 required check failed; fix the lines marked FAIL or MISSING and run codelead doctor again.
  1
  ```

## See also

- [Command-line reference](../cli/index.md)
- [Messages and what to do](messages.md)
- [Results and stops](../concepts/results-and-stops.md)
- [Unattended runs and the run lock](../concepts/unattended-runs.md)
- [Evidence record format](evidence-record-format.md)
