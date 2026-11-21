---
title: Start a new app from a template
description: Make a small template, describe it in its manifest, bring it into an empty folder with /create-app --template, and build a task manager app on it.
kind: tutorial
area: tutorials
date: 2026-11-20
---

# Start a new app from a template

A template is the starting folder of a new app: the stack, the UI kit and the layout that your team chose. With [`/create-app --template`](../chat-commands/create-app.md), CodeLead brings the template into an empty folder first. Then the plan builds the app on it, and the app keeps your team's shape.

**In short**
- You know the four kinds of template and which ones CodeLead runs without a question.
- You make a small folder template and describe it in its manifest, `codelead-template.json`.
- You build a task manager app on the template.
- You see the question that CodeLead asks before it runs a package that it does not trust.

In this tutorial, you:

> [!div class="checklist"]
> - Learn the four kinds of template
> - Make a small folder template
> - Describe the template in its manifest
> - Bring the template into an empty folder
> - Build the app on the template
> - Read what CodeLead keeps of the template
> - See the trust question for an unlisted package

## Prerequisites

What you must know:

- You know how to open a terminal and run a command.
- You know what the `scripts` of a `package.json` file are, such as `build` and `test`.

What you must have:

- CodeLead installed. See [Install CodeLead](../get-started/install.md).
- Node.js 22 and git.
- A model, for the plan and the build. That is a local model server, or a key for a remote provider. See [Set up local models](../how-to/set-up-local-models.md) or [Set up remote models](../how-to/set-up-remote-models.md). Bringing the template in needs no model.
- About 30 minutes.

## Learn the four kinds of template

You give the template with the option `--template`, before the request. CodeLead reads the one word after the option and decides its kind. See [`/create-app`](../chat-commands/create-app.md).

| Kind | What you type | What CodeLead does |
|---|---|---|
| An `npm create` package | `vite`, `next-app`, `vite@9.3.1`, `@example/create-app` | Runs `npm create <package> .` in the folder. `vite` runs `npm create vite@9.3.1 . -- --template react-ts`. |
| A folder on this machine | `../task-shell`, `./shells/web` | Copies the files, without `.git` and `node_modules`. Then it installs the dependencies with npm, or with the manager that the lockfile names. |
| A git address | `https://github.com/<owner>/<repo>`, `<owner>/<repo>`, `git@…` | Runs `git clone --depth 1` into a temporary folder, removes the clone's `.git`, then copies it as a folder. |
| A name from the configuration | A name under `greenfield.templates` in `.codelead/config.json` | Uses the source that the name points to: one of the three kinds above. |

Four packages are on CodeLead's trusted list: `vite`, `create-vite`, `next-app` and `create-next-app`. CodeLead runs them with no question. Any other package from the npm registry runs only after you answer yes.

> [!NOTE]
> **New to this?** A relative folder starts with `./` or `../`. A bare word is a name from the configuration first, then a package. A word that is one or two letters away from a configured name is refused. So a typing error never runs a package from the registry.

## Make a small folder template

In this tutorial, the template is a small Node.js shell in a folder. A folder template is the easiest kind to try, because it needs no network for the template itself.

1. Create the folder of the template and its `src` folder:

   ```console
   mkdir task-shell
   mkdir task-shell/src
   ```

1. Create the file `task-shell/package.json` with this content:

   ```json
   {
     "name": "task-shell",
     "private": true,
     "version": "0.1.0",
     "type": "module",
     "scripts": {
       "build": "node --check src/main.js",
       "test": "node --test",
       "dev": "node src/main.js"
     }
   }
   ```

   A template must have a `package.json` at its top. The templates of this version are Node.js apps.

1. Create the file `task-shell/src/main.js` with this content:

   ```javascript
   export function appName() {
     return "task-shell";
   }
   ```

## Describe the template in its manifest

The manifest tells CodeLead what the template is, so that the planner uses it well. The file is `codelead-template.json`, at the top of the template, with the schema `codelead.template/1`. See [Template manifest](../reference/template-manifest.md).

1. Create the file `task-shell/codelead-template.json` with this content:

   ```json
   {
     "schema": "codelead.template/1",
     "name": "task-shell",
     "stack": ["node", "javascript"],
     "scripts": { "build": "node --check src/main.js", "test": "node --test", "dev": "node src/main.js" },
     "routes": "src/pages/<route>.js",
     "notes": "One module per feature under src/. Tests beside the module, named <module>.test.js."
   }
   ```

