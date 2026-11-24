---
title: codelead rebuild
description: Make a rebuild of an old application, run its plan, and print the plan's parts, lanes and critical path.
kind: reference
area: cli
date: 2026-11-20
---

# codelead rebuild

**Applies to:** CodeLead 0.1 and later

`codelead rebuild` works with a **rebuild**: a new application, made beside the old one, that does what the old one did. Its subcommands make the rebuild and run its plan. With `--plan`, the command prints the plan.

## Name

`codelead rebuild` - Makes a rebuild (`init`), runs its plan (`run`), and prints the plan (`--plan`).

## Synopsis

```console
codelead rebuild init --out <FOLDER> [<OPTIONS>]
codelead rebuild run [<OPTIONS>]
codelead rebuild journeys [--target <REPO>] [--refresh]
codelead rebuild --plan [--target <REPO>] [--scope <PREFIXES>] [--json]
codelead rebuild -h|--help
```

## Description

A [rebuild](../modernization/rebuild.md) does not change the old code. CodeLead reads the old application into its [System Model](../modernization/system-model.md), then builds a new application part by part, and checks each part against the old behaviour.

| Subcommand | What it does |
|---|---|
| [`init`](codelead-rebuild-init.md) | Makes the rebuild repository beside the old application: the template, the manifest, the journeys, the plan and a dry run. |
| [`run`](codelead-rebuild-run.md) | Runs the plan's queue without a person: each part composed, validated, walked and judged. |
| [`journeys`](codelead-rebuild-journeys.md) | Compares the rebuild's copy of the journeys with the System Model's draft. With `--refresh`, it replaces the copy's drafted journeys with the draft's. |

### The overall flow

1. In the old application, run [`codelead model`](codelead-model.md). It writes the System Model and the feature map.
2. In the old application, run [`codelead rebuild init`](codelead-rebuild-init.md). It makes the rebuild folder and the plan.
3. In the rebuild, run `codelead rebuild --plan` to read the plan, and [`codelead rebuild run --dry-run`](codelead-rebuild-run.md) to check it.
4. In the rebuild, run [`codelead rebuild run`](codelead-rebuild-run.md).
5. Check the result with [`codelead walk`](codelead-walk.md), [`codelead parity --rebuild`](codelead-parity.md) and [`codelead rebuild-report`](codelead-rebuild-report.md).

The [modernization program](codelead-modernize.md) runs steps 1, 2, 4 and 5 as its stages `model`, `rebuild-init` and `rebuild`.

> [!NOTE]
> **New to this?** The plan cuts the rebuild into **parts**. A part is an endpoint of the new API, a screen of the new web application, or a prerequisite. Parts are grouped in **lanes**, one per area of the application, after the **foundations**.

### The plan

`codelead rebuild --plan` prints the plan as it stands, with no model call. It writes nothing and takes no lock, so it answers while a run works in the clone. It prints:

- the parts in order, each with its kind, its lane and its state (planned, dropped, accepted, parked);
- the lanes: the foundations, then one lane per area, each with its parts and its chain. A chain is a lane's longest line of parts that wait on each other;
- the **critical path**: the longest chain of the whole plan;
- the width per level, and the parallel preference.

`--scope` limits the plan to some features, by id or by prefix.

> [!TIP]
> **For experienced users:** the plan is computed as `/modernize --rebuild-plan` computes it, from `docs/codelead/rebuild/rebuild.json` and the old application's feature map. `--json` prints `project`, `parts` (`id`, `state`, `lane`) and `lanes`. See [Rebuild manifest](../reference/rebuild-manifest.md).

Without `--plan` and without a subcommand, `codelead rebuild` prints its help.

`codelead rebuild` is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, `--plan`, `run` and `journeys` are refused before anything runs, with exit code 1. `init` is refused with exit code 2. The help is free. A rebuild counts as on the Pro plan when its own folder has the plan, or the old application that its manifest names has it. See [Licence and plans](../concepts/licence-and-plans.md).

## Options

These options belong to `codelead rebuild --plan`. Each subcommand has its own options.

- **`--plan`**

  Prints the plan's parts in order, then its lanes (the foundations, a lane per area, each lane's parts and chain) and its critical path. Nothing is written.

