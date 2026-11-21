---
title: Template manifest
description: What a template for /create-app --template or codelead rebuild init --template must hold, and every key of codelead-template.json, codelead.template/1, with a real manifest.
kind: reference
area: reference
date: 2026-11-20
---

# Template manifest

**Applies to:** CodeLead 0.1 and later

A **template** is a starter app that a new app, or a rebuild, begins from: a folder, a git repository, or an `npm create` package. The **template manifest**, `codelead-template.json` at the template's top, tells CodeLead what the template is: its stack, its scripts, where a page goes, where its API is. The planner uses it to plan well. This page says what a template must hold for each use, and lists every key of the manifest.

## The two uses

`<TEMPLATE>` is a name, a package, a folder or a git address.

| Use | Command | The manifest |
|---|---|---|
| A new app | [`/create-app --template <TEMPLATE>`](../chat-commands/create-app.md) | Optional. Without it, CodeLead plans from `package.json` alone. |
| A rebuild | [`codelead rebuild init --template <TEMPLATE>`](../cli/codelead-rebuild-init.md) | Required. It says where the API and the web app are. |

## Where CodeLead finds a template by name

For `/create-app --template <name>`, a name of `greenfield.templates` in the [configuration file](configuration.md#greenfield). Any other word is an `npm create` package, such as `vite` or `@acme/create-app@2`. A path that starts with `./`, `../` or `/` is a folder. A git address is cloned.

For `codelead rebuild init --template <name>`, CodeLead looks, in this order:

1. `greenfield.templates.<name>` of the configuration.
2. `<name>` in the folder that `CODELEAD_TEMPLATES_DIR` names.
3. `../codelead-templates/<name>`, beside the repository.
4. A template that CodeLead knows by name. `dotnet-next-shadcn`, the default, is one.

`--template` takes a folder or a git address too.

## A template for a new app

| It must | Why |
|---|---|
| Hold `package.json` at its top. | The templates of this version are Node apps. Without it: `The template is not a Node app`. |
| Be a folder beside the app's folder, never around it. | Its files are copied into the app's folder. |

The app's folder must be empty, except for CodeLead's own setup. CodeLead copies the template's files, without `.git`, `node_modules` and CodeLead's session folder, and never over its own setup. Then it installs the dependencies with `npm install`, or with the manager that the lockfile names. A git address is cloned with `git clone --depth 1` first, and its `.git` is removed.

An `npm create` package runs as `npm create <package> . -- <args>`. A package that is not on CodeLead's trusted list runs only after you say yes to one question. Without your yes, nothing runs, and the command stops: `You did not confirm the package, so nothing was run (npm create <package>).`

CodeLead keeps what it brought in, in `.codelead/template.json`. When you send the request again, it plans on the template without bringing it in twice.

## A template for a rebuild

A rebuild template holds an ASP.NET Core API and a web app. `rebuild init` refuses a template that lacks one of these:

| It must hold | What CodeLead reads from it |
|---|---|
| `codelead-template.json` | Where the API and the web app are. |
| A `.csproj` with `Sdk="Microsoft.NET.Sdk.Web"` | The API project: `targets.api.project`. |
| `package.json` at the top, with a `dev` script | The scripts that start and check the app. |
| A web app: a folder with its own `package.json`, named by the `workspaces` of the top `package.json` or by the first folder of `routes` | `targets.web.path`. |

What it reads besides, when it is there:

| Item | What it becomes |
|---|---|
| A second `.csproj` that references `Microsoft.NET.Test.Sdk`, or that ends in `.Tests.csproj` | The API's tests: `targets.api.tests`. |
| The `api:dev` script | How the run profile starts the API. Without it: `dotnet run --project <the API project>`. |
| The `api:build`, `api:test`, `build` and `test` scripts | The gate's commands in the rebuild's `.codelead/validation.json`, in that order. |
| `Properties/launchSettings.json` of the API, its first `http://localhost:<port>` | The API's port. The default is 5080. |
| `-p <port>` or `--port <port>` in the web app's `dev` script | The web app's port. The default is 3000. |
| A path in the manifest's `api`, such as `/health` | The API's ready path. The default is `/`. |
| The manifest's `layout` | Which file each part of the plan goes to. |

`rebuild init` moves a port that the old application already uses, so that both can run at once.

## codelead-template.json

The manifest of `dotnet-next-shadcn`, the default rebuild template:

```json
{
  "schema": "codelead.template/1",
  "name": "dotnet-next-shadcn",
  "stack": ["node", "react", "next", "tailwind", "shadcn-ui", "typescript", "dotnet", "aspnet-core"],
  "ui": "shadcn/ui",
  "scripts": {
    "dev": "npm run dev",
    "build": "npm run build",
    "test": "npm run test",
    "api:build": "npm run api:build"
  },
  "routes": "web/src/app/<route>/page.tsx",
  "api": "api/Api (minimal API, /health)",
  "notes": "The web app is the UI; the API is a starting point. UI-only apps need no API change. shadcn components live in web/src/components/ui; add more with npx shadcn add <name>."
}
```

### Keys

| Key | Type | Limit | What it says |
|---|---|---|---|
| `schema` | string | | `codelead.template/1`. |
| `name` | string | 60 characters | The template's name. |
| `stack` | list of strings, or one comma-separated string | 16 names of 60 characters | The names of its stack, such as `next`, `material-ui`, `dotnet`. The planner names them first. |
| `ui` | string | 60 characters | Its UI kit, such as `@mui/material` or `shadcn/ui`. |
| `scripts` | object | 20 scripts, 400 characters each | Its scripts by name, such as `dev`, `build`, `test`, `api:build`. |
| `routes` | string, or a list joined with `; ` | 400 characters | Where a page goes, such as `src/app/<route>/page.tsx`. |
| `api` | string, or a list | 400 characters | Where its API is, and its ready path, such as `api/Api (minimal API, /health)`. |
| `notes` | string, or a list | 400 characters | What the planner should know, in a few sentences. |
| `layout` | object | | For a rebuild: the shared-file layout. See [layout](#layout). |

Spaces are collapsed and longer text is cut. A key of the wrong shape is left out. Any other key is left out too.

### When the manifest cannot be read

A file larger than 64 KB, a file that is not JSON, and a file that is not a JSON object are not read. For a new app, that is one warning line, never a failure:

<!-- described, not captured -->
```output
The template's codelead-template.json was not read (it is not valid JSON: Unexpected token } in JSON at position 212): CodeLead plans on the template without it.
```

For a rebuild, the same file is a refusal: a rebuild template must say where its API and its web app are.

### layout

The `layout` section tells a rebuild where each part's files go, so that no file is one that every part edits. The rebuild's own `standards/architecture.md` wins over it, key by key.

```json
"layout": {
  "layers": ["core", "shared", "area"],
  "typescript": {
    "root": "src/api",
    "core": "src/api/core",
    "area": "src/api/{area}/{area}.ts",
    "page": "src/api/{area}/{page}.ts",
    "shared": "src/api/shared/{page}.ts",
    "types": "src/api/{area}/{area}.types.ts",
    "test": "{dir}/{stem}.test.ts",
    "mock": "src/api/mock/{area}.ts",
    "mockIndex": "src/api/mock/index.ts",
    "mockRegistration": "import \"./{area}\";",
    "index": "src/api/index.ts"
  },
  "csharp": {
    "controller": "Controllers/{Area}/{Area}Controller.cs",
    "controllerPartial": "Controllers/{Area}/{Area}Controller.{Feature}.cs",
    "service": "Services/{Area}/{Area}Service.cs",
    "servicePartial": "Services/{Area}/{Area}Service.{Feature}.cs",
    "tests": "{Area}/{Area}ControllerTests.cs",
    "testsPartial": "{Area}/{Area}ControllerTests.{Feature}.cs",
    "granularity": "feature"
  },
  "thresholdBytes": 20000
}
```

- The TypeScript patterns are relative to the web folder. The C# patterns are relative to the API project's folder, and the tests' to the test project's folder.
- The placeholders: `{area}` (kebab case), `{Area}` (Pascal case), `{page}` (kebab case), `{Feature}` (the partial's name, Pascal case), and in `test`, `{dir}` and `{stem}`.
- `thresholdBytes` is the size above which a file gets a warning. The default is 20000.
- What no source gives takes the design's default. The plan says which source each key came from.

## See also

- [Start a new app from a template](../tutorials/start-from-a-template.md)
- [`/create-app`](../chat-commands/create-app.md)
- [`codelead rebuild init`](../cli/codelead-rebuild-init.md)
- [Configuration file](configuration.md#greenfield): `greenfield.templates`
- [The rebuild](../modernization/rebuild.md)
- [Rebuild manifest](rebuild-manifest.md)
