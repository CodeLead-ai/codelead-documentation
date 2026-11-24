---
title: Glossary
description: Every term of CodeLead in one or two plain sentences, in alphabetical order, each with a link to the page that explains it.
kind: concept
area: concepts
date: 2026-11-20
---

# Glossary

This page explains every term that CodeLead uses, in alphabetical order. Each entry has one or two plain sentences and a link to the page that explains the idea in full. When a word on another page is new to you, look for it here.

**In short**
- Each term has one meaning in this documentation. Pages use the same word for the same thing.
- Each entry links to its concept page, or to the reference page of its command.
- Start with **landing**, **patch**, **the patch check**, **the gate** and **evidence record**: most pages use them.

## A

- **Acceptance probe**. A check of one increment of a new app. CodeLead starts the app in a browser and looks for what the increment promised. See [`/build-incremental`](../chat-commands/build-incremental.md).
- **Analyst**. The CodeLead Analyst records a running application from its screens while a person uses it, and drafts journeys from the recording. See [Analyst and Explorer](../analyst/index.md).
- **Apply**. To write a checked patch into your files. Only your yes, or the fixed rules of an unattended run, applies a patch. See [Requests, plans and patches](requests-plans-and-patches.md).
- **Assumption**. A choice that the preflight would make unless you say otherwise. You accept, reject or correct the list in one answer. See [Prompts and recommended answers](prompts-and-recommended-answers.md).
- **Audit pack**. A folder that collects the evidence of a modernization for a reviewer, with secrets removed. See [The audit pack](../modernization/audit-pack.md).

## B

- **Blocking finding**. A problem that the patch check found and that stops a patch. Compare with a **note**. See [The patch check](the-patch-check.md).
- **Board**. Pinboard: the cards between you and your coding agents, kept as files in your repository under `docs/codelead/board`. See [Pinboard](../pinboard/index.md).
- **Build step**. One small part of a new app's execution plan, built and validated before the next one. See [`/build-step`](../chat-commands/build-step.md).

## C

- **Capability profile**. A named set of settings that fits how much work one model can be trusted with in a single call. See [The knowledge base](knowledge-base.md).
- **Capsule**. CodeLead's word for one unit of work under its rules: a request with its scope, plan, patch, gate and evidence. Today you see it as one [`/implement`](../chat-commands/implement.md) run, one build step or one increment. See [The knowledge base](knowledge-base.md).
- **Card**. One piece of work on the board, with a title, a description and a thread of comments. See [Cards, columns and participants](../pinboard/concepts.md).
- **Change surface**. The files that CodeLead expects a request to change. A patch that edits a file outside it gets a note. See [The patch check](the-patch-check.md).
- **Chat**. Questions in plain words about your repository. The chat never changes code: for a change, it offers the command to run. See [`/chat`](../chat-commands/chat.md) and [The chat](../extension/chat.md).
- **Checkpoint commit**. A git commit that CodeLead makes after a verified increment, so that every earlier increment stays safe. See [`/build-incremental`](../chat-commands/build-incremental.md).
- **Clarifying question**. A question that the preflight asks when the request is not clear. It shows a default, which is the recommended answer. See [Prompts and recommended answers](prompts-and-recommended-answers.md).
- **Composed request**. A request that CodeLead writes itself, for a pin or a transform. Its preflight asks nothing and records what it assumed. See [Requests, plans and patches](requests-plans-and-patches.md).
- **Context package**. Everything that one model call receives: the request, the plan, the chosen files, the map and the conventions. See [The knowledge base](knowledge-base.md).
- **Conventions**. The rules that new code must follow in a repository. Some you write yourself, and CodeLead reads the others from the code. See [Conventions](conventions.md).
- **Cycle**. A repair that would bring a file back to a state that an earlier round already tried. A second cycle stops the repair loop. See [Repair](repair.md).

## D

- **Default**. The answer that a clarifying question shows. It is CodeLead's recommended answer, and Enter takes it. See [Prompts and recommended answers](prompts-and-recommended-answers.md).
- **Dossier**. The read-only survey of an old application that the modernization starts with: its projects, rules, risks and options. See [The dossier](../modernization/dossier.md).

## E

- **Embedding**. A list of numbers that stands for what a text means. Two texts that mean the same thing get lists that point the same way. The PKB service searches by meaning with them. See [`codelead pkb embed`](../cli/codelead-pkb-embed.md).
- **Encyclopedia**. The project's knowledge entries in `.codelead/knowledge.json`, each with its trust level and its evidence. See [The knowledge base](knowledge-base.md).
- **Evidence record**. A small JSON file that CodeLead writes at the end of every run. It says what was asked, what changed and what the checks said. See [Evidence records](evidence-records.md).
- **Explorer**. The part of the Analyst that visits a running web application by itself and writes what it saw on each screen. See [Explore an application](../analyst/explore-an-application.md).

## G

- **Gate, the**. The set of your project's own commands, such as the build and the tests, that decides whether a change is done. See [Validation and the gate](validation-and-the-gate.md).

