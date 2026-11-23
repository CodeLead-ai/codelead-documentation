---
title: The dossier
description: What the Read stage writes about an old application, with no change to its code, and how to read the options report.
kind: concept
area: modernization
date: 2026-11-20
---

# The dossier

The dossier is CodeLead's report on an old application, written before anything changes. It says what the application is made of, what is risky in it, and which kinds of modernization fit. CodeLead writes it in the Read stage, from a survey of the repository that changes no code and calls no model.

**In short**
- You know what the survey reads, and which files of the dossier to open first.
- You can read a row of the options report: the kind of modernization, whether it fits, and why.
- You know what the narration adds, what it costs, and why the pins need it.

## Why it exists

A team that modernizes an application must first agree on what the application is. The dossier gives that agreement a written form, with the file and the line behind each finding. It also gives every later stage its inputs: the pins read its rules, and the transforms read its options.

## How it works

The [Read stage](the-eight-stages.md#1-read) runs the survey. You can also run it by hand in the chat with [`/modernize --plan`](../chat-commands/modernize.md#--plan).

1. **The survey** reads the repository. It reads the project files, the packages, the git history and the source files. When the projects are built, it also reads the built assemblies, to list the controllers, their actions and the data contexts. It writes nothing outside `docs/codelead/dossier/`.
2. **The intake** asks twelve questions about your goals and constraints, for example "What must not change?". In the program, and wherever no terminal can answer, CodeLead takes each default and marks it *assumed*.
3. **The options** are derived from the survey and the intake. CodeLead judges twelve kinds of modernization, T1 to T12, and proposes an order.

> [!TIP]
> Build the projects before the survey when you can. For a .NET Framework project, that means a build on Windows. Without a built assembly, the survey lists no action, and the narration has nothing to narrate.

### The sections

`DOSSIER.md` is the first page to open. Its table lists each section, with the files behind it and how much of the section the tool did by itself. On the Basic Store sample the dossier says `66% tool-driven by section weight — 7 automated, 7 partial, 2 manual.`

| Section | What it says | Files |
|---|---|---|
| D1 | The projects, their files and the dependencies between them. | `project-map.md` |
| D2 | The public types and methods, and the external integration points. | `contracts.md` |
| D3 | The packages, and the runtimes against their end-of-life dates. | `dependencies.md` |
| D4 | The git history: hot files, files that change together, few authors. | `git-history.md` |
| D5 | The business rules found in the code, each with its file and line. | `business-rules.md` |
| D6 | Where tests are missing, and how hard each area is to pin. | `test-census.md` |
| D7 | Modernization candidates, grouped in waves by dependency. | `wavefront.md` |
| D11 | Improvement opportunities, ranked by consequence. | `opportunities.md` |
| D12 | Coding patterns that make a change risky. | `coding-practices.md` |
| D13 | Readiness: formats, targets, support status, the build floor. | `modernization-readiness.md` |
| D14 | The structure read from the built assemblies. | `functional-structure.md` |
| D15 | The security exposure, found without running the application. | `security-exposure.md` |

D5b (interviews) and D8 (the validation strategy) are work for people. The dossier says so, and does not pretend to do them.

The readiness page gives the project names as the commands take them. On the Basic Store sample, the project is `webapp`, and its build floor reads:

```output
- 1 legacy-format .NET project: builds with MSBuild/Visual Studio on Windows (Mono reference assemblies elsewhere); `dotnet build` needs the SDK-style format
- targets .NET Framework: needs the matching targeting pack (or the reference-assemblies package on non-Windows)
```

### The options report

`options.md` judges each kind of modernization. Each row has a fit: **yes**, **possible, not indicated**, or **not assessed**. "Yes" means that the code and the intake both call for it. "Possible, not indicated" means that it is always available, and the row says what would make it a yes.

These are four rows of the options report of the Basic Store sample:

```output
| Type | Fit | Why | Scope | Pins + transforms | Tool h | Human h | Elapsed (working days) | Confidence |
|---|---|---|---|---|---|---|---|---|
| T1 Platform upgrade in place | yes | 1 runtime(s) out of support and 0 ending within a year; 1 legacy-format project(s) | webapp | 1 + 1 | 1.9 | 1 | 1–1 | low |
| T5 Module extraction / strangler | yes | webapp holds the EF data context StoreContext (4 entities) and no class library does: extract the data layer first with /modernize --transform T5D --scope webapp | webapp | 1 + 1 | 1.9 | 1 | 1–1 | low |
| T6 Full behaviour-preserving rewrite | possible, not indicated | always possible; not indicated because the intake's appetite is "upgrade in place where possible, rewrite only what blocks it" — the estimate below is the comp… | webapp | 16 + 20 | 53.5 | 26.8 | 10–20 | low |
| T12 Retire and replace | not assessed | not assessed by the survey: a build-versus-buy decision needs the market, not the code | - | - | - | - | - | - |
```

Read a row from left to right:

1. **Type**: the kind of modernization. T1 is a platform upgrade in place. T5 is a module extraction. T12 is a decision to buy instead of build.
2. **Fit** and **Why**: whether it fits, and the fact in the code or the intake behind that.
3. **Scope**: the projects that it touches.
4. **Pins + transforms**: how many increments it needs, first the pins, then the changes.
5. **Tool h**, **Human h** and **Elapsed**: an estimate. The basis is under the row, in its own section.
6. **Confidence**: how much the estimate is worth. "Low" means that one calibration project measured it.

Under the table, the report recommends an order. On the Basic Store sample it is `Sequence: T5 → T10 → T1 → T2 → T8.`, with one reason for each step. Here the first reason is the rule of pinning first, because the project has no tests.

> [!NOTE]
> **New to this?** CodeLead composes only some of these kinds by itself: T1, T5 and its data-layer form T5D, T10, and S for security. A person writes the others with [`/implement`](../chat-commands/implement.md), and the row is the brief. See [Transforms](transforms.md).

The T10 row names the unmaintained packages to put behind an adapter. It names the framework that a project is built on apart, because no adapter can replace it. On the Basic Store sample, the row says `not for an adapter, the platform itself: T1 or the rebuild: webapp: Microsoft.AspNet.Mvc, the ASP.NET MVC it is built on`.

### The narration

The narration is optional, and it calls a model. For each unit that the survey found (a controller action, a handler, a data context), a model writes what the code does in plain statements. Each statement is anchored to the lines that it rests on. CodeLead drops a statement whose lines fall outside the unit.

The narration adds two things:

- **The facts that the pins prove.** Each rule or output statement becomes a fact. The [pins](pins.md) decide the level of each fact, and count their coverage in facts. When there is no narration, the program's Pin stage offers to make one first. Without it, the stage runs the in-memory pins once, and nothing counts the facts.
- **A base for people's knowledge.** [`/modernize --testimony`](../chat-commands/modernize.md#--testimony) matches what people wrote about the application against the narration.

Run it in the chat with [`/modernize --narrate`](../chat-commands/modernize.md#--narrate). It makes one model call for each unit. On the Basic Store sample, after a build on Windows, it narrated 51 units in about 11 minutes on a local model.

## Details

- **The bundle**: 34 files on the Basic Store sample, in `docs/codelead/dossier/`. `README.md` lists them. `options.json`, `modernization-readiness.json` and `security-exposure.json` are what the later stages read.
- **The intake**: `intake.json` and `intake.md`. To answer the twelve questions yourself, run `/modernize --plan --intake` in a terminal session. The stored answers become the defaults.
- **End-of-life dates**: CodeLead ships a snapshot. `/modernize --plan --eol-refresh` downloads current dates.
- **A copy without code**: `/modernize --plan --redacted` also writes `redacted/`, a copy of the dossier without source lines, for a reader who must not see the code.
- **The narration's files**: `functional-narration.json` and `functional-narration.md`, in the bundle. Every statement starts with the status `Inferred`. A pin that proves it changes it to `Pinned`.
- **The knowledge base**: the survey also seeds findings into `.codelead/knowledge.json`. See [The knowledge base](../concepts/knowledge-base.md).
- **The Explorer**: when the program has the application's address, the Read stage also runs the Explorer. It writes `analyst-recordings/<time>/`, with `sitemap.json`. With `--as <role>=<account>` on `codelead modernize start`, the Explorer signs in as each account. See [Explore an application](../analyst/explore-an-application.md).

## What it is not

- **The dossier is not a plan that CodeLead follows blindly**. The options and the order are a proposal. The program takes the transforms from it, and you can run others by hand.
- **The dossier does not know why the code is what it is**. It states what the code does. The reasons live with people, and the dossier marks those sections as manual.
- **The dossier is not a test**. Nothing in it proves behaviour. The pins and the journeys do that.

## Related content

- [The eight stages](the-eight-stages.md)
- [Pins](pins.md) and [Transforms](transforms.md)
- [`/modernize`](../chat-commands/modernize.md): `--plan`, `--narrate`, `--testimony`, `--waive`
- [`codelead modernize start`](../cli/codelead-modernize-start.md): runs the Read stage
- [`codelead scan`](../cli/codelead-scan.md)
- [The audit pack](audit-pack.md): it copies the dossier into its first layer
- [Glossary](../concepts/glossary.md)
