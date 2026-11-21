---
title: codelead doctor
description: Check this machine and this folder for what CodeLead needs, with one line for each check and how to fix what is missing.
kind: reference
area: cli
date: 2026-11-20
---

# codelead doctor

**Applies to:** CodeLead 0.1 and later

`codelead doctor` checks this machine and the current folder before the first session. Each check is one line: what was found, and how to install or fix what is missing.

## Name

`codelead doctor` - Checks this machine and this folder for what CodeLead needs.

## Synopsis

```console
codelead doctor [--json] [--probe-model] [--licence]
codelead doctor -h|--help
```

## Description

Run `codelead doctor` after you install CodeLead, after you change the model settings, and whenever a session fails in a way that you do not understand. It reads, and it writes nothing.

> [!NOTE]
> **New to this?** CodeLead needs other programs: Node.js, git, and for some projects the .NET SDK or Docker. It also needs a model to talk to. `codelead doctor` checks all of them at once and tells you what to install.

### What it checks

- **CodeLead's build**: its version and the commit that it was built from. When CodeLead runs from a source checkout and the build is older than the checkout's commit, the row is a warning.
- **Tools**: Node.js (version 20 at least), npm, git, the .NET SDK and Docker. Visual Studio's MSBuild, for .NET Framework projects. SQL Server's tools, when this folder needs them. Playwright's Chromium, for [the walk](../modernization/walk-and-parity.md) and the [Analyst](../analyst/index.md).
- **The model**: whether the default role's model server answers and lists its models. The Anthropic key. The [repair role](../concepts/models-and-roles.md), and its server or, for a remote provider, its key: a missing key fails with `Set <KEY_VARIABLE> in .env`. The consent to send code to a remote model, described in [Privacy and remote models](../concepts/privacy-and-remote-models.md).
- **The environment**: every `CODELEAD_*` variable that is set, and a warning when one overrides the configuration file.
- **The machine**: disk space, that the current folder is writable, and Windows long paths on Windows.
- **The folder**: its licence. See [Licence and plans](../concepts/licence-and-plans.md).

### What each line says

Each line starts with a status:

| Status | Meaning |
|---|---|
| `ok` | Found and working. |
| `warn` | Works, with something to know. |
| `missing` | Not found. When the check is required, the exit code is 1. |
| `FAIL` | Found and not working. When the check is required, the exit code is 1. |
| `skip` | Not checked. For example, the model is not asked unless you give `--probe-model`. |

After the arrow `→`, the line says how to fix it. The last line says how many required checks failed.

### No model call by default

`codelead doctor` asks the model server for its list of models. It does not ask a model to answer, because a paid provider bills each request. Add `--probe-model` to send one tiny request.

> [!TIP]
> **For experienced users:** `--json` prints `{ "schema": "codelead.doctor/1", "ready", "exitCode", "rows": [...] }`. Each row has `check`, `status`, `found`, `fix`, `required` and `label`. A script reads `ready` or the exit code.

## Options

- **`--json`**

  Prints the checks as JSON, for scripts. The default is one line for each check.

- **`--probe-model`**

  Also sends the default role's model one tiny request: "Reply with the single word: ready". The line reports the answer, the time and, when known, the cost. A paid provider bills this request.

- **`--licence`**

  Also prints this folder's licence and its plan (free or pro). It also says how the seats were counted: the people active in the last 30 days, the records read, and who was left out. [`codelead licence`](codelead-licence.md) prints the same report without the other checks.

## Examples