## H

- **Headline**. The one sentence of a result that says what happened, for example "Validation failed: npm test exited 1". See [Results and stops](results-and-stops.md).
- **Hunk**. One block of changed lines in a patch, with a few unchanged lines around it. See [The patch check](the-patch-check.md).

## I

- **Increment**. A slice of a new app that works on its own: first the skeleton, then one capability at a time. Each verified increment gets a checkpoint commit. See [`/build-incremental`](../chat-commands/build-incremental.md).
- **Initialize**. To prepare a folder for CodeLead: the configuration, the ignore lines for keys, and the repository map. [`codelead init`](../cli/codelead-init.md) does it, and so does the extension's **Initialize** button.

## J

- **Journey**. A path that a person takes through an application, for example "create a product". A walk follows journeys. See [Roles, journeys and steps](../analyst/concepts.md).

## K

- **Knowledge base**. What CodeLead knows about a project and why: the encyclopedia, the evidence records and the repository map. See [The knowledge base](knowledge-base.md).

## L

- **Landing**. A change that was applied and whose validation passed. A landing is a finished change. See [How CodeLead works](how-codelead-works.md).
- **Licence**. A signed file, `docs/codelead/licence.json`, that names the holder, the project, the seats, the term and the plan. See [Licence and plans](licence-and-plans.md).

## M

- **Model**. A large language model: a program that writes text and code from a request. CodeLead contains no model. See [Models and roles](models-and-roles.md).
- **Model server**. A program that runs a model and answers requests from other programs, such as LM Studio on your computer. See [Set up local models](../how-to/set-up-local-models.md).
- **Modernization program**. The eight stages that take an old application to a new one, with its behaviour recorded first. See [The modernization program](../modernization/the-program.md).

## N

