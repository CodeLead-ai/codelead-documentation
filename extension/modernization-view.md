---
title: The Modernization view
description: The documents of a modernization in one view - the dossier, the narration, the feature map, the rebuild plan and report, the audit pack, the project documents and the specs - and the button that opens the wizard.
kind: concept
area: extension
date: 2026-11-20
---

# The Modernization view

The **Modernization** view lists the documents that a [modernization](../modernization/index.md) writes under `docs/codelead/`. They are the dossier, the feature map, the rebuild plan and report, and the audit pack. It also lists the project's specs and their last walks. Its title bar has the rocket, **Modernize this application**, which opens the [Modernize wizard](modernize-wizard.md).

**In short**
- A click on a document opens it as a page in a tab. The page renders the Markdown, the feature map as a table, and the rebuild plan with its parts.
- **Specs** lists each spec with the verdict of its last walk: passed, failed, blocked or never walked.
- The view is in the pro plan. On the free plan, it shows a lock row, then the specs only.

![The Modernization view on the pro plan. Its title bar has the rocket, refresh and collapse. The rows are Dossier with 4 documents, Functional narration, Feature map, Rebuild plan, Rebuild report, Audit pack, Catalog, Pinned, and Specs with 2 specs. Each row shows its file.](../media/extension/modernization-pro.png)

## Why it exists

A modernization writes many documents, stage by stage. A developer and a reviewer need to find each one, and see at a glance which specs pass. The view lists the documents that exist, in the order of the stages, and nothing else. See [The eight stages](../modernization/the-eight-stages.md).

## What the view shows

The view lists the projects that hold `docs/codelead/`. With one project, its rows are at the top. Each row appears only when its file exists.

| Row | File | What its page shows |
|---|---|---|
| **Dossier** (a group, `N documents`) | `docs/codelead/dossier/*.md` | One page for each section of the [dossier](../modernization/dossier.md). The rows carry the section's id, such as `D2`, when `dossier-index.json` names it. |
| **Functional narration** | `docs/codelead/dossier/functional-narration.md` | What the application does, in plain words. |
| **Feature map** | `docs/codelead/rebuild/feature-map.md` and `.json` | A table of the features: each controller action, its route, its view, its facts and its tests, in the order of the rebuild. |
| **Rebuild plan** | `docs/codelead/rebuild/rebuild-plan.json` | The counts by state, the blockers, the parts in order with their evidence, and the warnings. See [The rebuild](../modernization/rebuild.md). |
| **Rebuild report** | `.codelead/rebuild-report.html` | The progress report of a rebuild. A click opens the HTML file itself in the editor, not a rendered page. The wizard shows it rendered. |
| **Audit pack** | `docs/codelead/audit-pack/README.md` | The pack's page, and its manifest: every file with its SHA-256. See [The audit pack](../modernization/audit-pack.md). |
| **Catalog**, **Pinned**, **Project log** | `docs/codelead/CATALOG.md`, `PINNED.md`, `PROJECT-LOG.md` | The project documents that [`/log`](../chat-commands/log.md) and [`/pin`](../chat-commands/pin.md) keep. |
| **Specs** | `docs/codelead/specs/` | The specs, their journeys and their walks. See below. |

A click on a row opens its page in a tab. The file icon on the row opens the file in the editor.

### The specs

A spec is a short text that says what a person does in the application and what they see. CodeLead drafts it into a journey and walks it on the running application. See [`codelead spec`](../cli/codelead-spec.md).

**Specs** shows the number of specs, for example `3 specs`. Under it, each spec shows a mark and the words of its last walk:

| Mark | Last walk | Example of the words |
|---|---|---|
| Green check | Every step passed. | `passed · 6/6 steps pass · 2026-11-18` |
| Red cross | A step failed. | `failed · 4/6 steps pass · 2026-11-18` |
| Yellow warning | A step was blocked. | `blocked · 2/6 steps pass · 2026-11-18` |
| Empty circle | The spec was never walked. | `never walked` |

Open a spec to see its files: the spec, its journey (`<name>.journeys.json`) and its last walk in JSON and in Markdown. The screenshots of the walk follow, each with its step. The **Library page** (`docs/codelead/specs/README.md`) and **Coverage** (`coverage.md`) follow the specs when they exist.

> [!NOTE]
> **New to this?** A *walk* is CodeLead using the application in a browser, step by step, as the journey says. Each step passes, fails or is blocked. See [The walk and parity](../modernization/walk-and-parity.md).

## What you can do

- **Start or follow a modernization.** Select the rocket, **Modernize this application**. The wizard opens for the project. See [The Modernize wizard](modernize-wizard.md).
- **Check the specs.** In the **Projects** view, select **…** > **Check the specs**. See [The Projects view](projects-view.md#the-actions-of-the-title-bar).
- **Read a document beside the code.** Select the row, then use the file icon to open the source file in the editor.

## On the free plan

The view shows one row with a lock: "Modernization is in CodeLead's Pro plan". Its tooltip says: `Modernization reads an existing application into a dossier, then plans its rebuild and tracks it page by page.` Under it, the view lists the specs, because the spec walk is in the free plan. The rocket is hidden. See [Licence and plans](../concepts/licence-and-plans.md).

## When the view is empty

A project with no modernization document and no spec shows this text:

```output
The dossier, the feature map and the specs appear here when a modernization or a spec is made. Nothing yet.
```

## Details

- **The dossier's order.** With `docs/codelead/dossier/dossier-index.json`, the pages follow its sections, titled with their id and title. Without it, the view lists the `.md` files of the folder by name.
- **The rebuild's folder.** A rebuild writes its own plan and report in the new application's folder. Open that folder in the window to see them.
- **Refresh.** The view refreshes when CodeLead writes under `docs/codelead/` or `.codelead/`.

## Related content

- [Modernization](../modernization/index.md)
- [The Modernize wizard](modernize-wizard.md)
- [The eight stages](../modernization/the-eight-stages.md)
- [The dossier](../modernization/dossier.md) and [The rebuild](../modernization/rebuild.md)
- [`codelead spec`](../cli/codelead-spec.md)
- [Tutorial: modernize a .NET Framework application](../tutorials/modernize-a-dotnet-framework-app.md)
