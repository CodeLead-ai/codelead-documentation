---
title: Choose your path
description: Three reading paths, for a reader new to CodeLead, a user of other coding agents and a team lead, with five pages each and what you have at the end.
kind: overview
area: get-started
date: 2026-11-20
---

# Choose your path

CodeLead has many pages. You do not need all of them to start. Choose the path that fits you below. Each path has five pages, in order, and says what you will have done or know at its end.

**In short**
- **New to CodeLead:** install it, use it in VS Code, and learn the loop that every change goes through.
- **Already a user of coding agents:** learn what CodeLead adds around the model, then change real code with evidence.
- **A team lead:** learn what CodeLead records, how it decides that a change is done, and how it protects your code.

## I am new to CodeLead

You know git, a terminal and your own stack. You have not used CodeLead, and perhaps no AI coding tool either.

1. [Install CodeLead](install.md): install it, connect a model, and read `codelead doctor`.
2. [Quickstart: CodeLead in VS Code](quickstart-vscode.md): prepare a folder, ask a question, make one change in the chat.
3. [How CodeLead works](../concepts/how-codelead-works.md): the loop from your request to the evidence record.
4. [Quickstart: build a new app](quickstart-new-app.md): a small app from one sentence.
5. [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md): what the capital letter means, and when Enter is safe.

**At the end**, CodeLead runs on your machine with a model. You have made one change and built one app. You know when to press Enter and when to read first.

## I already use coding agents

You use tools that let a model edit code. You want to know what CodeLead does differently, and whether you can trust its results.

1. [How CodeLead works](../concepts/how-codelead-works.md): what CodeLead checks, and what it is not.
2. [Quickstart: change existing code](quickstart-existing-code.md): one `/implement` from the request to the evidence record.
3. [The patch check](../concepts/the-patch-check.md): what is checked before anything changes, with no model.
4. [Repair](../concepts/repair.md): how a failed gate is fixed, and why the loop stops when it goes in circles.
5. [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md): conventions, a failing gate, a repair round and a pull request.

**At the end**, you have changed real code with CodeLead and repaired a failure. You know where CodeLead's checks stop and your own build and tests take over. You can hand a change to a reviewer with its evidence.

## I lead a team

You decide which tools your team uses. You care about what leaves the building, what is recorded, and who can check the work later.

1. [How CodeLead works](../concepts/how-codelead-works.md): the loop, local-first, and what CodeLead is not.
2. [Evidence records](../concepts/evidence-records.md): the file that every run leaves, field by field.
3. [Validation and the gate](../concepts/validation-and-the-gate.md): your team's definition of done, in `.codelead/validation.json`.
4. [Privacy and remote models](../concepts/privacy-and-remote-models.md): what leaves the machine, when, and with whose consent.
5. [Modernization overview](../modernization/index.md): how CodeLead moves an old application to a current platform, with its behaviour recorded first.

**At the end**, you know what CodeLead records and where. You can set the gate that decides when a change is done. You know which settings keep code on your premises, and what the modernization program does.

## Next steps

- [Install CodeLead](install.md)
- [Concepts](../concepts/index.md)
- [Glossary](../concepts/glossary.md)
