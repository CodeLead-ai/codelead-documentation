---
title: Install CodeLead
description: Install CodeLead on macOS or Windows, check the version, connect a model, and read the output of codelead doctor.
kind: how-to
area: get-started
date: 2026-11-20
---

# Install CodeLead

This page installs CodeLead and checks that your machine has what it needs. It takes about ten minutes, plus the time to download a model when you use a local one.

**In short**
- You need Node.js 22, git, and a model: a local model server, or a key for a remote provider.
- You install CodeLead from one file with npm, then check it with `codelead --version`.
- [`codelead doctor`](../cli/codelead-doctor.md) prints one line for each thing that CodeLead needs, and how to fix what is missing.

## Before you begin

You must know how to open a terminal and type a command. You do not need to know any CodeLead words yet.

You need:

- **Node.js 22.** CodeLead runs on Node.js 20, but the CodeLead Analyst (the part that records a running application) needs 22. Check with `node --version`.
- **npm**, which comes with Node.js.
- **git.** Check with `git --version`.
- **A model.** CodeLead contains no model. Choose one of the two:
  - **A local model server**, such as LM Studio, on your computer or on a computer of your network. Nothing leaves your network. See [Set up local models](../how-to/set-up-local-models.md).
  - **A key for a remote provider** (a company that runs models on the internet), such as the Anthropic API. Your code goes to that provider for each call, and only with your consent. See [Set up remote models](../how-to/set-up-remote-models.md).
- **The CodeLead package**: one file named like `codelead-0.1.0.tgz`.

> [!NOTE]
> **New to this?** A *model server* is a program that runs a model and answers requests from other programs. LM Studio is one: you download a model in it, load the model, and start its local server.

For some projects, CodeLead also uses the .NET SDK, Docker, or SQL Server's command-line tools. `codelead doctor` tells you when a folder needs one of them.

## Install the package

1. Open a terminal in the folder that holds the package file.
2. Install it as a command:

   ```console
   npm install -g ./codelead-0.1.0.tgz
   ```

   Keep the `./` in front of the file name. Without it, npm reads the name as a GitHub repository.

3. Check the version:

   ```console
   codelead --version
   ```

   You see the version and the commit that it was built from:

   ```output
   0.1.0 (commit dc9202ca)
   ```

On the first command after the install, a packaged release shows a short text of its evaluation terms. These are the terms under which you may try CodeLead. Type `yes` to accept them. CodeLead keeps the answer on your machine only. `codelead --version`, `--help`, `codelead doctor` and `codelead licence` never ask.

> [!TIP]
> **For experienced users:** for a script or a scheduled job, set `CODELEAD_ACCEPT_TERMS=yes`, so that nobody has to type the answer.

## Connect a model

- **A local server.** Start LM Studio's server, with a model loaded. CodeLead uses `http://localhost:1234/v1` by default, so you set nothing. For a server on another computer, put this line in a `.env` file in your project folder:

  ```text
  CODELEAD_BASE_URL=http://<host>:1234/v1
  ```

- **A remote provider.** Put the provider, the model and the key in the project's `.env` file:

  ```text
  CODELEAD_PROVIDER=anthropic
  CODELEAD_MODEL=<model id>
  ANTHROPIC_API_KEY=<YOUR_API_KEY>
  ```

  Then allow remote models. Run [`codelead init`](../cli/codelead-init.md) in the project folder, if you have not yet. In the `.codelead/config.json` that it writes, set `"allowRemoteModels": true` under `privacy`. CodeLead then asks you once, before it sends code to the provider. See [Privacy and remote models](../concepts/privacy-and-remote-models.md).

> [!IMPORTANT]
> Never commit the `.env` file. `codelead init` adds `.env` to the project's `.gitignore` for you.

## Run the checks

1. Open a terminal in a project folder.
2. Run the doctor:

   ```console
   codelead doctor
   ```

Here is a real run on a Mac. In this example, no model server runs at the address that the folder's configuration names:

