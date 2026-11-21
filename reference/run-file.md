---
title: Run file for specification runs
description: The format of .codelead/run.json, codelead.run/1 - how CodeLead starts your application to walk a spec after a landing, with its servers, its seed and its reset.
kind: reference
area: reference
date: 2026-11-20
---

# Run file for specification runs

**Applies to:** CodeLead 0.1 and later

`.codelead/run.json` tells CodeLead how to start your application, so that it can walk a [spec](spec-files.md) after a change lands. [`/implement --spec`](../chat-commands/implement.md) reads it. CodeLead starts the servers that the file declares, waits until they answer, walks the spec, and stops what it started.

> [!NOTE]
> **New to this?** A *walk* follows the steps of a spec on the running application, the way a person would. It opens the first screen, clicks, fills in fields and checks what the page shows. To do that, the application must run. This file says how to start it.

## The file

| Fact | Value |
|---|---|
| Path | `.codelead/run.json` at the repository root. |
| Schema | `codelead.run/1`. Required. |
| Written by | You. |
| Read by | `/implement --spec <file>`, after the landing's validation passed. |
| In git | Yes. It holds commands and addresses, never a password. |

## How CodeLead chooses how to run the application

For the walk after a landing, CodeLead takes the first of these that exists:

1. `--base <url>`: an application that already runs. CodeLead starts nothing.
2. `.codelead/run.json`.
3. The `run` block of a rebuild's manifest, `docs/codelead/rebuild/rebuild.json`. See [Run profile](run-profile.md#the-rebuilds-run-profile).

With none of them, the walk does not run, and says why:

```output
no way to run the application (give --base, or declare .codelead/run.json)
```

The change still lands. The walk is evidence beside the gate, never in its place: the record's `specs` field holds its verdict. See [Evidence record format](evidence-record-format.md#the-fields-of-a-walk).

## A file

A task manager app with a web server, an API on its own port, a seed and a reset:

```json
{
  "schema": "codelead.run/1",
  "web": {
    "command": "npx",
    "args": ["vite", "--port", "5302", "--strictPort"],
    "cwd": "web",
    "url": "http://localhost:5302"
  },
  "api": {
    "command": "node",
    "args": ["server/index.js"],
    "url": "http://localhost:5301",
    "ready": "/health",
    "env": { "DB_FILE": "test.db" }
  },
  "seed": {
    "command": "node",
    "args": ["scripts/seed.js"],
    "cwd": "server",
    "env": { "DB_FILE": "test.db" },
    "allowed": true,
    "timeoutMs": 30000
  },
  "reset": {
    "command": "npm",
    "args": ["run", "reset"],
    "allowed": true
  }
}
```

## Keys

| Key | Type | Required | What it holds |
|---|---|---|---|
| `schema` | string | Yes | `codelead.run/1`. |
| `web` | server | Yes | The server that the walk opens. |
| `api` | server | No | The API that the pages call, when it runs apart from the web server. |
| `seed` | step | No | A step that runs once, before the servers start: a database file, fixtures. |
| `reset` | step | No | A step that runs before every journey of the walk, to put the data back. |

A key that starts with `//` is a comment. Any other key is refused.

### A server

| Key | Type | Required | What it does |
|---|---|---|---|
| `command` | string | Yes | One executable, on the `PATH` or as a path. No space: the arguments go in `args`. |
| `args` | list of strings | No | Its arguments. |
| `cwd` | string | No | A folder inside the repository, relative to its root. |
| `env` | object | No | Variables set over CodeLead's own environment. |
| `unsetEnv` | list of strings | No | Variables taken out of it. |
| `url` | string | Yes | The server's address, `http://` or `https://`, for example `http://localhost:5183`. `web` and `api` need different addresses. |
| `ready` | string | No | The path that answers once the server is ready, starting with `/`. The default is `/`. |
| `startMs` | whole number, 1000 or more | No | How long the server may take to answer. The default is 60000 for `web` and 120000 for `api`. |

The keys `restartWhen` and `prepare` of a rebuild's run profile are not taken here.

### A step: seed and reset

| Key | Type | Required | What it does |
|---|---|---|---|
| `command` | string | Yes | One executable. |
| `args` | list of strings | No | Its arguments. |
| `cwd` | string | No | A folder inside the repository. |
| `env` | object | No | Variables set for the step. |
| `allowed` | boolean | No | Your consent to run the step. Absent or `false`, the step is kept and nothing is run. |
| `timeoutMs` | whole number, 1000 or more | No | How long the step may run. The default is 120000, two minutes. |

A step without `allowed: true` is recorded as declared and `not allowed: set allowed: true in .codelead/run.json to consent`.

## What CodeLead does with it

1. It runs `seed`, when it is allowed, to its end.
2. It starts each server, and asks its `ready` path until the server answers or its `startMs` passes.
3. Before every journey of the walk, it runs `reset`, when it is allowed.
4. It walks the spec.
5. It stops every server that it started, also when you press Ctrl+C.

It never touches the servers of a rebuild's acceptance walk: those keep their own records, logs and ports.

## A refused file

A file with a problem is refused whole, with every problem in one line, and the walk does not run. For example, a file with no `web`, an unknown key and a folder outside the repository:

<!-- described, not captured -->
```output
.codelead/run.json is refused: extra is not a key here (the keys: schema, web, api, seed, reset). web is missing: how to start the application's web server (command, args, url). api.restartWhen is not a key here (the keys: command, args, cwd, env, unsetEnv, url, ready, startMs). api.cwd "../elsewhere" leaves the repository: a folder inside it, relative to its root.
```

Other problems that it names:

- `web.command "npm start" holds whitespace: it names one executable, and its arguments go in web.args.`
- `web.url and api.url are both http://localhost:5302: each server needs an address of its own.`
- On Windows, an argument of `npx` or `npm` that `cmd.exe` would read as syntax, such as `"a&b"`.

## See also

- [Specification files](spec-files.md)
- [`/implement`](../chat-commands/implement.md) and [`codelead spec`](../cli/codelead-spec.md)
- [Run profile](run-profile.md)
- [Journeys format](journeys-format.md)
- [Evidence record format](evidence-record-format.md)
