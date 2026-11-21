---
title: Reference
description: The formats, files, settings, protocols and codes of CodeLead, one page each, for when you need the exact facts.
kind: overview
area: reference
date: 2026-11-20
---

# Reference

These pages hold the exact facts behind the rest of the documentation. They list the keys of the configuration file, the files that CodeLead writes, the formats that it reads and the codes that it ends with. Each page has real examples. The command pages are in the [command-line reference](../cli/index.md) and the [chat command reference](../chat-commands/index.md).

**In short**
- Look here when you need a key, a field, a file name, a schema or an exit code.
- Each page lists every field of one file or one format, with its type, its default and a real example.
- The pages are grouped by what they describe: settings, the files of a project, the files of a modernization, formats, protocols, and codes.

## Where to start

- **I am new to CodeLead**: start with [Files and folders](files-and-folders.md). It says what appears in your repository after [`codelead init`](../cli/codelead-init.md), and what belongs in git.
- **I already use coding agents**: read [Configuration file](configuration.md) and [Environment variables](environment-variables.md). They say how to point CodeLead at your models.
- **I lead a team**: read [Evidence record format](evidence-record-format.md) and [Exit codes](exit-codes.md). They say what a run leaves behind, and what a script can rely on.

## Configuration

- [Configuration file](configuration.md): every key of `.codelead/config.json`, its type, its default and what reads it.
- [Environment variables](environment-variables.md): every `CODELEAD_*` variable and every key variable, and what each one overrides.

## Files of a project

- [Files and folders](files-and-folders.md): every file and folder that CodeLead writes, who writes it, and whether it belongs in git.
- [Evidence record format](evidence-record-format.md): the record that every run writes, field by field, with its outcomes and its chain.
- [Validation file](validation-file.md): `.codelead/validation.json`, the commands of the gate that you declare.
- [Run file for specification runs](run-file.md): `.codelead/run.json`, how CodeLead starts your application to walk a spec.
- [Specification files](spec-files.md): the specs in plain words under `docs/codelead/specs/`, and the files beside them.
- [Template manifest](template-manifest.md): `codelead-template.json`, what a template for a new app or a rebuild must hold.

## Files of a modernization

- [Modernization state file](modernization-state-file.md): `.codelead/modernization.json`, the stages, their statuses and the answers.
- [Run profile](run-profile.md): how CodeLead starts the old application, and how it starts the rebuilt one for a walk.
- [Rebuild manifest](rebuild-manifest.md): `docs/codelead/rebuild/rebuild.json`, the old application, the targets and the decisions.
- [Rebuild run summary](rebuild-run-summary.md): `.codelead/rebuild-run-<stamp>.json` and `.md`, what an unattended run did.
- [Pin levels file](pin-levels-file.md): `pin-levels.json`, the ledger of facts that each pin level leaves.
- [Extraction dispositions and security waivers](dispositions-and-waivers.md): the reasons a person records for the extraction gate and for security findings.
- [Parity waivers](parity-waivers.md): `docs/codelead/parity-waivers.json`, the page differences that you accept.

## Formats

- [Journeys format](journeys-format.md): `codelead.journeys/1`, the journeys that the walk follows.
- [Evidence record format](evidence-record-format.md): `codelead.evidence/1`.

## Protocols

- [Session protocol](session-io-protocol.md): `codelead.session-io/1`, the JSON lines between CodeLead and an editor such as the extension.

## Codes and messages

- [Messages and what to do](messages.md): the stops and the main refusals, word for word, with the way out of each.
- [Exit codes](exit-codes.md): the exit codes of every command family, and what a script can rely on.