- Check the machine and the current folder:

  ```console
  codelead doctor
  ```

  ```output
  ok       CodeLead build                                        0.1.0 (commit 61761f30 with changes not committed, built 2026-11-20)
  ok       Node.js 22 (20 at least)                              v22.23.1
  ok       npm                                                   10.9.8
  ok       git                                                   git version 2.54.0 (Apple Git-157)
  ok       .NET SDK (for .NET projects)                          SDK 10.0.302
  missing  MSBuild (Visual Studio, for .NET Framework projects)  not found: needed only for non-SDK .NET Framework projects  → Visual Studio's MSBuild runs on Windows only; a non-SDK .NET Framework project is built there (or declare its commands in .codelead/validation.json).
  ok       Docker (daemon reachable)                             Docker version 29.8.1, build 4a63305d74; daemon 29.9.1
  ok       SQL Server tools (sqlcmd)                             not installed; not needed here (no run profile or connection in this folder calls sqlcmd)
  ok       Playwright's Chromium (the walk, the Analyst)         playwright 1.61.1; Chromium starts (…)
  …
  skip     Model answers a request                               not asked: codelead doctor --probe-model sends one tiny request (a paid provider bills it)
  ok       Remote models consent (CODELEAD_ALLOW_REMOTE_MODELS)  not given (the default provider is local; nothing is sent off this machine)
  ok       Repair role                                           unset: the repairs go where the patch calls go (CODELEAD_PROVIDER_REPAIR and CODELEAD_MODEL_REPAIR split them off)
  ok       Disk space                                            1024.9 GB free
  ok       Licence                                               no licence file
  ```

- See what a failed check looks like. Here the model server does not answer, and an environment variable overrides the file:

  ```console
  codelead doctor
  ```

  ```output
  …
  warn     Model settings overridden by the environment          lmstudio's base URL is http://127.0.0.1:9/v1 from CODELEAD_BASE_URL (the file says http://localhost:1234/v1)  → If it is not meant for this folder, remove CODELEAD_BASE_URL from the shell's profile or the program that starts CodeLead, then open a new terminal. codelead config shows every value with its source.
  FAIL     Model server at http://127.0.0.1:9/v1                 Could not connect to lmstudio at http://127.0.0.1:9/v1.  → Start LM Studio's server (Developer tab); for a server on another machine turn on "Serve on Local Network" there and set CODELEAD_BASE_URL=http://<host>:1234/v1 here.
  missing  Anthropic API key (ANTHROPIC_API_KEY)                 not set  → Create a key in the Anthropic Console and put ANTHROPIC_API_KEY=<key> in the repository's .env (gitignored); needed for CODELEAD_PROVIDER=anthropic.
  …
  1 required check failed; fix the lines marked FAIL or MISSING and run codelead doctor again.
  ```

- Run it when CodeLead runs from a source checkout, and the checkout has a newer commit than the build. The first row warns, and says the way out:

  ```console
  codelead doctor
  ```

  ```output
  warn     CodeLead build                                        0.1.0 (commit f5ae05a3, built 2026-11-20); the build is older than the source: run npm run build (the build: commit f5ae05a3; the source: commit 5a8d11d7)  → Run npm run build in the checkout, then run the command again.
  …
  ```

  Run `npm run build` in the checkout. When the build's commit is not in the checkout's history at all, the row says `the build is not from the source's commit` instead. An installed package has no checkout, so it never shows this warning.

- Print the checks as JSON for a script:

  ```console
  codelead doctor --json
  ```

  ```output
  {
    "schema": "codelead.doctor/1",
    "ready": false,
    "exitCode": 1,
    "rows": [
      {
        "check": "CodeLead build",
        "status": "ok",
        "found": "0.1.0 (commit 61761f30 with changes not committed, built 2026-11-20)",
        "required": false,
        "label": "ok"
      },
  …
  ```

- Check the licence and the seat count too:

  ```console
  codelead doctor --licence
  ```

  ```output
  …
  Licence (docs/codelead/licence.json)
    No licence file. A project without one is not warned.
    Plan: free (no licence file)

  Warnings
    None.
  …
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Every required check passed. Warnings and optional checks do not change the code. |
| `1` | At least one required check is `FAIL` or `missing`. |

## Files

`codelead doctor` writes nothing. It reads:

- `.codelead/config.json` and `.env` in the current folder, and the environment.
- `docs/codelead/licence.json`, with `--licence` also the evidence records under `.codelead/evidence/` and the board's events.

## See also

- [`codelead setup`](codelead-setup.md): install what `doctor` finds missing.
- [`codelead config`](codelead-config.md): see every setting with its source.
- [`codelead models`](codelead-models.md): list the models of a provider.
- [Install CodeLead](../get-started/install.md)
- [Set up local models](../how-to/set-up-local-models.md)
- [Set up remote models](../how-to/set-up-remote-models.md)
- [Troubleshooting the extension](../extension/troubleshooting.md)