1. Read what each key tells CodeLead. The build reads seven keys:

   | Key | What it says | Example |
   |---|---|---|
   | `name` | The name of the template. | `task-shell` |
   | `stack` | The names of its stack. | `["node", "react", "next", "typescript"]` |
   | `ui` | Its UI kit. | `@mui/material`, `shadcn/ui` |
   | `scripts` | Its scripts, by name. | `build`, `test`, `dev` |
   | `routes` | Where a page goes. | `src/app/<route>/page.tsx` |
   | `api` | Where its API is. | `api/Api (minimal API, /health)` |
   | `notes` | A few sentences for the planner. | `App router. Pages are folders under src/app.` |

   The stack and the UI kit come first in the stack line that the planner reads. The routes, the API and the notes reach the planner as short lines. A key of the wrong shape is left out. A manifest that cannot be read is one warning line, and the build plans without it.

## Bring the template into an empty folder

CodeLead brings a template only into an empty folder. Only CodeLead's own setup may be there: the files that [`codelead init`](../cli/codelead-init.md) writes.

1. Create an empty folder beside the template, and open it:

   ```console
   mkdir task-manager
   cd task-manager
   ```

1. Prepare the folder for CodeLead:

   ```console
   codelead init
   ```

   You see output similar to the following:

   ```output
   ✔ Created <your project>/.codelead/config.json
   <your project>/.gitignore: .env, .env.local, .env.*.local ignored (the key in .env stays out of git).
   Repository map: nothing to read yet (no source files); it is written when the first build finishes.
   ```

1. Start a session:

   ```console
   codelead interactive
   ```

   See [`codelead interactive`](../cli/codelead-interactive.md).

1. Type the request with the template, and press **Enter**:

   ```console
   /create-app --template ../task-shell --auto-after-plan a task manager app with a list of tasks and a done checkbox
   ```

   With `--auto-after-plan`, CodeLead asks the plan's questions, shows the plan, and asks you before it builds.

1. Read the first lines. They come before any model call:

   ```output
   Template: bringing in ../task-shell (a folder on this machine).
   Copied 3 files from ../task-shell.
   Installing the template's dependencies: npm install …
   Template ../task-shell is in the folder: task-shell: Node.js, JavaScript; scripts build, test, dev.
   Create App Plan
   Request: a task manager app with a list of tasks and a done checkbox

   Step 1/4: Analyzing request...
   ```

   The last template line is the stack line. It comes from the manifest and from `package.json`.

> [!TIP]
> When the model server does not answer, the plan stops here: `Failed to create greenfield plan` and `Could not connect to lmstudio at …`. The template stays in the folder. Start the server, then type the same request again. See [Messages and what to do](../reference/messages.md).

## Build the app on the template

The planner knows that a template is in the folder. It asks no question about the stack, and the plan has no scaffold step. The template is only the shell: the request decides what the app is about.

1. Answer the plan's questions. They follow the line `Step 2/4: Asking clarifying questions...`. Press **Enter** at each one to take its default, the answer that CodeLead recommends.

1. Read the plan. You see output similar to the following:

   <!-- described, not captured -->
   ```output
   Step 3/4: Project specification generated.
   Step 4/4: Execution plan generated.
   Project Specification created.
   Execution Plan created.

   Plan summary:
   1. …
   Start the automated build from this plan? [Y/n]
   ```

1. Press **Enter** to start the build.

   CodeLead builds each step of the plan: a patch, [the patch check](../concepts/the-patch-check.md), the apply, and the step's validation with the template's own scripts. CodeLead gives each question its recommended answer.

1. Wait for the completion line:

   <!-- described, not captured -->
   ```output
   Automated build completed on the template ../task-shell: all plan steps are done. Run `npm run dev` in the target folder to open the app.
   Repository map written: 1 module, 9 files; docs/codelead/repository-map.md
   ```

   The line names the template. It gives the `dev` script of the template as the way to open the app.

1. Close the session:

   ```console
   /exit
   ```

> [!TIP]
> To read the plan and build one step at a time, use `--plan` in place of `--auto-after-plan`. Then use [`/plan --approve`](../chat-commands/plan.md) and [`/build-step`](../chat-commands/build-step.md). See [Build a new app, step by step](build-a-new-app.md) for the build in increments.

