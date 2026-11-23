---
title: Run profile
description: How CodeLead starts an application for a walk - the old application's profile in modernization.source, with the iis-express preset, and the rebuild's run block, codelead.run-profile/1, key by key.
kind: reference
area: reference
date: 2026-11-20
---

# Run profile

**Applies to:** CodeLead 0.1 and later

A **run profile** tells CodeLead how to start an application on this machine, where it answers, and when it is ready. CodeLead starts the application for a stage that needs it running, such as the Explorer's crawl or a [walk](../modernization/walk-and-parity.md), and stops it after. There are two profiles: one for the old application, and one for the rebuilt application. This page describes both.

| Profile | Where | Schema | Written by |
|---|---|---|---|
| [The old application's](#the-old-applications-run-profile) | `modernization.source` in the old application's `.codelead/config.json` | none of its own | [`codelead modernize source`](../cli/codelead-modernize-source.md), `/modernize --source` |
| [The rebuild's](#the-rebuilds-run-profile) | the `run` block of `docs/codelead/rebuild/rebuild.json` in the rebuild | `codelead.run-profile/1` | [`codelead rebuild init`](../cli/codelead-rebuild-init.md), then you |

A plain repository that walks its specs after a landing uses a third file of the same shape, `.codelead/run.json`. See [Run file for specification runs](run-file.md).

## Rules for every command of a profile

A command in a profile is one executable and its arguments. CodeLead starts it without a shell, so that a profile can never run something that you did not write.

- **`command`** names one executable, on the `PATH` or as a path. A command with a space is refused, with one exception: an absolute path, such as `C:\Program Files\IIS Express\iisexpress.exe`, may hold spaces and brackets.
- **`args`** holds the arguments, one word each.
- A command with a shell operator is refused: `; & | < > $ ( ) ' " * ? { } [ ] # ! % ^` and the backtick.
- On Windows, `npm`, `npx` and a `.cmd` file run through `cmd.exe`. An argument that holds a space, a quote or `& | < > ^ %` is refused there.
- **`cwd`** is a folder inside the repository, relative to its root. A path that leaves the repository is refused.
- **`env`** sets variables over CodeLead's own environment. **`unsetEnv`** takes variables out of it. A name cannot be in both.
- A key that starts with `//` is a comment. CodeLead passes over it.
- A profile with a problem is refused whole. CodeLead says every problem in plain words, and starts nothing.

## The old application's run profile

The old application's profile lives in its own configuration, under `modernization.source`. The Read stage of the [modernization program](../modernization/the-program.md) starts the application from it for the Explorer. It waits until the application answers, and stops it when the crawl ends. See [Start the old application for CodeLead](../modernization/source-run-profile.md).

| Key | Type | Required | What it does |
|---|---|---|---|
| `command` | string | Yes | The executable that starts the application, for example `dotnet` or `iisexpress.exe`. |
| `args` | list of strings | No | Its arguments. |
| `cwd` | string | No | Where it runs, inside the repository. |
| `env` | object | No | Variables set for it. |
| `unsetEnv` | list of strings | No | Variables taken out for it. |
| `url` | string | Yes | Where it answers: `http://localhost:<port>/` or `http://127.0.0.1:<port>/`, nothing after the port. |
| `ready` | string | No | The path that answers once it is up, starting with `/`. The default is `/`. The application is ready when the path answers HTTP 200 to 399, 401 or 403: a page, a redirect (not followed) or a sign-in challenge. No answer, a 5xx or another status is not ready. |
| `startMs` | whole number, 1000 or more | No | How long it may take to answer `ready`, in milliseconds. The default is 120000: IIS Express compiles the site on its first request. |
| `allowed` | boolean | No | Your yes to CodeLead starting the application. Absent or `false`, nothing is started, and the stage waits for you to start it. |

When the address already answers before the stage, CodeLead uses the application as it is, and never stops it.

### Example: an SDK-style project

The command:

```console
codelead modernize source --url http://localhost:5080/ --command dotnet -- run --project WebApplication1/webapp.csproj --urls http://localhost:5080
```

writes this block:

```json
"modernization": {
  "source": {
    "command": "dotnet",
    "args": ["run", "--project", "WebApplication1/webapp.csproj", "--urls", "http://localhost:5080"],
    "url": "http://localhost:5080/",
    "allowed": true
  }
}
```

and prints it in words:

<!-- described, not captured -->
```output
The old application's run profile is recorded in .codelead/config.json (modernization.source):
  command: dotnet run --project WebApplication1/webapp.csproj --urls http://localhost:5080
  answers at: http://localhost:5080/ (ready when / answers with a page, a redirect or a sign-in challenge (HTTP 200 to 399, 401 or 403), within 120 s)
  allowed: yes, CodeLead starts it for the stages that need it running (the read stage's Explorer crawl) and stops it after
```

### The iis-express preset

`--preset iis-express` composes the profile that serves the web project with IIS Express, as Visual Studio does. It runs on Windows only.

```console
codelead modernize source --preset iis-express --url http://localhost:5080/
```

<!-- described, not captured -->
```json
"source": {
  "command": "C:\\Program Files\\IIS Express\\iisexpress.exe",
  "args": ["/path:<your project>\\WebApplication1", "/port:5080"],
  "url": "http://localhost:5080/",
  "allowed": true
}
```

- The executable is `IIS Express\iisexpress.exe` under the `ProgramFiles` folder, else under `ProgramFiles(x86)` when only that one holds it. When neither holds it, the profile is still written, with a note to install IIS Express or to correct the path.
- `/path:` is the web project's folder, as an absolute path. With several web projects, name one with `--project <NAME>`.
- The address must use `localhost`. IIS Express answers `400 Invalid Hostname` to `127.0.0.1`, so the preset refuses it.
- On another system the preset is refused, with the line for an SDK-style project instead:

```output
Refused: IIS Express runs on Windows, and this machine is macOS: run the preset on the Windows machine the application runs on, or record another way to start it here (an SDK project: codelead modernize source --url http://localhost:5080/ --command dotnet -- run --project <web project>.csproj --urls http://localhost:5080).
```

`codelead modernize source --show` prints the profile, and `--clear` takes it out. `--no-allow` records the profile without your yes.

## The rebuild's run profile

The rebuild's profile is the `run` block of the [rebuild manifest](rebuild-manifest.md). It starts the new API and the new web application for the acceptance walk of each part, and it holds what an unattended [`codelead rebuild run`](../cli/codelead-rebuild-run.md) reads. Without the block, acceptance by walking is off.

`codelead rebuild init --identity keep --database sample-seed` writes this block for the Basic Store sample. The time zone is the machine's:

<!-- described, not captured -->
```json
"run": {
  "schema": "codelead.run-profile/1",
  "api": { "command": "npm", "args": ["run", "api:dev"], "url": "http://localhost:5080", "ready": "/health", "startMs": 180000 },
  "web": { "command": "npm", "args": ["run", "dev"], "url": "http://localhost:3000", "ready": "/", "env": { "NEXT_PUBLIC_API_URL": "http://localhost:5080" } },
  "journeys": "docs/codelead/rebuild/journeys.json",
  "parking": { "attempts": 3 },
  "clock": { "timeZone": "America/Los_Angeles" },
  "reset": { "command": { "command": "node", "args": ["scripts/reset-sample-seed.mjs"] }, "allowed": true },
  "unattended": { "maxParts": 12, "maxCostUsd": 20, "maxMinutes": 480, "repairsPerSession": 8 }
}
```

`rebuild init` says it in one line:

```output
Run profile: API npm run api:dev on http://localhost:5080, web npm run dev on http://localhost:3000; clock America/Los_Angeles; parking after 3 attempts; day reset node scripts/reset-sample-seed.mjs; unattended caps 12 parts, $20, 480 minutes.
```

The ports are the template's, unless the old application answers on one of them. Then `rebuild init` moves the port, and its `Ports:` line says so. With the program's address `http://localhost:5080/`, the API moves to 5081: `api.url` and `NEXT_PUBLIC_API_URL` both read `http://localhost:5081`. See [`codelead rebuild init`](../cli/codelead-rebuild-init.md#the-ports).

`docs/` is outside the code stamp: a change to the profile never makes a stored walk stale.

### The keys of run

| Key | Type | Required | What it holds |
|---|---|---|---|
| `schema` | string | Yes | `codelead.run-profile/1`. |
| `api` | server | Yes | How to start the new API. |
| `web` | server | Yes | How to start the new web application. |
| `checks` | list | No | What must answer besides the ready paths. |
| `journeys` | string | No | The journeys file, relative to the root. The default is the walk's own. |
| `acceptance.repairRounds` | whole number, 0 to 10 | No | The repair rounds that a walk may drive after it refused a part. |
| `parking.attempts` | whole number, 1 to 20 | No | After how many attempts that did not land a part is parked. The default is 3. |
| `clock` | object | No | The server's time zone and, with consent, how CodeLead sets the clock. |
| `reset` | object | No | What puts the data back before every journey. |
| `models` | object | No | The model ladder. |
| `unattended` | object | No | What an unattended run reads. |
| `lanes` | object | No | How lane runs claim their lanes. |
| `parallel` | `sequential` or `maximize` | No | The plan's preference. The default is `sequential`. `maximize` is noted in the plan, and changes nothing yet. |

The plan reads `parking`, `models`, `unattended` and `lanes` with or without the rest of the block.

### A server: api and web

| Key | Type | Required | What it does |
|---|---|---|---|
| `command`, `args`, `cwd`, `env`, `unsetEnv` | | `command` | As in [the rules](#rules-for-every-command-of-a-profile). |
| `url` | string | Yes | `http://localhost:<port>` or `http://127.0.0.1:<port>`. The two servers need different ports. |
| `ready` | string | No | The path that answers 200 once the server is up. The default is `/`. |
| `startMs` | whole number, 1000 or more | No | How long the server may take. The default is 120000 for the API and 60000 for the web application. |
| `restartWhen` | list of patterns | No | Changed paths that restart a server kept between walks, written as `.gitignore` patterns. The API starts again for every walk anyway. |
| `prepare` | command | No | A step that runs to its end before every start, such as a production build. It takes `timeoutMs`, 180000 by default. |

### checks

Each check runs once the server that it names is ready:

| Key | Type | What it holds |
|---|---|---|
| `name` | string | The words that a failed check is reported by. Unique. |
| `method` | string | An HTTP method, such as `OPTIONS`. |
| `url` | string | An address on the API's or the web server's port. |
| `headers` | object | Optional request headers. |
| `expect` | whole number, 100 to 599 | The status that it must answer. |

Example: the browser's preflight request to the API, which the web application's calls depend on:

```json
"checks": [
  { "name": "CORS preflight", "method": "OPTIONS", "url": "http://localhost:5080/api/products", "headers": { "Origin": "http://localhost:3000", "Access-Control-Request-Method": "GET" }, "expect": 204 }
]
```

### clock

| Key | Type | What it holds |
|---|---|---|
| `timeZone` | string, required | The server's time zone, an IANA name such as `Europe/Berlin`, or `UTC`. The walk reads the server's day in it. |
| `control` | `command` | Declares that CodeLead sets the clock for a journey that needs it. |
| `set` | command | Sets the clock. `{time}` in its arguments stands for the wall time in the server's zone, such as `2026-10-09T23:58:00`. It must hold `{time}`. |
| `restore` | command | Puts the clock back after the journey. |
| `allowed` | boolean | The operator's consent to set this machine's clock. Without it nothing is run, and those journeys are listed, not walked. |

A command of `clock` and of `reset` is either a line, split at spaces and never given to a shell, or an object `{ command, args, cwd, env, unsetEnv }`. A line cannot hold a quote.

### reset

| Key | Type | What it holds |
|---|---|---|
| `command` | command, required | What puts the application's data back where the recorded journeys found it, before every journey. |
| `allowed` | boolean | The operator's consent to run it. Without it nothing is run. |

### run.models

The model ladder of a rebuild run. A part climbs from the compose rung to the repair rung, then to an intervention, then to a person. Without `models` and without a `CODELEAD_LADDER_*` variable, the ladder is off.

| Key | Type | What it holds |
|---|---|---|
| `compose` | `{ provider, model }` | The model that composes the parts. Absent: the roles as configured. |
| `repair` | `{ provider, model, after }` | The model that a part climbs to once its repair rounds run out or it parks. `after` is `rounds`. Absent: the compose model. |
| `intervene` | `{ provider, model, effort, maxTurns, maxCostUsd, maxMinutes }` | The model that takes a part over once the repair rung failed too. The defaults are 40 turns, 5 dollars and 60 minutes, at most 200, 100 and 480. Absent: the repair model at the highest effort that its provider takes. |
| `escalate` | `{ stopsAfter, then }` | The tries that the repair rung gets, 2 by default and at most 10, and what follows: `ask`, a person's turn. |

A provider is a name of the `providers` of the rebuild's [configuration](configuration.md#providers). The `CODELEAD_LADDER_*` variables override each field for one run. See [Environment variables](environment-variables.md#the-model-ladder).

### run.unattended

What [`codelead rebuild run`](../cli/codelead-rebuild-run.md) reads. Every key is optional. The caps stop the run between sessions: the session in flight finishes. The command's own options win over them.

| Key | Type | What it holds |
|---|---|---|
| `maxParts` | whole number, 1 to 500 | The most parts that the run takes. |
| `maxCostUsd` | number above 0, at most 1000 | The most that the run's model calls may cost, in dollars, where the provider prices its calls. A local model costs nothing here. |
| `maxMinutes` | whole number, 1 to 10080 | The longest that the run may go on. |
| `cleanTree` | boolean | `true` refuses to start on a tree with uncommitted changes to its code. |
| `repairsPerSession` | whole number, 0 to 20 | The most repair rounds that one session takes. The default is 8. |
| `credentials` | string | The file that the walk reads the accounts' passwords from, outside the repository. `~/` is your home folder. |
| `env` | object | CodeLead's own switches for the run, `CODELEAD_*` names only. A name that holds `PASSWORD`, `SECRET`, `TOKEN`, `API_KEY` or `PRIVATE` is refused, because the profile is committed. |

Example:

```json
"unattended": {
  "maxParts": 12,
  "maxCostUsd": 20,
  "maxMinutes": 480,
  "cleanTree": true,
  "repairsPerSession": 8,
  "credentials": "~/.codelead/walk-credentials.json",
  "env": { "CODELEAD_SKIP_PREFLIGHT": "1", "CODELEAD_CHECKPOINT_COMMITS": "1" }
}
```

### lanes

| Key | Type | What it holds |
|---|---|---|
| `branches` | boolean | `true`: each lane run works on the branch `codelead/lane-<area>`, made from the current branch and left for a person to merge. The default is `false`. |
| `claimStaleMinutes` | whole number | How long another person's claim on a lane stays live with no heartbeat. The default is 720, twelve hours. |
| `sync` | boolean | Syncs the lanes' claims with the remote `origin`: a fetch before a lane is chosen, a push after each commit of a claim. On by default when the clone has `origin`. `false` turns it off. It never pulls or merges. |

## A refused profile

A profile with problems is refused whole. Each problem names its key:

```output
Refused: the run profile is not recorded: modernization.source.command "dotnet run" holds whitespace: it names one executable (on the PATH, or a path without spaces), and its arguments go in modernization.source.args.
```

Other examples of the words:

- `run.api.url and run.web.url both use port 5080: each server needs a port of its own.`
- `run.web.ready "health" is not a path on the server: it starts with "/" (for example "/health").`
- `run.unattended.env names CODELEAD_REPAIR_API_KEY, which holds a secret: the profile is committed with the repository, so a secret stays in the environment or the git-ignored .env.`

## See also

- [Start the old application for CodeLead](../modernization/source-run-profile.md)
- [`codelead modernize source`](../cli/codelead-modernize-source.md) and [`codelead rebuild init`](../cli/codelead-rebuild-init.md)
- [`codelead rebuild run`](../cli/codelead-rebuild-run.md) and [`codelead walk`](../cli/codelead-walk.md)
- [Rebuild manifest](rebuild-manifest.md)
- [Run file for specification runs](run-file.md)
- [Unattended runs and the run lock](../concepts/unattended-runs.md)
