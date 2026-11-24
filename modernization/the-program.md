---
title: The modernization program
description: The program behind the wizard and the commands - its state file, its commands, what a waiting stage waits for, and what approve and unattended mode mean.
kind: concept
area: modernization
date: 2026-11-20
---

# The modernization program

The modernization program runs the eight stages of a modernization, one after another, and remembers where it stands. The wizard in Visual Studio Code, the `codelead modernize` commands and the chat's `/modernize --run` all drive the same program. Its state is one file in the old application's folder: `.codelead/modernization.json`.

**In short**
- You know what the state file holds, and that every way to run the program reads and writes it.
- You know each command of the program, and its line in the chat.
- You know what a waiting stage waits for, and what to do about it.
- You know exactly what unattended mode does, and what it spends.

## Why a program

A modernization takes days. People stop, restart their machine, and come back. The program keeps the state of each stage in a file, so that nothing is lost and anyone can see where the work stands. It also keeps the order: a stage runs only when the stages before it are done or skipped.

## The state file

The program writes `.codelead/modernization.json` after every change of a stage's status. On the Basic Store sample, after the Read stage, `codelead modernize status --json` prints (shortened):

```output
{
  "schema": "codelead.modernization/1",
  "application": { "root": ".", "webProject": "webapp", "kind": "aspnet-mvc", "baseUrl": null },
  "mode": "approve",
  "answers": {
    "rebuildFolder": "../basic-store-rebuild",
    "template": "dotnet-next-shadcn",
    "framework": null,
    "identity": null,
    "database": null
  },
  "current": "recordings",
  "stages": [
    {
      "id": "read",
      "title": "Read the application",
      "status": "done",
      "command": "codelead modernize run read",
      "chat": "/modernize --run read",
      "outputs": ["docs/codelead/dossier/DOSSIER.md", "docs/codelead/dossier/options.md"],
      "evidence": ["ev-20261120050531-l4lg29"],
      "problem": null
    },
    …
  ]
}
```

Each stage has a status: `pending`, `running`, `waiting`, `done`, `failed` or `skipped`. A stage that waits or failed has a **problem**: a title, the reason, the way out (`fix`) and the command that runs it again (`retry`). The Pin stage also has its coverage. See [Modernization state file](../reference/modernization-state-file.md).

## The commands

