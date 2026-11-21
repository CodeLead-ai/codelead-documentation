---
title: Analyst and Explorer
description: What the CodeLead Analyst and the Explorer are, when to use which, and the path from a recording to a specification that a rebuild is checked against.
kind: overview
area: analyst
date: 2026-11-20
---

# Analyst and Explorer

The **CodeLead Analyst** documents a running web application from its screens, with no source code. A person uses the application while the Analyst records each step. The Analyst then drafts journeys that CodeLead's [walk](../modernization/walk-and-parity.md) can replay. The **Explorer** is the part of the Analyst that visits the application's screens by itself and writes a site map, the forms and user stories.

**In short**
- Use the **Explorer** first, for a map of every screen and form in minutes. It needs no person.
- Use the **Analyst's recordings** next, for the journeys that matter, with what each step must show.
- A recording becomes a **journeys file** (`codelead.journeys/1`). You check it, replay it on the old application, and confirm it.
- No command of the Analyst calls a model. The Analyst and the Explorer are in the pro plan.

> [!NOTE]
> **New to this?** A *journey* is one task that a person does in the application, such as "create a product". A *step* is one part of it, such as "open the product form". See [Roles, journeys and steps](concepts.md).

## Who it is for

- **You are new to CodeLead.** You want to document an application that you did not write, or that nobody documented. Start with [Explore an application](explore-an-application.md), then [Record a journey](record-a-journey.md).
- **You already use coding agents.** You want checks that a rebuilt application does what the old one did, from the outside. Start with [From a recording to a specification](from-recording-to-spec.md).
- **You lead a team.** You want the behaviour of an old application written down, with evidence for each statement. Start with [Roles, journeys and steps](concepts.md) and [Limits](limits.md).

## The Explorer or the Analyst?

| | The Explorer | The Analyst's recording |
|---|---|---|
| Who drives | The Explorer, by itself | A person who knows the application |
| What it finds | Every screen it can reach, its forms, and what each role can see and do | The journeys that you choose, with what each step must show |
| What it knows of purpose | Nothing: it does not know what a journey is for | What you say, with **Expect this** and the context boxes |
| Time | 5 to 15 minutes for a small application | 20 to 40 minutes for five journeys |
| Command | [`codelead analyst explore`](reference/analyst-explore.md) | [`codelead analyst record`](reference/analyst-record.md) |
| In VS Code | The [Explorer view](../extension/explorer-view.md) | The [Analyst view](../extension/analyst-view.md) |

Use both. The Explorer covers the screens that nobody would record. Your recordings give the outcomes that only a person knows, such as the message that must appear when a SKU is already used.

## From a recording to a specification

The path is the same for a person's recording and for the Explorer's:

1. **Record or explore.** The Analyst writes `recording.json` in a folder under `analyst-recordings/`.
2. **Draft.** [`codelead analyst draft`](reference/analyst-draft.md) writes `journeys.json`, the specification, and `journeys.md`, the list of what to confirm.
3. **Check.** [`codelead analyst check`](reference/analyst-check.md) says whether the walk can read the specification.
4. **Replay.** [`codelead analyst replay`](reference/analyst-replay.md) walks it on the old application. Every step must pass.
5. **Confirm.** You read the list in `journeys.md`, change `journeys.json` where it says so, and check and replay again.

The [modernization program](../modernization/the-eight-stages.md) uses the result. In its Observe stage, you record the journeys that matter. Then [`codelead rebuild init`](../cli/codelead-rebuild-init.md) merges your journeys into the rebuild's specification, and the walk measures the new application against them.

> [!TIP]
> **For experienced users:** [`codelead analyst rules`](reference/analyst-rules.md) and [`codelead analyst facts`](reference/analyst-facts.md) read one or more recordings and write what the application enforces and what it holds. Each statement cites its step.

## Get started

- [Install CodeLead](../get-started/install.md): the Analyst needs Node.js 22 and Playwright's Chromium.
- [Explore an application](explore-an-application.md)
- [Record a journey](record-a-journey.md)

## Tutorials

- [Record journeys with the Analyst](../tutorials/record-journeys-with-the-analyst.md)
- [Modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)

## How-to guides

- [Explore an application](explore-an-application.md)
- [Record a journey](record-a-journey.md)
- [Turn a recording into a specification](from-recording-to-spec.md)
- [Give the Analyst its passwords](passwords.md)

## Concepts

- [Roles, journeys and steps](concepts.md)
- [Limits](limits.md)
- [The walk and parity](../modernization/walk-and-parity.md)
- [The Analyst view](../extension/analyst-view.md) and [the Explorer view](../extension/explorer-view.md) in VS Code

## Reference

- [`codelead analyst`](../cli/codelead-analyst.md): the command and its plan.
- [`codelead analyst record`](reference/analyst-record.md)
- [`codelead analyst explore`](reference/analyst-explore.md)
- [`codelead analyst draft`](reference/analyst-draft.md)
- [`codelead analyst check`](reference/analyst-check.md)
- [`codelead analyst replay`](reference/analyst-replay.md)
- [`codelead analyst rules`](reference/analyst-rules.md)
- [`codelead analyst facts`](reference/analyst-facts.md)
- [Files the Analyst writes](files.md)
- [Journeys format](../reference/journeys-format.md)
