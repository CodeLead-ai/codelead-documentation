---
title: codelead register
description: Write the behaviour register of an old application, every rule and behaviour that was found, with the check that the new application must pass.
kind: reference
area: cli
date: 2026-11-20
---

# codelead register

**Applies to:** CodeLead 0.1 and later

`codelead register` writes the **behaviour register** of an old application. The register lists every business rule and behaviour that CodeLead found, one line each. Each line has the check that a walk of the new application must pass. The command calls no model.

## Name

`codelead register` - Writes the behaviour register.

## Synopsis

```console
codelead register --model <SYSTEM_MODEL> --out <FOLDER>
                  [--observed <FOLDER>]... [--journeys <JOURNEYS>]... [--walk <WALK>]...
                  [--dispositions <FILE>]
codelead register -h|--help
```

## Description

A rebuild is finished when the new application does everything that the old one did. The register is the list of "everything". It joins three sources:

- **Read**: what the code does, from the [System Model](../modernization/system-model.md) that [`codelead model`](codelead-model.md) writes.
- **Observed by the explorer**: what the [Analyst's Explorer](../analyst/explore-an-application.md) saw when it crawled the running application.
- **Observed by the recorder**: what a person showed while the [Analyst recorded a journey](../analyst/record-a-journey.md).

Each line of the register has:

- a **kind**: validation, refusal, access, effect, navigation, data precondition, clock rule, "story can do", "story can see", or finding;
- its **marks**: read, observed by the explorer, observed by the recorder, each with its evidence;
- a **confidence** from the marks: `confirmed` (read and observed), `read only` or `observed only`;
- an **acceptance check**: what a walk of the new application must show;
- a **status** from the walks of the new application: pass, fail, blocked or not covered.

> [!NOTE]
> **New to this?** A *journey* is a path that a person takes through the application, for example "create a product". A *walk* follows journeys on the running application, as a new user would. See [The walk and parity](../modernization/walk-and-parity.md).

### Dropped features

A feature that the plan drops by a disposition (a recorded decision) is marked with its reason, and asked nothing. The dispositions come from the model's gate, or from the file that `--dispositions` names.

> [!TIP]
> **For experienced users:** the register is `codelead.register/1`. The same inputs write the same bytes, so the register can be committed and compared.

## Options

- **`--model <SYSTEM_MODEL>`**

  Required. The System Model of the old application, `system-model.json`. [`codelead model`](codelead-model.md) writes it.

- **`--observed <FOLDER>`**

  A folder of CodeLead Analyst's observed files. CodeLead reads the ones it holds: `rules.observed.json`, `facts.observed.json`, `stories.json`, `sitemap.json`. Give it again for more folders.

- **`--journeys <JOURNEYS>`**

  A journeys spec that the walks walked. The `covers` of its steps name the register lines that they cover. Give it again for more specs.

- **`--walk <WALK>`**

  A walk's report of the new application, `walk.json`. [`codelead walk`](codelead-walk.md) writes it. It gives the status of the lines that its steps cover. Give it again for more reports.

- **`--dispositions <FILE>`**

  The old repository's dispositions, `docs/codelead/rebuild/extraction-dispositions.json`. The lines of a feature that the plan drops are marked with the reason. The default is the dispositions of the model's gate.

- **`--out <FOLDER>`**

  Required. The folder that `register.json` and `register.md` are written to. CodeLead makes it when it is missing.

## Examples

- Write the register of the Basic Store sample from its System Model alone:

  ```console
  codelead register --model docs/codelead/rebuild/system-model.json --out docs/codelead/register
  ```

  ```output
  Register of webapp: 75 lines and 6 findings; of these 81, 0 confirmed, 81 read only, 0 observed only.
  The new application: 0 pass, 0 fail, 0 blocked, 81 not covered.
  Written: <your project>/docs/codelead/register/register.json
  Written: <your project>/docs/codelead/register/register.md
  ```

- Read the summary at the top of `register.md`:

  ```output
  # Behaviour register of webapp
  …
  | Kind | Lines | read | observed-explorer | observed-recorder |
  |---|---:|---:|---:|---:|
  | validation | 0 | 0 | 0 | 0 |
  | refusal | 0 | 0 | 0 | 0 |
  | access | 6 | 6 | 0 | 0 |
  | effect | 8 | 8 | 0 | 0 |
  | navigation | 40 | 40 | 0 | 0 |
  | data precondition | 1 | 1 | 0 | 0 |
  | clock rule | 0 | 0 | 0 | 0 |
  | story can do | 20 | 20 | 0 | 0 |
  | story can see | 0 | 0 | 0 | 0 |
  | finding | 6 | 6 | 0 | 0 |
  | **all** | **81** | **81** | **0** | **0** |
  …
  ```

- Add what the Analyst observed, and the status from a walk of the new application:

  ```console
  codelead register --model docs/codelead/rebuild/system-model.json --observed analyst/observed --journeys ../basic-store-rebuild/docs/codelead/rebuild/journeys.json --walk ../basic-store-rebuild/docs/codelead/parity/walk.json --out docs/codelead/register
  ```

  <!-- described, not captured -->
  ```output
  Register of webapp: … lines and … findings; of these …, … confirmed, … read only, … observed only.
  The new application: … pass, … fail, … blocked, … not covered.
  Written: …/register.json
  Written: …/register.md
  ```

- Leave out a required option. Nothing is written, and the exit code is 2:

  ```console
  codelead register --out docs/codelead/register
  ```

  ```output
  register needs --model <system-model.json> and --out <folder>. Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Both files were written. |
| `2` | Nothing was written: an input is missing or cannot be read, a file has another schema, or an option was refused. |

## Files

| File | What `register` does with it |
|---|---|
| The `--model`, `--observed`, `--journeys`, `--walk` and `--dispositions` files | Reads them. |
| `<out>/register.json` | Writes the register, `codelead.register/1`. |
| `<out>/register.md` | Writes the register for people: a summary, then one table for each feature. |

## See also

- [The System Model](../modernization/system-model.md)
- [The walk and parity](../modernization/walk-and-parity.md)
- [`codelead model`](codelead-model.md): writes the System Model that the register reads.
- [`codelead walk`](codelead-walk.md): writes the walk reports that give the status.
- [Files the Analyst writes](../analyst/files.md): the observed files.
- [The audit pack](../modernization/audit-pack.md)