| What you want | In a terminal | In the chat | In the wizard |
|---|---|---|---|
| Start, and run Read | [`codelead modernize start`](../cli/codelead-modernize-start.md) | (the terminal's line) | **Start** |
| Run the next stages | [`codelead modernize next`](../cli/codelead-modernize-next.md) | `/modernize --run` | (after **Save**) |
| Run one stage, again | [`codelead modernize run <STAGE>`](../cli/codelead-modernize-run.md) | `/modernize --run <STAGE>` | **Retry** |
| Say the recordings are made | [`codelead modernize continue`](../cli/codelead-modernize-continue.md) | `/modernize --continue` | **Continue** |
| Give answers | [`codelead modernize answer <KEY=VALUE>...`](../cli/codelead-modernize-answer.md) | `/modernize --answer <KEY>=<VALUE>` | **Save** |
| Go past a stage that failed or waits | [`codelead modernize skip <STAGE> --reason <WHY>`](../cli/codelead-modernize-skip.md) | `/modernize --skip <STAGE> --reason <WHY>` | **Skip this stage…** |
| See where it stands | [`codelead modernize status`](../cli/codelead-modernize-status.md) | | the page itself |
| Stop | [`codelead modernize stop`](../cli/codelead-modernize-stop.md) | the **Stop** button | **Stop** |
| Let CodeLead start the old application | [`codelead modernize source`](../cli/codelead-modernize-source.md) | `/modernize --source` | |

`next` runs one stage after another until a stage waits for you, a stage fails, or every stage is done. It runs a waiting stage again, because you can do what it waited for at any time. Three stages are the exceptions: it never runs the recordings, and it stops at a waiting Pin stage and at a waiting Rebuild stage. It never runs a failed stage again: fix the cause, then use `run`. Or go past the stage with `skip` and your reason.

`status` is the place to come back to. Its last line says what to run now:

```output
The modernization program of webapp (aspnet-mvc), approve mode; updated 2026-11-20T05:07:06.834Z.
  1. Read the application: done (05:05Z–05:05Z)
  2. Record the journeys: done (05:05Z–05:06Z)
  3. Pin the contracts: failed (05:06Z–05:06Z)  ← the program is here
       The pins of webapp need Visual Studio's MSBuild, and it is not on this machine: …
       The way out: Run it on Windows with Visual Studio or its Build Tools (the web development build tools workload): …
  4. Transform in place: pending
  5. Extract the model: done (05:07Z–05:07Z)
  6. Initialize the rebuild: pending
  7. Build and verify: pending
  8. Write the audit pack: pending
Answers: rebuildFolder=../basic-store-rebuild, template=dotnet-next-shadcn, framework=(none), identity=keep, database=sample-seed, pinsDatabase=done.
Next: codelead modernize run pins
```

This status comes from a Mac, which has no Visual Studio's MSBuild. The Model stage is done because it was run by hand with `codelead modernize run model`.

## What a waiting stage waits for

`waiting` means that it is your turn. The stage's problem says what it needs and the command to run after.

| Stage | It waits for | You do | Then |
|---|---|---|---|
| Read | The old application, when the program must start it and cannot (`Start the old application`). | Start the application by hand, or record how to start it. | `codelead modernize run read` |
| Read | An Explorer that explored nothing: every sign-in was refused, or no role reached a screen (`The Explorer's sign-in as clerk was not accepted`). The dossier is written. | Try the address and the sign-in in a browser. Check each account and its `CODELEAD_WALK_PASSWORD_<ROLE>`. Or go on without the crawl. | `codelead modernize run read`, or `codelead modernize skip read --reason "<WHY>"`, then `next` |
| Observe | The recordings, always. | Record the journeys with the Analyst. | `codelead modernize continue` |
| Pin | Provable facts that remain (`6 facts still wait at the in-memory level`). | Run the stage again, or accept the coverage. | `codelead modernize run pins`, or `codelead modernize answer pins=enough` |
| Pin | A database of the pins' own, for a database on a server. | Put a copy's connection string in the file that CodeLead prepared. | `codelead modernize answer pinsDatabase=done`, then `codelead modernize run pins` |
| Pin | Your decision after a declined patch, or an increment applied and not validated. | Apply it, look at it, or validate it, in the chat. | `codelead modernize run pins` |
| Upgrade in place | The security findings whose fix changes who may do what (`9 security findings are decisions for a person`). | Waive each one with its reason, or fix it on purpose. | `codelead modernize answer security=decided`, then `codelead modernize run in-place` |
| Model | Reasons for gate findings that are not the ASP.NET Identity template's. | Write each reason under its key in `extraction-dispositions.json`. | `codelead modernize next` |
| Set up the rebuild | The answers `identity` and `database`. | Give them. | `codelead modernize answer identity=keep database=sample-seed`, then `next` |
| Rebuild | A person's turn: parts of the plan that the run cannot take on by itself (`The rebuild waits for you`). The reason first counts the parts that are ready for the next run. Then it names each other part with its line, such as a parked part's `--reopen` line. | Act on each part in the rebuild, or go on with what is built. | `codelead modernize run rebuild`, or `codelead modernize answer rebuild=enough` |
| Rebuild | The run stopped at its caps with parts left (`The rebuild is not finished`). | Run the stage again, or go on with what is built. | `codelead modernize run rebuild`, or `codelead modernize answer rebuild=enough` |

[Waive a security finding](../how-to/waive-a-security-finding.md) says how to decide each security finding.

A **failed** stage is different: something went wrong that is not a question for you. Its problem says the cause and the way out. The program does not run a failed stage again by itself. Fix the cause and run the stage again. When the cause cannot be fixed now, go past the stage with [`codelead modernize skip`](../cli/codelead-modernize-skip.md) and your reason. The stage then ends `skipped`, and the state file keeps who skipped it, why and when. See [Fix a stop](../how-to/fix-a-stop.md#unblock-a-modernization-stage).

The Pin stage fails, for example, when the gate refuses an increment and its repair rounds end. Its ways out work from a terminal as well as in the chat:

- Run the stage again: `codelead modernize run pins`. It runs the gate of the pins that were applied first, and composes nothing over them.
- Or open the chat in this folder with `codelead interactive`, then run `/repair` and `/validate` there.
- Or go on with what is pinned: `codelead modernize answer pins=enough`.

See [Pins](pins.md#the-pin-stage-of-the-program).

## The answers

| Key | Values | Default |
|---|---|---|
| `rebuildFolder` | A folder beside the old application. | `../<folder>-rebuild` |
| `template` | A template name or path. | `dotnet-next-shadcn` |
| `framework` | `net10.0` or `net8.0`. | The template's |
| `identity` | `keep` or `drop`. | None: asked when the application signs users in |
| `database` | A connection string's name, or `sample-seed`. | None: asked when the application reaches a database |
| `pinsDatabase` | `done`, `env:<NAME>`, or the path of a copy of the database file. | None: asked for a database on a server |
| `security` | `fix-none`, `fix-all` or `decided`: what Upgrade in place does with the findings whose fix changes who may do what. | None: asked when such findings exist |
| `pins` | `enough`: the Pin stage ends with the coverage as it stands. | Taken only while the Pin stage waits or failed |
| `rebuild` | `enough`: the Rebuild stage ends with what is built, and the program goes on to the audit pack. | Taken only while the Rebuild stage waits or failed |

The first seven keys are **stored answers**. The program keeps them in the state file, and you can give one at any time, whatever stage waits. `pins=enough` and `rebuild=enough` are **acts** on their stages, and the state file does not keep them. At any other time, an act is refused:

```output
Refused: pins=enough: Pin the contracts (pins) has not run (it is pending): pins=enough is taken only while the stage waits for you or failed; run it first (codelead modernize run pins), then answer again if you choose. Nothing was recorded.
```

A key that the program does not take is refused, with the list of all nine. A connection string is never an answer: it holds a password, and a command line is kept in history.

## Approve mode and unattended mode

The program runs in one of two modes. You choose it when you start: **Approve each landing** or **Run unattended** in the wizard, or `codelead modernize start --unattended` in a terminal.

- **Approve mode** is the default. Before each landing of a pin or a transform, CodeLead asks `Apply this patch? [Y/n]`, and you answer in the terminal **CodeLead: modernize** or the chat. The recommended answer is in capitals, and Enter gives it.
- **Unattended mode** gives each question its recommended answer. A patch that passed the patch check with no blocking finding gets yes and lands. A patch with a blocking finding gets no. A pin that cannot run in the test process is set aside, so that the pins that pass land.

In unattended mode, a step that is hard to undo is never answered. The stage stops there at once, with the reason and the way out. A question with no recommended answer stops it too: CodeLead does not guess.

When a role uses a remote provider, unattended mode asks you for the consent at its start, before the first model call. It asks once for each remote provider that the configured roles would use. This happens when you run the program from the chat, or from a terminal where you can type. A no ends the command before any stage runs, with exit code 2 at the terminal and the outcome `stopped` in the chat:

<!-- described, not captured -->
```output
The modernization program did not start: anthropic was not allowed (the answer was no), and the repair role uses it.
  The way out: Pick a local model for the repair role (Settings → Model roles, or modelRoles in .codelead/config.json), then send the command again
  The way out: Once the code's owner agreed, set CODELEAD_ALLOW_REMOTE_MODELS=1 (environment or .env), or privacy.confirmBeforeSendingRepoContextToRemote false in .codelead/config.json
  The way out: Or send the command again and answer y
```

A consent that still comes up later, while nobody is there, stops the run. Approve mode asks the consent where it comes, as before. See [Privacy and remote models](../concepts/privacy-and-remote-models.md).

The Pin stage can ask one more question. The pins take their facts from the [narration](dossier.md#the-narration). When the web project has none, the stage asks `Narrate webapp now? [Y/n]` in approve mode, and yes is recommended. In unattended mode, it narrates without asking. Where no answer can come, for example when no terminal is attached, the question is declined, as every yes-or-no question is. See [Narrate the code before you pin it](../how-to/narrate-before-pins.md).

> [!WARNING]
> **Unattended means this:** CodeLead lands changes without asking you. It stops only at a refusal, a failure, or a step that is hard to undo. It spends money on the models, up to the caps. By default, the Pin stage runs up to 24 increments. The rebuild stops at 12 parts, 20 US dollars or 480 minutes. Read the dossier and the options before you choose it.

The Rebuild stage is unattended in both modes: [`codelead rebuild run`](../cli/codelead-rebuild-run.md) answers its own questions, under its caps. Some things only you can give, so they wait in both modes. They are the recordings, the answers, the reasons of the extraction gate, the pins' database, and the security decisions. The rebuild's parts that the run cannot take on by itself wait for you too, such as a parked part.

## One program at a time

One program runs in a folder at a time. A run takes the folder's run lock, `.codelead/run.lock`, and gives it back at its end. A second run is refused while the lock is held. Sometimes a stage shows `running` and nobody holds the lock. Then the process that ran it has ended, and the next run marks the stage failed, with that reason. See [Unattended runs and the run lock](../concepts/unattended-runs.md).

`codelead modernize stop`, in a second terminal, writes the stop file `.codelead/modernization.stop`. The running stage ends failed, "stopped by the person", and no further stage runs. During the Rebuild stage, the rebuild run stops after the part that is running. See [Stop and resume work](../how-to/stop-and-resume.md).

## Details

- **Evidence**: every stage that ends writes an [evidence record](../concepts/evidence-records.md), `modernize · <stage>`, with the outcome success, failure or asked.
- **Exit codes** of the subcommands: `0` when every stage is done or a stage waits for you. `1` when a stage failed. `2` when the command was refused, for example in a folder without the Pro plan. Your turn is not a failure.
- **`status --json`** prints the state file as it is. The wizard reads the same file and watches it.
- **The address**: `application.baseUrl` is `--base` when you give it, else the run profile's address.
- **Ignored by git**: the state file and the stop file are listed in `.codelead/.gitignore`.
- **Today's status view**: the wizard's page and `codelead modernize status`. A separate status page of the eight stages is planned, and is not built.

## What it is not

- **The program is not a second engine**. Each stage calls what CodeLead already has, such as the pin and transform actions of the chat, `codelead model` and `codelead rebuild init`. Running a stage by hand gives the same result.
- **The program does not run a shell line**. It runs CodeLead's own commands, in its own process.

## Related content

- [The eight stages](the-eight-stages.md)
- [`codelead modernize`](../cli/codelead-modernize.md) and [`/modernize`](../chat-commands/modernize.md#--run---continue---answer---skip)
- [The Modernize wizard](../extension/modernize-wizard.md) and [The Modernization view](../extension/modernization-view.md)
- [Modernization state file](../reference/modernization-state-file.md)
- [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md)
- [Results and stops](../concepts/results-and-stops.md)
- [Unattended runs and the run lock](../concepts/unattended-runs.md)