## Read what CodeLead keeps of the template

CodeLead records the template in `.codelead/template.json`. A request that you send again plans on the template, and does not bring it in twice.

1. Read the record:

   ```console
   cat .codelead/template.json
   ```

   You see output similar to the following, trimmed:

   ```output
   {
     "source": "../task-shell",
     "kind": "folder",
     "resolvedAt": "2026-11-20T05:03:19.833Z",
     "stack": {
       "name": "task-shell",
       "frameworks": [],
       "scripts": {
         "build": "node --check src/main.js",
         "test": "node --test",
         "dev": "node src/main.js"
       },
       "packageManager": "npm",
       "declared": [
         "Node.js",
         "JavaScript"
       ],
       "line": "task-shell: Node.js, JavaScript; scripts build, test, dev"
     },
     "manifest": {
       "name": "task-shell",
   …
   ```

1. Read the line that a second request with the same template prints:

   ```output
   The template ../task-shell is in this folder already (brought in 2026-11-20T05:03:19.833Z): planning on it.
   ```

1. Read what CodeLead refuses, and the way on for each:

   | Title | When | The way on |
   |---|---|---|
   | `The folder is not empty` | The folder holds files that are not CodeLead's setup. | Move those files out, or open an empty folder. |
   | `The folder has another template` | You name a different template in a folder that has one. | Plan on the template that is there, or open an empty folder. |
   | `The template is not a Node app` | The template has no `package.json`. | Give a template with a `package.json` at its top. |
   | `There is no template folder` | The folder that you named does not exist. | Check the path. Relative paths start at the app's folder. |
   | `The template could not be cloned` | git is missing, the network does not answer, or the repository is private or not found. | Fix the cause, or clone it yourself and give its folder. |

> [!TIP]
> **For experienced users:** when `.codelead/template.json` exists, the patch check does not apply its design-token rule. The stylesheet's tokens belong to the UI kit of the template. To name a template once for the team, add it to `greenfield.templates` in `.codelead/config.json`, for example `"task-shell": { "source": "../task-shell" }`. Then type `--template task-shell`.

## See the trust question for an unlisted package

A package that is not on the trusted list is code from the registry that CodeLead has never checked. So CodeLead asks you before it runs one, and it recommends no.

1. Leave the app's folder, and create a second empty folder:

   ```console
   cd ..
   mkdir trust-check
   cd trust-check
   ```

1. Prepare the folder:

   ```console
   codelead init
   ```

1. Start a session:

   ```console
   codelead interactive
   ```

1. Type a request with a package that is not on the list:

   ```console
   /create-app --template @example/create-app --plan a task manager app
   ```

1. Read the question:

   ```output
   Template: bringing in @example/create-app (an npm create package).

   Run `npm create @example/app .` from the registry? It is not on CodeLead's trusted list. [y/N]
   ```

   The capital `N` is the recommended answer, and Enter gives it. See [Prompts and recommended answers](../concepts/prompts-and-recommended-answers.md).

1. Press **Enter**. CodeLead runs nothing:

   ```output
   Template not brought in: You did not confirm the package, so nothing was run (npm create @example/app).
   Next: Name the template again (/create-app --template @example/create-app <request>) and answer y to run the package
   Next: Or give a template on the trusted list (vite, next-app), a folder or a git address
   Next: Or build without a template: /create-app <request>
   ```

   The command's result is `stopped`: your no is a choice, not a failure.

1. Close the session:

   ```console
   /exit
   ```

When nobody can answer, CodeLead does not run the package either. An unattended run is refused with the words `no one can answer here`.

> [!WARNING]
> Answer `y` only for a package that you trust. `npm create` runs the package's code on your machine. Your answer is kept in the session log and in `.codelead/template.json`.

## Clean up

Delete the three folders of this tutorial:

```console
cd ..
rm -rf task-manager trust-check task-shell
```

On Windows, in PowerShell, use `Remove-Item -Recurse -Force task-manager, trust-check, task-shell`.

## Next steps

> [!div class="nextstepaction"]
> [Template manifest](../reference/template-manifest.md)

- [Build a new app, step by step](build-a-new-app.md)
- [`/create-app`](../chat-commands/create-app.md) reference
- [Conventions](../concepts/conventions.md)
- [Configuration file](../reference/configuration.md)
