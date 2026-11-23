---
title: Modernization
description: How CodeLead moves an old application to a current platform, and records what the application does before anything changes.
kind: overview
area: modernization
date: 2026-11-20
---

# Modernization

Modernization moves an old application to a current platform without losing what it does. CodeLead first records the application's behaviour, as tests and journeys. Then it changes the old code, or builds a new application beside it, and checks every change against that record.

**In short**
- Modernization in CodeLead is one program of eight stages, the same for every application.
- CodeLead does most of the work. You record the journeys, give two answers, and approve the changes.
- You run the program with the wizard in Visual Studio Code, or with commands in a terminal or in the chat.
- Every change that lands writes an [evidence record](../concepts/evidence-records.md), and the last stage gathers them in an audit pack.

## Where to start

- **I am new to CodeLead**: follow the [tutorial](../tutorials/modernize-a-dotnet-framework-app.md). It takes the Basic Store sample through all eight stages.
- **I already use coding agents**: read [The eight stages](the-eight-stages.md), then [Pins](pins.md). The pins are what makes a change safe here.
- **I lead a team**: read [The modernization program](the-program.md) and [The audit pack](audit-pack.md). They say who acts, what is recorded, and what a reviewer gets.

## Who it is for

- **A developer** who must move an old .NET Framework application to current .NET, and must show that it still does the same things.
- **A technical lead** who must show what changed, why it changed, and how each change was checked.
- **A team that does not know the old code well**. CodeLead reads the code first, and writes what it finds in plain words.

## The eight stages

The stages run in this order. Each stage reads what the stages before it wrote. The times are rough, from runs on the Basic Store sample, and depend on your application and your model.

| # | Stage | What CodeLead does | Who acts | How long, roughly |
|---|---|---|---|---|
| 1 | **Read** | Reads the code with no change and no model call, and writes the [dossier](dossier.md). With an address, the Explorer visits the running application's screens. | CodeLead | Seconds for the survey; 5 to 15 minutes with the Explorer |
| 2 | **Observe** | Waits while a person records the journeys that matter with the Analyst. | You | 20 to 40 minutes for five journeys |
| 3 | **Pin** | Writes [pins](pins.md): tests that record what the code does today. | CodeLead; you approve | 1 to 2 hours, with model calls |
| 4 | **Upgrade in place** | Runs the [transforms](transforms.md) that fit: the data layer into a library, the library's platform upgrade, dependencies, security findings. | CodeLead; you approve each landing | 30 to 90 minutes, with model calls |
| 5 | **Model** | Reads the application into its [System Model](system-model.md) and its feature map, with no model call. | CodeLead | About a minute |
| 6 | **Set up the rebuild** | Makes the new application's folder from a template, with its plan. | CodeLead; you give two answers | Minutes, plus the package install |
| 7 | **Rebuild** | Builds the new application part by part, then runs [the walk and parity](walk-and-parity.md). | CodeLead, unattended | Hours, with model calls, under caps |
| 8 | **Hand over** | Writes the [audit pack](audit-pack.md): before, after, and the evidence of every landing. | CodeLead | About 2 minutes |

See [The eight stages](the-eight-stages.md) for what each stage needs, produces and says.

## What works today, and what does not

- **ASP.NET MVC on .NET Framework is the class that CodeLead reads in full**. The Basic Store sample is one.
- **Web Forms pages are not read**. For a Web Forms project, the System Model holds only the database. There is no rebuild of its pages.
- **Windows is needed for the pins and the transforms of a .NET Framework web project in the old project format**. They build with Visual Studio's MSBuild, and it runs on Windows only. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md). The pins of an SDK-style .NET Framework project can also run on macOS or Linux, when Mono is installed. See [Pins](pins.md#details).
- **A model is needed for the narration, the pins, the transforms and the rebuild**. The other parts call no model: the dossier, the Explorer, the System Model, the rebuild's set-up and plan, the walk, parity and the audit pack.
- **The walk and the Explorer need the application running**. CodeLead can start it for you. See [Start the old application for CodeLead](source-run-profile.md).
- **The modernization program is in the Pro plan**: the Explorer, the Analyst, the wizard, and every command of the program. That is `codelead modernize`, the chat's `/modernize`, `codelead rebuild`, `codelead model`, `register`, `seed`, `parity`, `rebuild-report` and `codelead walk --rebuild`. On the free plan, each one is refused before it runs. `codelead modernize stop`, every `--help`, and the walk of a spec stay free. See [Licence and plans](../concepts/licence-and-plans.md).

> [!WARNING]
> The pins, the transforms and the rebuild call models for every increment. With a paid provider, they cost money. One recorded run of the pins on the Basic Store sample cost about 13 US dollars, for 14 calls to a remote model. The rebuild's run stops at a cost cap, 20 US dollars by default.

## Two ways to run it

Both ways drive the same program. The program keeps its state in one file, `.codelead/modernization.json`, so you can start in one way and continue in the other.

- **The wizard in Visual Studio Code**. In the Modernization view, select **Modernize this application** (the rocket in the view's title bar). The page shows each stage with its status, its problem and a **Retry** button. Each button shows the command that it runs. See [The Modernize wizard](../extension/modernize-wizard.md).
- **The commands**. In a terminal, run [`codelead modernize start`](../cli/codelead-modernize-start.md), then [`codelead modernize next`](../cli/codelead-modernize-next.md) when the program waits for you. In the chat, type [`/modernize --run`](../chat-commands/modernize.md#--run---continue---answer---skip).

To see where the program stands, open the wizard's page, or run [`codelead modernize status`](../cli/codelead-modernize-status.md). A separate status page for the eight stages is planned, and is not built yet.

> [!NOTE]
> **For team leads:** in approve mode, which is the default, CodeLead asks before each change lands. Unattended mode lands changes without asking. See [Approve mode and unattended mode](the-program.md#approve-mode-and-unattended-mode).

## Get started

- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)

## Concepts

- [The eight stages](the-eight-stages.md)
- [The dossier](dossier.md)
- [Pins](pins.md)
- [Transforms](transforms.md)
- [The System Model](system-model.md)
- [The rebuild](rebuild.md)
- [The walk and parity](walk-and-parity.md)
- [The audit pack](audit-pack.md)
- [The modernization program](the-program.md)

## How-to guides

- [Start the old application for CodeLead](source-run-profile.md)
- [Build .NET Framework projects on Windows](../how-to/build-on-windows.md)
- [Declare validation commands](../how-to/declare-validation-commands.md)
- [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md)
- [Fix a stop](../how-to/fix-a-stop.md)

## Reference

- [`codelead modernize`](../cli/codelead-modernize.md) and its subcommands
- [`/modernize`](../chat-commands/modernize.md): every modernization action in the chat
- [`codelead model`](../cli/codelead-model.md), [`codelead rebuild init`](../cli/codelead-rebuild-init.md), [`codelead rebuild run`](../cli/codelead-rebuild-run.md)
- [`codelead walk`](../cli/codelead-walk.md) and [`codelead parity`](../cli/codelead-parity.md)
- [Modernization state file](../reference/modernization-state-file.md)
- [Rebuild manifest](../reference/rebuild-manifest.md) and [Run profile](../reference/run-profile.md)
- [Journeys format](../reference/journeys-format.md) and [Parity waivers](../reference/parity-waivers.md)
