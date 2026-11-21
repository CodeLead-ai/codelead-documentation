---
title: The behaviour register and seed data
description: Learn how the behaviour register lists everything an old application does with a check for each line, and how seed data fills an empty database for the walks.
kind: concept
area: concepts
date: 2026-11-20
---

# The behaviour register and seed data

The behaviour register and seed data belong to a modernization: the move of an old application to a new platform. The **behaviour register** lists every rule and behaviour that CodeLead found in the old application, one line each. Each line has the check that the new application must pass. **Seed data** is a script of sample rows for a database that has its tables and no data. With it, the screens have something to show when CodeLead walks them.

**In short**
- The register joins what CodeLead read in the code with what the Analyst (CodeLead's recorder of a running application) saw.
- Each line has a kind, its marks of evidence, a confidence and an acceptance check. Its status comes from the walks: CodeLead following the new application's journeys.
- [`codelead register`](../cli/codelead-register.md) writes it, and [`codelead seed`](../cli/codelead-seed.md) writes the seed script. Neither calls a model.
- You run the seed script yourself. CodeLead never connects to the database.

## Why it exists

A [rebuild](../modernization/rebuild.md) is a new application that replaces the old one. It is finished when it does everything that the old one did. "Everything" must be a list, or nobody can say when the work is done. The register is that list, and each line says how it is checked. A team lead reads in one file what is proven, what failed and what nobody has checked yet.

An old application often arrives with its database's definition and no data. Then most screens are empty, and a [walk](../modernization/walk-and-parity.md) (CodeLead following journeys on the running application, as a new user would) cannot check them. Seed data fills the tables with rows that fit the schema.

## How it works

### The register

`codelead register` reads up to three sources:

- **Read**: what the code does, from the [System Model](../modernization/system-model.md) that [`codelead model`](../cli/codelead-model.md) writes.
- **Observed by the explorer**: what the Analyst's Explorer saw when it crawled the running application.
- **Observed by the recorder**: what a person showed while the [Analyst](../analyst/index.md) recorded a journey.

Each line gets a mark for each source that shows it. The marks give the confidence: `confirmed` (read and observed), `read only` or `observed only`. The walks of the new application give the status: pass, fail, blocked or not covered.

A real run on the Basic Store sample, from its System Model alone:

```console
codelead register --model docs/codelead/rebuild/system-model.json --out docs/codelead/register
```

```output
Register of webapp: 75 lines and 6 findings; of these 81, 0 confirmed, 81 read only, 0 observed only.
The new application: 0 pass, 0 fail, 0 blocked, 81 not covered.
Written: <your project>/docs/codelead/register/register.json
Written: <your project>/docs/codelead/register/register.md
```

Every line is "read only" here, because nothing was observed yet. Every line is "not covered", because no walk of a new application was given.

### Seed data

`codelead seed` reads the schema that `codelead model` wrote, and the System Model's people, roles and rules that read a clock. It writes three files: the T-SQL script, a markdown file that says what each table holds and why, and a keys file. A table gets rows only when something asks for them. That is the `--rows` option, a rule that reads a clock, a seeded row that points at the table, or the people.

The script is safe to run again. It runs in one transaction. It first deletes the rows that it inserted before, then inserts, then counts each table. It never truncates or drops anything.

> [!NOTE]
> **For team leads:** the register is the list a reviewer signs off. Its `register.md` starts with a summary by kind, then one table for each feature, with the evidence of each line.

## What it is not

- **Not a model's opinion**. Both commands are code. The same inputs write the same bytes, so you can commit the files and compare them.
- **Not a test suite**. The register states the check of each line. The walks run the checks, and the register reads their reports.
- **Not real data**. Seed rows are plausible, not copied. CodeLead never reads or writes your database: you run the script where you choose.

## Details

- **Register files**. `<out>/register.json` (schema `codelead.register/1`) and `<out>/register.md`.
- **Line kinds**. Validation, refusal, access, effect, navigation, data precondition, clock rule, "story can do", "story can see" and finding.
- **Register inputs**. `--model` (required), `--observed <folder>` (the Analyst's `rules.observed.json`, `facts.observed.json`, `stories.json`, `sitemap.json`), `--journeys`, `--walk` and `--dispositions`. A feature that the plan drops by a recorded decision is marked with its reason, and asked nothing.
- **Seed files**. `<out>`, `<out>.md` and `<out>.keys.json`. The tag `codelead-seed` marks the rows, in a column that the code never reads.
- **Seed days**. A run covers `--days` days (the default is 30), ending the day before the script runs, by the database server's clock. The day of the run is left to the walk.
- **Exit codes**. `codelead register`: 0 written, 2 nothing written. `codelead seed`: 0 written and complete, 1 written with something not seeded, 2 nothing written.
- **A rebuild's sample data**. [`codelead rebuild init`](../cli/codelead-rebuild-init.md) `--database sample-seed` sets up a script that resets the sample data before the walks.

## Related content

- [`codelead register`](../cli/codelead-register.md) and [`codelead seed`](../cli/codelead-seed.md)
- [`codelead model`](../cli/codelead-model.md) and [`codelead walk`](../cli/codelead-walk.md)
- [`codelead rebuild init`](../cli/codelead-rebuild-init.md)
- [The System Model](../modernization/system-model.md)
- [The walk and parity](../modernization/walk-and-parity.md)
- [The rebuild](../modernization/rebuild.md)
- [Files the Analyst writes](../analyst/files.md)
- [The audit pack](../modernization/audit-pack.md)
- [Glossary](glossary.md)
