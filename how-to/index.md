---
title: How-to guides
description: Short guides for one task each, grouped by task - models, a project's rules, runs and stops, sessions, modernization and upkeep.
kind: overview
area: how-to
date: 2026-11-20
---

# How-to guides

A how-to guide answers one question, such as "how do I give repairs a stronger model?". Each guide is a few numbered steps with the real commands and their output. It assumes that you have CodeLead installed. To learn CodeLead from the start, follow a [tutorial](../tutorials/index.md) first.

**In short**
- The guides are grouped by task: models, a project's rules, runs and stops, sessions, modernization and upkeep.
- Each guide ends with a check that it worked, and links to the reference pages.
- When a command stopped and you do not know why, start with [Fix a stop](fix-a-stop.md).

## Where to start

| You are | Start here |
|---|---|
| **New to CodeLead** | [Set up local models](set-up-local-models.md) or [Set up remote models](set-up-remote-models.md), then [Fix a stop](fix-a-stop.md). |
| **Already a user of coding agents** | [Use a stronger model for repairs](use-a-stronger-model-for-repairs.md), [Write your conventions](write-conventions.md) and [Declare validation commands](declare-validation-commands.md). |
| **A team lead** | [Declare validation commands](declare-validation-commands.md), [Export a session](export-a-session.md) and [Waive a security finding](waive-a-security-finding.md). |

## Models

- [Set up local models](set-up-local-models.md): a model server on your machine or your network, its address, the model id and the context length.
- [Set up remote models](set-up-remote-models.md): a provider on the internet, its key, the consent, and what each call costs.
- [Use a stronger model for repairs](use-a-stronger-model-for-repairs.md): the repair role on another model, and the two switches for new apps.

## A project's rules

- [Write your conventions](write-conventions.md): the rules of your team, in plain words, for every change.
- [Declare validation commands](declare-validation-commands.md): the build and test commands that decide that a change is done.
- [Build .NET Framework projects on Windows](build-on-windows.md): Visual Studio's MSBuild for an old .NET Framework project.

## Runs and stops

- [Fix a stop](fix-a-stop.md): what each kind of stop means, and what to do about it.
- [Stop and resume work](stop-and-resume.md): stop a command, and continue it later where it stopped.

## Sessions

- [Export a session](export-a-session.md): a zip of a session's log, to read later or to give to a person who helps you.
- [Read a session in the Session Log Viewer](read-a-session.md): the session folder, its events, and the views that show them.

## Modernization

- [Narrate the code before you pin it](narrate-before-pins.md): the narration that gives the pins their facts.
- [Explain a finding of the extraction gate](explain-a-gate-finding.md): a reason for a hole in the System Model.
- [Waive a security finding](waive-a-security-finding.md): decide a finding that changes who may do what.

## Upkeep

- [Update CodeLead](update-codelead.md): a newer CodeLead, a newer extension, and the files to refresh in each project.

## Related areas

- [Tutorials](../tutorials/index.md)
- [Concepts](../concepts/index.md)
- [Command-line reference](../cli/index.md) and [chat command reference](../chat-commands/index.md)
- [Messages and what to do](../reference/messages.md)