- **`--target <REPO>`**

  The rebuild repository. The default is the current folder.

- **`--scope <PREFIXES>`**

  The plan's scope: feature ids or prefixes, separated by commas.

- **`--json`**

  Prints the plan's lanes as JSON.

## Examples

- Print the plan of the rebuild of the Basic Store sample, in the rebuild's folder:

  ```console
  codelead rebuild --plan
  ```

  ```output
  Rebuild plan of webapp: 80 parts, 36 blockers. Nothing is written: /modernize --rebuild-plan writes the plan's files.
  Parts, in order:
    1. identity-seam (prerequisite; lane foundations): planned
    2. shell:startup (the shell's start-up; lane foundations): dropped
    3. shell:menu (the shell's menu; lane foundations): planned
    4. shell:channel (the shell's channel; lane foundations): dropped
    5. clients.create-get:screen (screen; lane clients): planned
    6. clients.index:endpoint (endpoint; lane clients): planned
  …
  Lanes: foundations (4 parts) then 5 lanes: clients (14 parts, chain 4), orders (15 parts, chain 4), products (14 parts, chain 4), home (6 parts, chain 2), manage (27 parts, chain 5); critical path: identity-seam, shell:menu, manage.index:screen, manage.manage-logins:screen, manage.link-login:endpoint, manage.link-login:screen.
  …
    Assign in this order: foundations; then clients, orders, products, home, manage (at once).
    Critical path (6 parts): identity-seam → shell:menu → manage.index:screen → manage.manage-logins:screen → manage.link-login:endpoint → manage.link-login:screen.
    Width per level: 27, 20, 11, 10, 10, 2.
  …
  ```

- Print only the parts of the products area, and the foundations that it needs:

  ```console
  codelead rebuild --plan --scope products
  ```

  ```output
  Rebuild plan of webapp (scope products): 18 parts, 36 blockers. Nothing is written: /modernize --rebuild-plan writes the plan's files.
  Parts, in order:
    1. identity-seam (prerequisite; lane foundations): planned
    2. shell:startup (the shell's start-up; lane foundations): dropped
    3. shell:menu (the shell's menu; lane foundations): planned
    4. shell:channel (the shell's channel; lane foundations): dropped
    5. products.create-get:screen (screen; lane products): planned
    6. products.index:endpoint (endpoint; lane products): planned
  …
  ```

- Print the plan of a rebuild in another folder, as JSON. Here the command runs in the old application:

  ```console
  codelead rebuild --plan --target ../basic-store-rebuild --json
  ```

  ```output
  {
    "project": "webapp",
    "parts": [
      {
        "id": "identity-seam",
        "state": "planned",
        "lane": "foundations"
      },
      {
        "id": "shell:startup",
        "state": "dropped",
        "lane": "foundations"
      },
  …
  ```

  After `parts` comes `lanes`: the foundations' parts, then each lane with its `parts`, its `chain` and the lanes it `waitsOn`.

- Run it in a folder that is not a rebuild:

  ```console
  codelead rebuild --plan
  ```

  ```output
  No rebuild manifest at docs/codelead/rebuild/rebuild.json: the plan's commands run in the rebuild repository, whose manifest names the old repository, its feature map and the targets. Nothing is planned.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The plan was printed. |
| `1` | The rebuild cannot be read: no manifest, or the old application's files are missing. Also a folder without the Pro plan. |

## Files

`codelead rebuild --plan` writes nothing. It reads `docs/codelead/rebuild/rebuild.json` in the rebuild, and the old application's feature map and System Model that the manifest names.

## See also

- [The rebuild](../modernization/rebuild.md)
- [`codelead rebuild init`](codelead-rebuild-init.md) and [`codelead rebuild run`](codelead-rebuild-run.md)
- [`codelead rebuild journeys`](codelead-rebuild-journeys.md): refresh the rebuild's journeys from the System Model's draft.
- [Rebuild manifest](../reference/rebuild-manifest.md)
- [`codelead modernize`](codelead-modernize.md): the program that runs the rebuild as a stage.
- [`/modernize`](../chat-commands/modernize.md): `--rebuild-plan` writes the plan's files.
- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)
