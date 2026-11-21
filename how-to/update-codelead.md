---
title: Update CodeLead
description: Install a newer CodeLead and a newer extension, check the machine, and refresh the files that CodeLead generated in each project.
kind: how-to
area: how-to
date: 2026-11-20
---

# Update CodeLead

This guide updates CodeLead, the CodeLead extension for VS Code, and the files that CodeLead generated in your projects. An update keeps your configuration, your keys, your board and your evidence. A few generated files are refreshed by one command each.

**In short**
- You install the newer package over the old one with `npm install -g`, and check it with `codelead --version`.
- You run [`codelead doctor`](../cli/codelead-doctor.md), and [`codelead setup`](../cli/codelead-setup.md) when the doctor asks for it.
- You install the newer `.vsix` file of the extension in the same way as the first time.
- In each project, you refresh the repository map, and the conventions when you use them.

## Before you begin

- You have the newer package file, named like `codelead-0.1.0.tgz`, and the newer extension file, named like `codelead-vscode-0.7.15.vsix`.
- No CodeLead command runs: no session, no chat and no unattended run. A running command keeps the old version until it ends.

## Check the version you have

Run this command in any folder:

```console
codelead --version
```

```output
0.1.0 (commit b18ba17f)
```

The commit names the exact build. Write it down: it tells you later which build made a record.

## Install the newer CodeLead

1. Open a terminal in the folder that holds the newer package file.
2. Install it over the old one:

   ```console
   npm install -g ./codelead-0.1.0.tgz
   ```

   On Windows, in PowerShell, write `.\codelead-0.1.0.tgz`. Keep the `./` or `.\`: without it, npm reads the name as a GitHub repository.

3. Run `codelead --version` again. The commit is the new one.
4. Run any command. When the terms of evaluation of the new package differ from the ones that you accepted, CodeLead shows them again. Type `yes` to accept them. `codelead --version`, `--help`, `codelead doctor` and `codelead licence` never ask.

## Check the machine

1. Run `codelead doctor` in a project folder. The first line names the new build, with the same commit as `codelead --version`:

   <!-- described, not captured -->
   ```output
   ok       CodeLead build                                        0.1.0 (commit <commit>, built <date>)
   ```

   > [!NOTE]
   > **For experienced users:** in a source checkout with a newer commit than the build, this line is a `warn`. It says `the build is older than the source: run npm run build (…)`, and `codelead --version` says the same. Run `npm run build` in the checkout.

2. Read every line marked `FAIL` or `MISSING`. A newer CodeLead can carry a newer Playwright, which needs its own Chromium. The line `Playwright's Chromium` then says to install it.
3. When it does, run `codelead setup`. It says what it will install, and asks once.
4. Run `codelead doctor` again. It ends with `Ready: every required check passed.`

## Update the extension

1. Install the newer `.vsix` file, as the first time:

   ```console
   code --install-extension codelead-vscode-0.7.15.vsix
   ```

   Or use **Install from VSIX…** in the **Extensions** view. You do not uninstall first.

2. Reload the window when VS Code asks.
3. Open a new chat. A chat that was open comes back with its history, but its session belonged to the old window. Select **Open a new chat**.

The extension runs the `codelead` command of the setting `codelead.command`. After an update of CodeLead alone, start a new chat too, so that the chat runs the new CodeLead. See [Install the extension](../extension/install.md#update-the-extension).

## Refresh each project

Run these commands in each project folder. None of them calls a model.

1. Read the repository into its map again with [`codelead map`](../cli/codelead-map.md), so that the new reader writes it:

   ```console
   codelead map --refresh
   ```

   ```output
   Repository map read again (--refresh): 1 module, 2 files; docs/codelead/repository-map.md
   …
   ```

2. When the project uses conventions, read them again: [`codelead conventions`](../cli/codelead-conventions.md). When you share them with other agents, also run `codelead conventions --agents`. It replaces CodeLead's block in `AGENTS.md` and `CLAUDE.md`, and keeps your text.
3. Commit what changed, with a message that names the new version.

You do not need to refresh the rest:

| File | What happens after an update |
|---|---|
| `.codelead/config.json` | Kept. A newer optional key that the file does not have takes its default. An older format is still read. |
| `.env` | Kept. CodeLead never writes it, except for a key that you save on the extension's Settings page. |
| `docs/codelead/board/AGENTS.md` | Refreshed by CodeLead when it opens the board, when the file is still the generated text. A file that a person edited stays as it is. |
| `.codelead/evidence/`, `.codeleadsessions/` | Kept. |
| `.codelead/modernization.json` | Kept. The program continues where it stood. |

> [!CAUTION]
> Do not run [`codelead init`](../cli/codelead-init.md) `--force` to "update" a project. It writes a new `.codelead/config.json` over yours, and your providers, roles and switches are lost. A plain `codelead init` in an initialized folder refuses: `Config already exists at <your project>/.codelead/config.json.`

## Check that it worked

- `codelead --version` prints the new commit.
- `codelead doctor` ends with `Ready: every required check passed.`, and its first line names the new build.
- In VS Code, the **Extensions** view shows the new version of CodeLead for VS Code.
- A new chat starts, and answers `/help`.

## Details

- **A configuration that is too new.** A file in a format that this CodeLead does not read is refused with `The config at … is in a format this CodeLead does not read`. Install the newer CodeLead again, or ask the person who wrote the file.
- **The plan in the extension.** The extension reads the folder's plan with `codelead licence --json`. A CodeLead without that command reads as the free plan. See [Licence and plans](../concepts/licence-and-plans.md).
- **The modernization wizard.** A CodeLead without the `modernize` command shows "This CodeLead has no `modernize` command yet. Update CodeLead, or run the steps by hand:". See [The Modernize wizard](../extension/modernize-wizard.md).
- **The global command.** `npm install -g` replaces the `codelead` command on the `PATH`. On Windows, it is a `.cmd` file, and the extension starts it in a way that works with it.

## See also

- [Install CodeLead](../get-started/install.md)
- [Install the extension](../extension/install.md)
- [`codelead doctor`](../cli/codelead-doctor.md) and [`codelead setup`](../cli/codelead-setup.md)
- [`codelead map`](../cli/codelead-map.md) and [`codelead conventions`](../cli/codelead-conventions.md)
- [Troubleshoot the extension](../extension/troubleshooting.md)