```output
ok       CodeLead build                                        0.1.0 (commit 28eb8275 with changes not committed, built 2026-11-20)
ok       Node.js 22 (20 at least)                              v22.23.1
ok       npm                                                   10.9.8
ok       Platform                                              darwin 25.6.0
ok       git                                                   git version 2.54.0 (Apple Git-157)
ok       .NET SDK (for .NET projects)                          SDK 10.0.302
missing  MSBuild (Visual Studio, for .NET Framework projects)  not found: needed only for non-SDK .NET Framework projects  → Visual Studio's MSBuild runs on Windows only; a non-SDK .NET Framework project is built there (or declare its commands in .codelead/validation.json).
ok       Docker (daemon reachable)                             Docker version 29.8.1, build 4a63305d74; daemon 29.9.1
ok       SQL Server tools (sqlcmd)                             not installed; not needed here (no run profile or connection in this folder calls sqlcmd)
ok       Playwright's Chromium (the walk, the Analyst)         playwright 1.61.1; Chromium starts (…)
FAIL     Model server at http://127.0.0.1:9/v1                 Could not connect to lmstudio at http://127.0.0.1:9/v1.  → Start LM Studio's server (Developer tab); for a server on another machine turn on "Serve on Local Network" there and set CODELEAD_BASE_URL=http://<host>:1234/v1 here.
missing  Anthropic API key (ANTHROPIC_API_KEY)                 not set  → Create a key in the Anthropic Console and put ANTHROPIC_API_KEY=<key> in the repository's .env (gitignored); needed for CODELEAD_PROVIDER=anthropic.
skip     Model answers a request                               not asked: codelead doctor --probe-model sends one tiny request (a paid provider bills it)
ok       Remote models consent (CODELEAD_ALLOW_REMOTE_MODELS)  not given (the default provider is local; nothing is sent off this machine)
ok       Repair role                                           unset: the repairs go where the patch calls go (CODELEAD_PROVIDER_REPAIR and CODELEAD_MODEL_REPAIR split them off)
…
ok       Current folder is writable (session logs, evidence)   <your project>
ok       Disk space                                            1024.3 GB free
ok       Licence                                               no licence file
ok       CODELEAD_* switches in effect                         none (defaults: local LM Studio at http://localhost:1234/v1)

1 required check failed; fix the lines marked FAIL or MISSING and run codelead doctor again.
```

### How to read it

- **The first word** of each line is the status. `ok` means found and working. `warn` means it works, with something to know. `missing` means not found. `FAIL` means found and not working. `skip` means not checked.
- **After the arrow `→`**, the line says how to fix the problem.
- **Not every line matters to you.** Here, MSBuild is missing, but only a .NET Framework project needs it. The Anthropic key is missing, but only the remote provider needs it. Neither is a required check.
- **The last line** counts the required checks that failed. Here it is one: the model server. Start the server and run `codelead doctor` again.

> [!TIP]
> **For experienced users:** `codelead doctor --probe-model` also sends the model one tiny request, and reports the answer and the time. A paid provider bills that request. `codelead doctor --json` is for scripts.

### Install what is missing

[`codelead setup`](../cli/codelead-setup.md) installs what CodeLead can install for itself. An example is Playwright's Chromium: a browser that CodeLead uses to open running applications, for the walk and the Analyst. It says what it will do and asks once. It does not install the .NET SDK, Docker or SQL Server's tools: it says which are missing and how to install each.

## Windows notes

- Install from PowerShell with a backslash: `npm install -g .\codelead-0.1.0.tgz`.
- When PowerShell says that `codelead.ps1` is not digitally signed, the cause is PowerShell's execution policy, not the package. Run `Set-ExecutionPolicy -Scope CurrentUser RemoteSigned` once, or type `codelead.cmd` in place of `codelead`.
- `codelead doctor` also checks that Windows long paths are turned on.
- A .NET Framework project needs Visual Studio or its Build Tools. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).

## Check that it worked

- `codelead --version` prints a version and a commit.
- `codelead doctor` ends with no failed required check, and its exit code is `0`.
- The model server line is `ok`, or the remote provider's key line is `ok` when you use a remote provider.

## See also

- Next: [Quickstart: CodeLead in VS Code](quickstart-vscode.md), or [Quickstart: build a new app](quickstart-new-app.md) in a terminal.
- [`codelead doctor`](../cli/codelead-doctor.md), [`codelead setup`](../cli/codelead-setup.md), [`codelead config`](../cli/codelead-config.md)
- [Set up local models](../how-to/set-up-local-models.md)
- [Set up remote models](../how-to/set-up-remote-models.md)
- [Update CodeLead](../how-to/update-codelead.md)
- [Models and roles](../concepts/models-and-roles.md)