- **Non-goal**. Something that the plan says the change must not touch. A patch that crosses one gets a note. See [Requests, plans and patches](requests-plans-and-patches.md).
- **Not in the knowledge base**. What the PKB service says when nothing it found clears its floor. Then no model writes an answer. See [`codelead pkb search`](../cli/codelead-pkb-search.md#not-in-the-knowledge-base).
- **Note**. A finding of the patch check that never blocks a patch, for example a convention that a new line does not follow. See [The patch check](the-patch-check.md).

## O

- **ORM**. An object-relational mapper: a library that maps a database's tables to classes, such as Entity Framework. T10 never puts one behind an adapter when the code uses its query types directly. See [Transforms](../modernization/transforms.md#t10-a-dependency-behind-an-adapter).
- **Outcome**. The one word of a result: `passed`, `failed`, `stopped`, `done` or `nothing`. See [Results and stops](results-and-stops.md).

## P

- **Parity**. A comparison that shows whether the new application does what the old one did. See [The walk and parity](../modernization/walk-and-parity.md).
- **Patch**. A description of changes to files, as a diff, that nobody has applied yet. See [Requests, plans and patches](requests-plans-and-patches.md).
- **Patch check, the**. CodeLead's own reading of a patch against your files, before the apply and with no model. See [The patch check](the-patch-check.md).
- **Pin**. A test that records what the old code does today, so that a later change cannot break it unseen. See [Pins](../modernization/pins.md).
- **PKB service**. The project knowledge base shared by a team: what several projects know, in one place, with every answer cited. Today its store, its sync and its search exist: [`codelead pkb sync`](../cli/codelead-pkb-sync.md) sends a project's knowledge into the store, and [`codelead pkb search`](../cli/codelead-pkb-search.md) finds it again. See [`codelead pkb`](../cli/codelead-pkb.md).
- **Plan (of a change)**. The preflight's result: the files to change, the non-goals and how to validate. The patch check holds the patch to it. See [Requests, plans and patches](requests-plans-and-patches.md).
- **Plan (of a licence)**. Free or pro: which parts of CodeLead a folder can use. See [Licence and plans](licence-and-plans.md).
- **Preflight**. The first step of a change: CodeLead chooses the files, and a model analyzes them and writes the plan. It can ask questions. See [Requests, plans and patches](requests-plans-and-patches.md).
- **Probe authority**. How much the probes of an increment decide. `evidence`, the default: a probe's finding is a warning, and code that builds and passes its tests is kept. `gate`: a finding refuses the attempt. See [New apps](new-apps.md#the-gates-of-an-increment).
- **Problem**. A failure that needs you, with its reason, the ways out and the command to try again. In the chat it is a card with a **Retry** button. See [Results and stops](results-and-stops.md).
- **Project library**. CodeLead's index of a repository's files, modules, symbols and relationships, built from the code with no model. See [The project library](project-library.md).
- **Provider**. A model server that CodeLead can call, local or remote, named in the configuration. See [Models and roles](models-and-roles.md).

## R

- **Rebuild**. A new application built in its own repository from what the old one does, part by part. See [The rebuild](../modernization/rebuild.md).
- **Recommended answer**. The answer that CodeLead would give in your place. Enter gives it, except for a step that is hard to undo. See [Prompts and recommended answers](prompts-and-recommended-answers.md).
- **Register**. The behaviour register: every rule and behaviour found in an old application, each with the check that the new one must pass. See [The behaviour register and seed data](register-and-seed.md).
- **Remote model**. A model on a provider outside your machine and your network. CodeLead sends it nothing without your consent. See [Privacy and remote models](privacy-and-remote-models.md).
- **Repair round**. One round of [`/repair`](../chat-commands/repair.md), [`/edit`](../chat-commands/edit.md) and [`/validate`](../chat-commands/validate.md): a small fix for what the gate refused. See [Repair](repair.md).
- **Repair role**. The role whose model writes repairs. You can give it a stronger model than the rest. See [Models and roles](models-and-roles.md).
- **Repository map**. CodeLead's summary of a repository: the stack, the validation commands, the modules and the entry points. See [The repository map](repository-map.md).
- **Request**. What you want changed, in plain words. See [Requests, plans and patches](requests-plans-and-patches.md).
- **Result**. How every command ends: an outcome, a headline, a few facts and the way on. See [Results and stops](results-and-stops.md).
- **Role**. A kind of work, such as `preflight` or `repair`, that points to a provider and a model. See [Models and roles](models-and-roles.md).
- **Run lock**. The file `.codelead/run.lock` that lets only one unattended run work in a folder at a time. See [Unattended runs and the run lock](unattended-runs.md).
- **Run report**. The page `.codelead/run-report.html` that shows a new app's build, one row per increment, and refreshes while the build runs. See [Tutorial: build a new app](../tutorials/build-a-new-app.md).

## S

- **Scope**. A part of the repository with its own build and tests. CodeLead validates the scope of the files that changed. See [Validation and the gate](validation-and-the-gate.md).
- **Seat**. One person who may use CodeLead on a licensed project. Seats come in blocks of 10. See [Licence and plans](licence-and-plans.md).
- **Seed data**. Sample rows for an empty database, so that an application's screens have something to show. See [The behaviour register and seed data](register-and-seed.md).
- **Session**. One conversation with CodeLead about one repository, in the terminal or in the extension's chat. See [Sessions](sessions.md).
- **Session log**. The record of a session in `.codeleadsessions/<session id>/`: what you typed, each model call and each result. See [Sessions](sessions.md).
- **Skeleton**. The first increment of a new app: a shell that builds and runs, with no features yet. See [`/build-incremental`](../chat-commands/build-incremental.md).
- **Spec**. A file that describes, in plain words, the steps of a journey and what each step must show. See [`codelead spec`](../cli/codelead-spec.md).
- **Stage**. One of the eight steps of the modernization program, such as `read` or `pins`. See [The eight stages](../modernization/the-eight-stages.md).
- **Stop**. A run that ends before its job is done. A stop always says why and what you can do next. See [Results and stops](results-and-stops.md).
- **Stop file**. A file whose presence asks an unattended run to stop after its current part, for example `.codelead/STOP`. See [Unattended runs and the run lock](unattended-runs.md).
- **Strategy**. How much the preflight does, from no questions to a deep analysis: `auto`, `fast`, `clarify`, `architect` or `deep`. See [Strategies](strategies.md).
- **Structural route**. The PKB search's way into the code. A question that asks where, which file or how do I takes it, and so does a name in the code. See [`codelead pkb search`](../cli/codelead-pkb-search.md#the-structural-route).
- **System Model**. What CodeLead reads from an old application's sources: its entry points, its rules and its data, with no model call. See [The System Model](../modernization/system-model.md).

## T

- **Template**. A starting folder for a new app: an `npm create` package, a folder, a git repository, or a name in the configuration. See [Tutorial: start a new app from a template](../tutorials/start-from-a-template.md).
- **Template manifest**. The file `codelead-template.json` in which a template says what it is: its stack, its scripts, where pages go. See [Template manifest](../reference/template-manifest.md).
- **Transform**. A change of a known kind that CodeLead applies to old code, for example moving the data access into its own library. See [Transforms](../modernization/transforms.md).

## U

- **Unattended run**. A run that nobody watches. It gives each question its recommended answer, records each answer and stops at its caps. It stops at a step that is hard to undo. See [Unattended runs and the run lock](unattended-runs.md).

## V

- **Validation**. Running the gate: your project's build, tests and other checks. See [Validation and the gate](validation-and-the-gate.md).
- **Validation command**. One command of the gate, declared in `.codelead/validation.json` or detected from the project's files. See [Validation and the gate](validation-and-the-gate.md).

## W

- **Walk, the**. CodeLead follows journeys on the running application as a new user would, and reports each step. See [The walk and parity](../modernization/walk-and-parity.md).
- **Way on, the**. The part of a result that says what to do next, for example `/repair`. See [Results and stops](results-and-stops.md).

## Related content

- [How CodeLead works](how-codelead-works.md)
- [Concepts overview](index.md)
- [Command-line reference](../cli/index.md)
- [Chat command reference](../chat-commands/index.md)
- [Files and folders](../reference/files-and-folders.md)
