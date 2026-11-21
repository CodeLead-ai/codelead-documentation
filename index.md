---
title: CodeLead documentation
description: Learn how to plan, build, change and modernize software with CodeLead, with evidence at every step.
kind: overview
area: get-started
date: 2026-11-20
---

# CodeLead documentation

CodeLead turns AI coding agents into an engineering workflow that you control. Every task is planned, limited to a scope, done one approved step at a time, checked, repaired when a check fails, and recorded as evidence. These pages explain how to use it.

**In short**
- CodeLead plans each change, shows it to you, checks it, and records what happened.
- You can use it in Visual Studio Code, in a terminal, or both.
- It builds new applications, changes existing code, and modernizes old applications.
- Your code stays on your machine unless you allow a remote model.

## Choose where to start

| You are | Start here | Then |
|---|---|---|
| **New to CodeLead** | [Install CodeLead](get-started/install.md) | [Quickstart: CodeLead in VS Code](get-started/quickstart-vscode.md) |
| **Already a user of coding agents** | [How CodeLead works](concepts/how-codelead-works.md) | [Quickstart: change existing code](get-started/quickstart-existing-code.md) |
| **A team lead or an architect** | [Evidence records](concepts/evidence-records.md) | [Modernization overview](modernization/index.md) |

Not sure? Read [Choose your path](get-started/choose-your-path.md).

## Get started

- [Install CodeLead](get-started/install.md)
- [Quickstart: build a new app](get-started/quickstart-new-app.md)
- [Quickstart: change existing code](get-started/quickstart-existing-code.md)
- [Quickstart: CodeLead in VS Code](get-started/quickstart-vscode.md)

## Tutorials

Step-by-step lessons. Each one works from start to end on a sample.

- [Build a new app, step by step](tutorials/build-a-new-app.md)
- [Change existing code with evidence](tutorials/change-existing-code.md)
- [Start a new app from a template](tutorials/start-from-a-template.md)
- [Modernize a .NET Framework application](tutorials/modernize-a-dotnet-framework-app.md)
- [Record journeys with the Analyst](tutorials/record-journeys-with-the-analyst.md)
- [Work with agents on the board](tutorials/work-with-agents-on-the-board.md)

## Concepts

The ideas behind the product. Read these to understand why CodeLead does what it does.

- [How CodeLead works](concepts/how-codelead-works.md)
- [Requests, plans and patches](concepts/requests-plans-and-patches.md)
- [Validation and the gate](concepts/validation-and-the-gate.md)
- [Evidence records](concepts/evidence-records.md)
- [Models and roles](concepts/models-and-roles.md)
- [All concepts](concepts/index.md) and the [glossary](concepts/glossary.md)

## How-to guides

Short answers to "how do I…".

- [Set up local models](how-to/set-up-local-models.md)
- [Set up remote models](how-to/set-up-remote-models.md)
- [Write your conventions](how-to/write-conventions.md)
- [Fix a stop](how-to/fix-a-stop.md)
- [All how-to guides](how-to/index.md)

## The products

### CodeLead for VS Code

The extension is the easiest way to use CodeLead: projects, a chat, the board and the modernization wizard inside the editor.

- [Overview](extension/index.md)
- [A tour of the extension](extension/tour.md)
- [The chat](extension/chat.md)
- [The Modernize wizard](extension/modernize-wizard.md)

### Modernization

Move an old application to a current platform, with its behaviour recorded first.

- [Overview](modernization/index.md)
- [The eight stages](modernization/the-eight-stages.md)
- [Pins](modernization/pins.md)
- [The rebuild](modernization/rebuild.md)

### Pinboard

The board between you and your coding agents: cards with threads, kept as files in your repository.

- [Overview](pinboard/index.md)
- [Use the board](pinboard/use-the-board.md)
- [Work with agents](pinboard/work-with-agents.md)

### Analyst and Explorer

Document a running web application from its screens, without its source code.

- [Overview](analyst/index.md)
- [Explore an application](analyst/explore-an-application.md)
- [Record a journey](analyst/record-a-journey.md)

## Reference

Exact facts: every command, every option, every file.

- [Command-line reference](cli/index.md): `codelead init`, `codelead doctor`, `codelead modernize`, and the rest
- [Chat command reference](chat-commands/index.md): `/implement`, `/validate`, `/repair`, and the rest
- [Board commands](pinboard/cli/index.md)
- [Analyst commands](analyst/index.md#reference)
- [Configuration file](reference/configuration.md) and [environment variables](reference/environment-variables.md)
- [Files and folders](reference/files-and-folders.md)
- [Messages and what to do](reference/messages.md)
- [All reference pages](reference/index.md)
