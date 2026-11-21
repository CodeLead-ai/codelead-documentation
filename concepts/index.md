---
title: Concepts
description: The ideas behind CodeLead, grouped, with one line each and a reading order for a reader who is new to the product.
kind: overview
area: concepts
date: 2026-11-20
---

# Concepts

These pages explain the ideas behind CodeLead: why it works the way it does, and what each part is for. They do not list steps. For steps, see the [tutorials](../tutorials/index.md) and the [how-to guides](../how-to/index.md).

**In short**
- Read [How CodeLead works](how-codelead-works.md) first. Every other page builds on it.
- The pages are grouped by the question that they answer.
- Every page ends with the commands that touch its idea, and every term is in the [glossary](glossary.md).

## Where to start

| You are | Read these, in this order |
|---|---|
| **New to CodeLead** | [How CodeLead works](how-codelead-works.md), [Requests, plans and patches](requests-plans-and-patches.md), [The patch check](the-patch-check.md), [Validation and the gate](validation-and-the-gate.md), [Results and stops](results-and-stops.md) |
| **Already a user of coding agents** | [How CodeLead works](how-codelead-works.md), [Repair](repair.md), [Models and roles](models-and-roles.md), [Conventions](conventions.md), [The knowledge base](knowledge-base.md) |
| **A team lead** | [Evidence records](evidence-records.md), [Validation and the gate](validation-and-the-gate.md), [Privacy and remote models](privacy-and-remote-models.md), [Unattended runs and the run lock](unattended-runs.md), [Licence and plans](licence-and-plans.md) |

## The loop of a change

- [How CodeLead works](how-codelead-works.md): the loop that every change goes through, and what CodeLead is not.
- [Requests, plans and patches](requests-plans-and-patches.md): from your words to a plan, and from the plan to a patch.
- [The patch check](the-patch-check.md): what CodeLead checks in a patch before it asks you to apply it.
- [Validation and the gate](validation-and-the-gate.md): how your own build and tests decide that a change is done.
- [Repair](repair.md): small fixes after a failed gate, and why the loop stops when it goes in circles.
- [Evidence records](evidence-records.md): the file that every run leaves, and why a reviewer reads it.

## How CodeLead talks to you

- [Results and stops](results-and-stops.md): how every command ends, and the way on from each common stop.
- [Prompts and recommended answers](prompts-and-recommended-answers.md): the capital letter, the Enter key, and the steps that are hard to undo.
- [Sessions](sessions.md): one conversation with CodeLead, and the log that it keeps.

## Models

- [Models and roles](models-and-roles.md): providers, roles, and a stronger model for repairs.
- [Strategies](strategies.md): how much planning the preflight does, and how `auto` chooses.
- [Privacy and remote models](privacy-and-remote-models.md): what leaves your machine, when, and with whose consent.

## What CodeLead knows about your project

- [The repository map](repository-map.md): the stack, the validation commands and the modules, read from the code.
- [The project library](project-library.md): the index of files, symbols and relationships behind the map and the file choice.
- [Conventions](conventions.md): the rules that you write, and the habits that CodeLead reads from the code.
- [The knowledge base](knowledge-base.md): the encyclopedia, capsules, context packages and capability profiles.

## Larger work

- [Unattended runs and the run lock](unattended-runs.md): runs that nobody watches, their caps, and one run per folder.
- [The behaviour register and seed data](register-and-seed.md): what an old application does, and sample data for its database.
- [Licence and plans](licence-and-plans.md): the licence file, the free and pro plans, and seats.

## Words

- [Glossary](glossary.md): every term of CodeLead in one or two sentences.

## Related areas

- [Get started](../get-started/index.md)
- [Tutorials](../tutorials/index.md)
- [Modernization](../modernization/index.md)
- [Command-line reference](../cli/index.md) and [chat command reference](../chat-commands/index.md)
