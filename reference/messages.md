---
title: Messages and what to do
description: The stops and the main refusals of CodeLead, word for word, grouped by where they come from - the patch check, the gate, the repair loop, composed requests, models, the modernization program, the licence and the board - with what each means and the way out.
kind: reference
area: reference
date: 2026-11-20
---

# Messages and what to do

**Applies to:** CodeLead 0.1 and later

CodeLead says why it stopped, and what you can do next, every time. This page lists the main messages word for word, grouped by the part of CodeLead that writes them. Look up the words that you see, read what they mean, and take the way out. In the words, `<…>` stands for a part that changes, such as a file name.

> [!NOTE]
> **New to this?** A *stop* is a run that ended before its goal. A stop is not always a failure: "You declined the patch; nothing changed" is your own choice. See [Results and stops](../concepts/results-and-stops.md), and [Fix a stop](../how-to/fix-a-stop.md) for the steps of each case.

## How a message is built

Every command ends with a **result**: a headline, a few facts, and the way on. In a terminal, it is the last line, such as `✖ Validation failed: tests exited 1; next: /repair`. In the extension and in the [session protocol](session-io-protocol.md#the-command-result), it is a card. A stop that needs you is also a **problem**: a title, the reason, the ways out and a **Retry**.

The result's outcome says what kind of end it is:

| Outcome | Meaning |
|---|---|
| `passed` | The verdict is a pass. |
| `failed` | The verdict is a failure, or the job could not be done. |
| `stopped` | It stopped short of its job: you answered no, or a step must come first. Not a failure. |
| `done` | A command with no verdict did its job. |
| `nothing` | There was nothing to do. |

## The patch check

The [patch check](../concepts/the-patch-check.md) reads every patch before it touches a file. Its status is `can_apply`, `cannot_apply`, `unsafe` or `invalid_patch`.

| Words | Meaning | Way out |
|---|---|---|
| `Patch check passed: <n> files can apply cleanly` | The patch fits the files as they are. | `/edit` applies it. `/implement` asks `Apply this patch? [Y/n]`. |
| `The patch check refused the patch: <the first problem>` | A blocking finding. | `/patch --retry` asks for a new patch with the problems. `/patch --show` shows it. |
| `<file>: Hunk 1 does not match current file contents. Tried line <n> and searched nearby/whole file. Expected context line was not found: "<line>".` | The patch was written against another version of the file. | `/patch --retry`. Or make the request name the file's current state. |
| `The patch changes nothing: every hunk leaves its file as it is.` | The model returned a patch with no change. | Say the change more precisely, then `/implement` again. |
| `Unsafe path "<path>": sensitive files are not allowed.` | The patch names a key file, such as `.env`. | Never applied. Keep secrets out of the request. |
| `Unsafe path "<path>": absolute paths are not allowed.` | The patch names a path of the machine. | `/patch --retry`. |
| `Unsafe path "<path>": parent directory segments are not allowed.` | The patch names a path with `..`. | `/patch --retry`. |
| `Unsafe path "<path>": path resolves outside the repository.` | The path leaves the repository. | `/patch --retry`. |
| `Unsafe path "<path>": generated, vendor, or build-output paths are not allowed, including dependency, cache, and session artifact paths.` | The patch edits `node_modules`, `dist`, `bin`, a session folder, and so on. | `/patch --retry`. |
| `… a note, it does not block applying` | A note of the change surface: a file outside the files that the request is likely to change. | Read it. You can apply. |
| `[conventions] <file>: <rule>` | A new line does not follow the repository's [conventions](../concepts/conventions.md). A note. | Apply, then fix the line, or `/patch --retry`. |
| `accepted_parts: the patch breaks what the accepted part <part> built (<record>): <what it breaks>. <the way out>` | In a rebuild: the patch removes an endpoint, a screen's route or a test that an accepted part built, or changes what such a test sends or asserts. | Do what the way out says: add the change beside what the part built, and leave that as it is. `/implement` tells its retry the same words. See [The rebuild](../modernization/rebuild.md#what-an-accepted-part-built-stays-built). |

When `/implement` gives up on a patch, it says:

```output
Implementation stopped.

Reason:
Patch check still failed after 2 retry attempts.

Next:
- Review with /patch --show
- Retry manually with /patch --retry if you want
- Adjust the request and run /implement again
```

An `unsafe` patch stops at once, with the reason `Patch check failed with non-recoverable safety or compliance issues.`

## The gate

The [gate](../concepts/validation-and-the-gate.md) runs the build and test commands after a patch applied. See [Validation file](validation-file.md).

| Words | Meaning | Way out |
|---|---|---|
| `Validation passed: <n> commands` | Every command exited 0. | The change lands. |
| `Validation failed: <command> exited <n>` | A command failed. The facts name it, its first error, and how many commands passed before it. | `/repair`, or fix it by hand, then `/validate`. |
| `Validation failed: <command> could not run: <why>` | The tool is missing or blocked, such as MSBuild on a Mac. | Do what `<why>` says, or declare the commands in `.codelead/validation.json`, then `/validate`. |
| `Validation failed: <command> stopped on the environment: <why>` | The environment is not ready, such as the database pins' own database. | Do what `<why>` says. The code is not at fault. |
| `Validation failed: <command> timed out` | The command ran past its time. | `/validate` again: a timeout says nothing about the code. |
| `Validation not run: the answer was no` | You answered no to `Run these commands? [Y/n]`. | `/validate` when you are ready. |
| `No validation commands found` | Nothing to run in this scope. | Declare the commands, or check by hand, then `/validate --override`. |
| `The change was applied, but validation failed: <why>` | `/implement` applied the patch, and the gate failed. | `/repair`. |
| `The change landed: applied to <n> files and validation passed` | Done. | `/pr` opens a pull request. |

## The repair loop

[Repair](../concepts/repair.md) rounds fix what the gate refused.

| Words | Meaning | Way out |
|---|---|---|
| `Nothing to repair: no failed validation in this session` | No failure to repair. | `/validate` first. |
| `Nothing to repair: no patch was applied in this session` | No patch to repair. | `/implement <request>`. |
| `The repair is ready to apply (<n> files); nothing is changed yet` | The repair patch passed its check. | `/edit`, then `/validate`. |
| `The repair produced no patch: <why>` | The model gave no change. | Change something, then `/validate`. Or `/validate --override`. |
| `The repair answer held no usable patch` | The answer could not be read as a patch. | `/repair` again. |
| `The repair patch could not be checked: <problem>` | The check refused the repair patch. | `/repair` again, or fix by hand. |
| `The repair loop stopped: <cycle>` | The loop went round in a circle twice. | See below. |

The words of a cycle:

- `This repair returns <file> to its state after <patch>, which failed with <error>.`
- `The validation is back at the failure after <patch>: <error>.`

After the first cycle, CodeLead asks for one more round, with the history, the cycle named and the scope widened. After a second cycle, it stops for you:

<!-- described, not captured -->
```output
The repair loop stopped for a person.

Reason:
<the cycle> The one more round asked after the first cycle went round again; CodeLead asks no further round.

What each round tried:
- The implementation patch: changed <files> → <command> exited 1: <error>
- Repair 1: changed <files> → …

Next:
- Read the failure and the files it names, fix it by hand, then /validate (a new failure starts the rounds afresh)
- Or say what to change and where: /patch <the change, in which file>
- Or run the request again with a narrower scope
```

## Composed requests: pins and transforms

A pin or an in-place transform is a **composed request**: CodeLead writes the request itself. When one does not land, CodeLead names the **class** of the stop, read from the session's state and records. Three classes are your own choice or wait on the gate: they are `stopped`, never a failure.

| Class | Headline | Outcome | Way out |
|---|---|---|---|
| `model_unreachable` | `The model server of the <role> row did not answer: <provider> at <address>` | failed | `Check the model server: codelead doctor`, then run it again. Nothing changed. |
| `declined_patch` | `You declined the patch; nothing changed` | stopped | `/edit` applies the kept patch. `/patch --show` shows it. Or compose it again. |
| `declined_validation` | `The pins were applied and not validated: the gate did not run` | stopped | `/validate`. A pass lands the increment, and declares the pins' gate entry. |
| `applied_not_validated` | the same, with `The validation did not run: <why>` | stopped | `/validate`. |
| `reply_cut` | `The change did not land: <the reply was cut>` | failed | A smaller request, or a higher cap by its configuration key. |
| `check_refused` | `The change did not land: <the check's problem>` | failed | Run it again. |
| `gate_refused` | `<title>: <n> of <m> tests failed: <test>: <message>` when tests failed, else `<title>: <command> exited <n>` | failed | CodeLead offers `The gate refused <subject>: <failure>. Repair it?` |
| `repair_stopped` | the repair loop's words | failed | As for the repair loop. |
| `precondition` | the words of what is missing, such as the pins' database | failed | Do what the words say. No model was called. |
| `objection` | the preflight's words: the request cannot be done | failed | Read the objection, and change the request. |

For a transform, the headline reads `Transform <type> was applied and not validated: the gate did not run`.

When you run a pin again over an applied increment that the gate never judged, CodeLead first asks:

```output
An increment is applied and not validated: validate it first? [Y/n]
```

Answer no, and nothing is composed: `An increment is applied and not validated; nothing was composed`. The ways on: `/validate`, or the same command with `--again`.

When you run a pin again over an applied increment that its gate refused, CodeLead first asks `An increment is applied and its gate refused it: run its gate again first? [Y/n]`. Answer no, and nothing is composed: `An increment is applied and its gate refused it; nothing was composed`.

The repair rounds of a composed request, and the pins that cannot run in the test process:

| Words | Meaning | Way out |
|---|---|---|
| `Repair round 3 of at most 4 (2 of the 2 before it moved the failure): <failure>` | The round past the limit, earned because the rounds before it moved the failure. | Nothing. The rounds go on by themselves. |
| `Repair rounds: <k> rounds ran and the failure did not change: the limit of <n> (modernization.repair.rounds) counts the rounds that did not move it.` | The rounds that did not move the failure reached the limit. When some rounds moved it, the line says `did not change in <s> of them`. | Read what each round tried. Fix it by hand, then `/validate`. |
| `Repair rounds: <k> rounds ran: the cap, twice the limit of <n> (modernization.repair.rounds); each moved the failure.` | The rounds reached the cap, twice the limit. | Read the last failure. Fix it by hand, then `/validate`. |
| `Set the <n> failing pins aside and land the <m> that pass? [Y/n]` (`1 failing pin` for one) | A few pins cannot run in the test process, and their repair left no patch. | Enter gives yes: the rest land. See [Pins](../modernization/pins.md#a-pin-that-cannot-run-in-the-test-process). |
| `<m> pins landed; <n> set aside: <test>: <reason>; the walk or the database level may prove it` | The increment landed without the tests set aside. | Read `setAside` in `pin-levels.json`. |
| `Not set aside: <n> of the increment's <m> tests failed, more than half: the request was wrong, not a few facts; nothing is set aside.` | Too many tests failed to set them aside. | Repair the increment, or pin fewer units. |
| `The gate did not pass without them either: the pins set aside are put back as they were; nothing landed.` | The set-aside did not help. | Repair the increment. |

## New apps

These words come from [`/create-app`](../chat-commands/create-app.md) and its build steps.

| Words | Meaning | Way out |
|---|---|---|
| `Planning stopped: the folder holds an existing project` (outcome `stopped`) | A no to the folder question: yours, or the recommended no of `--auto`. A workspace or root folder says `the folder looks like a workspace or root folder`. | Open or create a dedicated folder for the app, then send the request again. |
| `You did not confirm the package, so nothing was run (npm create <package>).` (outcome `stopped`) | You did not answer yes to a template package that is not on the trusted list. | Name the template again and answer `y`, give a trusted template, a folder or a git address, or build without a template. |
| `Automated build: answered no, as CodeLead recommends (<why>).` | `--auto` gave the recommended no to a question. | Read the reason. The command goes on, or stops, as after your own no. |
| `Automated build: the scaffold command failed; retrying it once.` | `--auto` runs a failed scaffold command once more by itself. | Nothing. A second failure gets the recommended no, and the model writes the files. |
| `Automated build: nobody confirmed the manual validation; the step's gates ran instead.` | A step with only a manual check. The step's gates decided it, and its record says `manualValidation: "unconfirmed"`. | Check the step by hand when the build is done. |
| `The automated build stopped: <what the step does>, and nobody is here to answer.` (outcome `stopped`) | An automated build came to a step that is hard to undo, such as `this deletes 4 entries of the target folder`. CodeLead did not answer it. An unattended run says `The unattended run stopped: …`. | Use the setting that the next line names, when it names one. Or run it where a person answers: the same command at the terminal or in the chat asks the question. |
| `The automated build stopped at a question CodeLead recommends no answer to, and nobody is here to choose.` (outcome `stopped`) | A question with no recommended answer, in an automated build or an unattended run. CodeLead does not guess. | Run the same command at the terminal or in the chat, and answer the question. |
| `The automated build did not start: <provider> was not allowed (the answer was no), and <role> uses it.` (outcome `stopped`) | You answered no to the consent for a remote model, asked at the start of `--auto` or `/build-incremental`. `/build-incremental` says `The incremental build did not start: …`. Nothing was sent. | Pick a local model for that role (Settings → Model roles). Or allow remote models without the question, once the code's owner agreed. Or send the command again and answer `y`. |

## Models and configuration

| Words | Meaning | Way out |
|---|---|---|
| `Could not connect to <provider> at <address>.` `If this is LM Studio, make sure it is open and the Local Server is running.` | The model server does not answer. | Start the server. [`codelead doctor`](../cli/codelead-doctor.md) checks it. |
| `The model of <roles> does not answer: <provider> (<address>) answered no list of its models (…).` | A dry run of [`codelead implement`](../cli/codelead-implement.md) or `codelead rebuild run` asked the model server for its list of models, and got no answer. Exit code 2. | Start the model server, or name a model that answers in `.codelead/config.json`. Then run the dry run again. |
| `The model call failed` | A problem card around the provider's words. | Read the words. When they name a model: `Open Settings → Model roles and pick a listed model`. |
| `Unable to complete preflight` / `The preflight did not complete: …` | The preflight's model call failed. | As above, then send the request again. |
| `Failed to create greenfield plan` / `Failed to create the increment plan` | The plan of a new app failed, often on the model server. | `codelead doctor`, start the server, send the request again. |
| `Remote model blocked by privacy settings.` | A role is on a remote provider, and remote models are not allowed. | Allow them in Settings (`privacy.allowRemoteModels`), or set `CODELEAD_ALLOW_REMOTE_MODELS=1`, once the code's owner agrees. |
| `Remote model request cancelled.` | You answered no to sending code to a remote provider. | Nothing was sent. Answer yes to send it. |
| `The automated build stopped: this sends repository snippets to <provider> / <model>, and nobody is here to answer.` (outcome `stopped`) | A consent for a remote model came up while nobody was there to answer. | Set `CODELEAD_ALLOW_REMOTE_MODELS=1`, or `privacy.confirmBeforeSendingRepoContextToRemote` to `false`, once the code's owner agreed. Or run it where a person answers. |
| `CODELEAD_MODEL names <model>, but the <role> role's provider is <provider> (the default role's is <provider>): a generic model variable does not apply across providers.` | The [override rule](environment-variables.md#the-override-rule). Exit code 2. | Unset `CODELEAD_MODEL`, or set `CODELEAD_MODEL_<ROLE>`, or change the role in the file. |
| `The model from the environment was refused` | The same rule, as a problem card in the chat. | The same, then start the chat again. |
| `Anthropic found no such resource (404) for <who>: <detail>` | The model's id is not one that Anthropic knows. | Use a bare id: `claude-sonnet-5-5`, `claude-opus-5-5`, `claude-haiku-5-5`, `claude-fable-5-1`. |
| `The anthropic provider needs a model name: …` | A role on Anthropic names no model. | Set `CODELEAD_MODEL`, `CODELEAD_MODEL_<ROLE>` or the role's `model`. |
| `Invalid config JSON at <path>.` `- <key>: <problem>` | A value of `.codelead/config.json` has the wrong type. | Fix the key that it names. See [Configuration file](configuration.md). |
| `The config at <path> is in a format this CodeLead does not read (schema …)` | The file is from another release. | Update CodeLead, or `codelead init --force`. |
| `CODELEAD_EXTRA_BODY_JSON is not valid JSON: <why>` | The variable holds no JSON object. | Fix or unset it. |

## The modernization program

The [program](../modernization/the-program.md) refuses with `Refused: … Nothing was recorded.`, exit code 2, and changes nothing. A stage that fails or waits carries a problem. See [Modernization state file](modernization-state-file.md#the-problem).

| Words | Meaning | Way out |
|---|---|---|
| `The modernization program is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.` | A command of the program, in a folder without the Pro plan. Exit code 2, and nothing ran. `codelead rebuild run` and `codelead rebuild --plan` exit with code 1. | Bring a licence of the Pro plan into the folder: `docs/codelead/licence.json`. See [Licence and plans](../concepts/licence-and-plans.md). |
| `Refused: There is no CodeLead configuration in this folder (.codelead/config.json): initialize the folder first (codelead init, or Initialize in the extension), then start the program. Nothing was written or run.` | `codelead modernize start` in a folder where CodeLead is not initialized. Exit code 2. | Run [`codelead init`](../cli/codelead-init.md), or select **Initialize** in the extension. Then start the program again. |
| `The modernization program did not start: <provider> was not allowed (the answer was no), and <role> uses it.` | You answered no to the consent for a remote model, at the start of the program in unattended mode. Exit code 2 at the terminal; the outcome `stopped` in the chat. | Pick a local model for that role. Or allow remote models without the question. Or send the command again and answer `y`. |
| `The Explorer needs the password of an account` (problem) | The read stage explores as an account of `--as`, and no password is set for its role. The problem names the variable, such as `CODELEAD_WALK_PASSWORD_CLERK`. Nothing was started. | Set the variable in the environment that the program runs in, or name a credentials file outside the folder with `CODELEAD_WALK_CREDENTIALS`. Then run the read stage again. |
| `The Explorer's sign-in as <role> was not accepted` (problem), or `The Explorer's sign-ins were not accepted` | The read stage's Explorer explored nothing: the application refused the sign-in. The dossier is written and kept. | Try the sign-in in a browser. Check the account and its `CODELEAD_WALK_PASSWORD_<ROLE>`, then run the read stage again. Or skip it: `codelead modernize skip read --reason "<WHY>"`. |
| `The Explorer explored no screen of the application` (problem) | No role reached a screen. The dossier is written and kept. | Open the address in a browser. A sign-in page needs an account: `codelead modernize start --base <URL> --as <ROLE>=<ACCOUNT>`. Or skip the read stage with a reason. |
| `The pins of <project> cannot run on this machine` (problem) | The pins of a .NET Framework project need the .NET Framework runtime or Mono, and this machine has neither. Nothing was asked of the model. | Install Mono, then run the pins again. Or pin the project on Windows. Or declare the pins' commands in `.codelead/validation.json`. |
| `Refused: there is no modernization program here (.codelead/modernization.json): …` | No program in this folder. | `codelead modernize start [--base <url>] [--unattended]`. |
| `Refused: .codelead/modernization.json: the stage <id> has the status "<x>", not one of pending, running, waiting, done, failed, skipped.` | The state file is not valid. | Fix the file, or remove it to start the program again. |
| `Refused: <stage> is not a stage of the program: read, recordings, pins, in-place, model, rebuild-init, rebuild, audit.` | A stage id that does not exist. | Use one of the ids. |
| `Refused: Record the journeys is not skipped: say the recordings are made (or that none will be): codelead modernize continue. Nothing was recorded.` | The recordings are never skipped. | `codelead modernize continue`. |
| `Refused: <stage> has not run (it is pending): a stage is skipped only when it failed or waits for you; …` | Only a stage that failed or waits can be skipped. | Run it first. |
| `Refused: security=maybe: fix-none (…), fix-all (…) or decided (…). Nothing was recorded.` | An answer's value is not one that it takes. | Give one of the values. |
| `Refused: pins=enough: Pin the contracts (pins) has not run (it is pending): pins=enough is taken only while the stage waits for you or failed; …` | `pins=enough` acts on the Pin stage, and only while it waits or failed. | Run the stage first: `codelead modernize run pins`. |
| `Refused: rebuild=enough: Build and verify (rebuild) has not run (it is pending): rebuild=enough is taken only while the stage waits for you or failed; …` | `rebuild=enough` acts on the Rebuild stage, and only while it waits or failed. | Run the stage first: `codelead modernize run rebuild`. |
| `Refused: <key>: not an answer the program takes (rebuildFolder=<folder>, …, pins=enough, rebuild=enough). Nothing was recorded.` | A key that the program does not take. | Use one of the nine keys. See [`codelead modernize answer`](../cli/codelead-modernize-answer.md). |
| `Refused: <stage> is skipped already (by the person: <reason>): codelead modernize next goes on.` | The stage was skipped before. | `codelead modernize next`. See [`codelead modernize skip`](../cli/codelead-modernize-skip.md). |
| `Record the journeys` (problem) | The Observe stage waits for your recordings. | Record with the Analyst, then `codelead modernize continue`. |
| `An answer is needed` / `Answers are needed` (problem) | A stage needs an answer. | The problem's `fix` gives the line: `codelead modernize answer <key>=<value>`. |
| `The extraction gate needs your reasons` (problem) | Findings of the extraction gate wait for a reason. | Record them in `extraction-dispositions.json`. See [Extraction dispositions and security waivers](dispositions-and-waivers.md). |
| `<n> security findings are decisions for a person` (problem) | Fixing them changes who may do what. | Waive or fix each, then `codelead modernize answer security=decided`. |
| `The rebuild waits for you` (problem) | The rebuild run ended with parts of the plan not accepted, and a person's turn stands between: a parked part, a part landed and not accepted, a part that the model ladder asked about, or a blocker of the plan. The reason names each, with its line. | Act on what the reason names in the rebuild, then `codelead modernize run rebuild`. Or go on with what is built: `codelead modernize answer rebuild=enough`. Or `codelead modernize skip rebuild --reason "<WHY>"`. |
| `The rebuild is not finished` (problem) | The rebuild run stopped at a cap, with parts left that it can still take. Nothing in the plan asks a person. | `codelead modernize run rebuild` goes on from what is accepted. Or `codelead modernize answer rebuild=enough`. |
| `<stage>: stopped` (problem) | You stopped the stage. | Run it again: what it finished is kept. |
| `<stage>: did not finish` (problem) | The process ended before the stage did. | Run the stage again. |
| `This clone is in use: <who>. One run per clone: …` | Another run holds the run lock. | Wait for it, or stop it. A lock with no heartbeat is taken over. |

In the extension's wizard: "The modernization state could not be read. Check .codelead/modernization.json, then refresh this page."

## The walk

[`codelead walk`](../cli/codelead-walk.md) stops with exit code 2, and records nothing, when it cannot walk honestly.

| Words | Meaning | Way out |
|---|---|---|
| `Nothing answers at <base> (<why>): nothing was walked, the run profile's reset was not run and no walk report was written. Start the application first …, then walk again.` | Nothing answered the walk's one request to `--base`, before the first journey. A report from an earlier walk stays as it was. | Start the application. For a rebuild, start the API and the web application of its run profile. Then walk again. |
| `The journey spec is refused and nothing was walked: The user key "<key>" is not one a placeholder can name: …` | A key of the spec's `users` does not start with a letter, or holds another character than letters, digits and underscores. | Rename the user in `users` and in the steps, for example `clerk_2`. See [Journeys format](journeys-format.md#users). |

## The licence

A licence problem is a warning, and never stops a run. Only the plan gate refuses. See [Licence and plans](../concepts/licence-and-plans.md).

| Words | Meaning | Way out |
|---|---|---|
| `The <feature> is in CodeLead's Pro plan; this folder's licence is <free, missing, invalid: why, or for other repositories (…)>. Ask for a licence that includes it.` | A pro command on a folder without the pro plan. The Analyst checks it, and so do the commands of the modernization program (`The modernization program is in CodeLead's Pro plan; …`). | Ask for a licence with the pro plan. |
| `Licence: the file docs/codelead/licence.json cannot be used (<why>).` | The file cannot be read, is not JSON, or `it changed since it was issued: its signature does not verify`. | Put back the file as it was issued. |
| `Licence: this repository is not among the licence's repositories (<project>).` | The licence names other repositories. | Ask for a licence that lists this one. |
| `Licence: <n> people active in the last 30 days, <n> seats (<n> blocks). Over by <n>. Ask for another block.` | More people ran CodeLead than the seats. | Ask for another block. |
| `Licence ends <date> (<n> days).` / `Licence ends <date> (today).` / `Licence ended <date>. Renew.` | The term ends within 14 days, today, or ended. | Renew. |

## The board

The [board](../pinboard/index.md) refuses a write with `Refused: … Nothing was written.` Each refusal has a code for programs. See [Rules for agents](../pinboard/rules-for-agents.md).

| Words | Meaning | Way out |
|---|---|---|
| ``No board at <dir> (no board.json there). Run `codelead board init` in the repository first.`` | No board in this folder. | [`codelead board init`](../pinboard/cli/board-init.md). |
| `No card #<id> on the board.` | The card does not exist. | Check the id with `codelead board list`. |
| `Refused: <agent> is an agent and may not move a card into <column> (a person's column); an agent may move a card into <columns>. Nothing was written.` | An agent tried a person's column. | Move it to a column that agents may use. |
| `Refused: <who> is an agent; the decision is the person's: an agent writes its recommendation in fields.ask. Nothing was written.` | Only a person decides. | Write the recommendation in `fields.ask`. |
| `Refused: <who> is an agent and may not tag a card for:codelead: the person queues a card for CodeLead; ask in the thread. Nothing was written.` | Only the person gives a card to CodeLead. | Ask the person in the card's thread. |
| `Refused: <who> is an agent and may not set a card's colour: the colour is the person's own mark. Nothing was written.` | The colour mark is the person's. | Leave the colour as it is. |
| `Refused: <what> looks like it holds a secret (a key, a connection-string password or a bearer token); nothing was written. …` | The text holds a secret. | Leave the secret out, and say where it is kept. |
| `Refused: <what> holds a path of this machine outside the workspace (under the home folder); nothing was written. …` | The text holds a machine path. | Cite the file by its path in the repository, or give `--rewrite-paths`. |
| `Refused: <what> names someone the board keeps private (listed in private-names.txt); nothing was written.` | A private name. | Leave the name out. |
| `Refused: card #<id> changed since it was read (…); nothing was written. Read the card again and redo the change.` | Someone wrote the card meanwhile. | Read it again, then write. |
| `Refused: another writer holds the board (.board.lock in <dir>); nothing was written. Retry; …` | Another writer holds the lock. | Retry. |
| `Refused: "<id>" is not a participant of the board (…); add it to board.json's participants first. Nothing was written.` | An unknown participant. | Add it to `board.json`. |

## The session protocol

| Words | Meaning |
|---|---|
| `Dropped a line that is not of the protocol (<why>): <line>` | The host sent a line that is not JSON, or not a known type. |
| `Dropped the answer to <id>: no question is open.` | The answer came after its question closed. |
| `Interrupt: the open question <id> is declined (no answer; nothing is done by default).` | An interrupt answered a yes-no with no answer. |
| `No answer can come (stdin is closed): …` | Standard input ended while a question was open. |

## See also

- [Results and stops](../concepts/results-and-stops.md)
- [Fix a stop](../how-to/fix-a-stop.md)
- [Exit codes](exit-codes.md)
- [Evidence record format](evidence-record-format.md#stop-reasons)
- [The patch check](../concepts/the-patch-check.md) and [Validation and the gate](../concepts/validation-and-the-gate.md)
- [Session protocol](session-io-protocol.md)
- [Troubleshooting](../extension/troubleshooting.md) in the extension
