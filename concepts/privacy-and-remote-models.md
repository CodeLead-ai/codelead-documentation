---
title: Privacy and remote models
description: Learn what leaves your machine when CodeLead works, when it leaves, how you give or refuse consent for a remote model, and where your keys live.
kind: concept
area: concepts
date: 2026-11-20
---

# Privacy and remote models

CodeLead runs on your machine. The one thing that leaves the machine while it works is a **model call**. The request, the plan and the parts of your code that CodeLead selected go to the model server that you configured. With the default configuration, that server is a local one, and nothing leaves your machine or your network. A remote model receives your code only after you consent.

**In short**
- By default every role (a kind of work) uses a local provider (a model server on your machine), and remote models are not allowed.
- A remote provider is refused until you allow remote models. Then CodeLead asks before the first call to each remote provider, and you type `y` or `n`.
- `CODELEAD_ALLOW_REMOTE_MODELS=1` is the consent of a run that nobody watches.
- Keys live only in the repository's `.env`, which git ignores. CodeLead never prints a key.

## Why it exists

Code is often the most private thing a company owns. Some code may not leave the building at all. A coding tool that sends code silently is not usable there. CodeLead makes the choice explicit: local first, remote only by consent, and a record of each call.

## How it works

### What leaves the machine, and when

| What CodeLead does | What leaves the machine |
|---|---|
| Reading the repository, the [repository map](repository-map.md), the scan, the patch check, the gate, the evidence records | Nothing. These are local. |
| A model call: the preflight, a patch, a repair, a plan, a chat answer | The prompt: CodeLead's rules, the request, the plan, the selected files, and build and test output. It goes to the provider of the call's [role](models-and-roles.md). |
| Your project's own build and test commands | Whatever those commands do themselves, such as a package restore. |
| A template from a package or a git address | The download of that template. |

A local provider, such as LM Studio on your machine or on a machine of your network, keeps every prompt on your premises. CodeLead has no telemetry, no analytics, no crash reports and no account. Its licence check uses no network.

### The consent

The `privacy` section of `.codelead/config.json` holds the consent. [`codelead init`](../cli/codelead-init.md) writes it like this:

```json
"privacy": {
  "allowRemoteModels": false,
  "confirmBeforeSendingRepoContextToRemote": true,
  "redactSensitiveFiles": true
}
```

- With `allowRemoteModels` set to `false`, a call to a remote provider is refused, with these words: `Remote model blocked by privacy settings.`
- With `allowRemoteModels` set to `true` and `confirmBeforeSendingRepoContextToRemote` set to `true`, CodeLead asks before the first call to each remote provider:

  <!-- described, not captured -->
  ```output
  Remote model selected: anthropic / <model id>

  CodeLead may send repository snippets to this provider.

  Continue? [y/n]
  ```

  This step is hard to undo: what was sent cannot be called back. So CodeLead recommends nothing, and Enter gives nothing: you type `y` or `n`. A yes holds for that provider until the process ends. See [Prompts and recommended answers](prompts-and-recommended-answers.md).

- `CODELEAD_ALLOW_REMOTE_MODELS=1` sets `allowRemoteModels` and drops the question. Use it for a run that nobody watches, once the code's owner has agreed. The session log records it.

> [!IMPORTANT]
> In the extension's chat, a remote role with remote models not allowed stops the session before it starts. With remote models allowed, the chat asks the same consent question as the terminal.

A real refusal, in a folder whose repair role is remote, as the chat's session receives it:

```output
{"type":"session.ended","reason":"error","message":"A remote model is configured but the privacy settings block remote models, so the session does not start. Turn on \"Allow remote models\" in the settings (privacy.allowRemoteModels in .codelead/config.json), or set CODELEAD_ALLOW_REMOTE_MODELS=1. the repair role goes to anthropic, a remote provider the privacy settings block: …"}
```

### Keys

A provider names the environment variable of its key, such as `ANTHROPIC_API_KEY` or `OPENROUTER_API_KEY`. Put the key in the `.env` file of the repository. `codelead init` adds `.env`, `.env.local` and `.env.*.local` to `.gitignore`, so that git never commits the key. [`codelead config`](../cli/codelead-config.md) and [`codelead doctor`](../cli/codelead-doctor.md) say whether a key is set, never its value. `codelead config` masks a default key, for example `lm*******`.

## What it is not

- **Not a filter of your code**. CodeLead leaves `.env` files, private keys and database files out of the files that it selects for a prompt. It goes by the file's name only. The code files that it selects go to the model as they are, and so does what you type or paste.
- **Not a promise about the provider**. What a remote provider keeps, and for how long, is that provider's policy. Read its terms.
- **Not hidden**. The session log keeps every prompt that left the machine. The run report of a new app says when a remote provider was in use, and what left.

## Details

- **What a remote provider receives** is exactly what the session log keeps under `model/` for each call. Treat the sessions folder as you treat the repository. See [Sessions](sessions.md).
- **Evidence records** hold the request, the file names, the check's verdict and the commands with their exit codes. They hold no code and no diff. See [Evidence records](evidence-records.md).
- **`redactSensitiveFiles`**. `codelead config` shows this switch, and says that it is not used. Whatever its value, CodeLead's ignore rules keep the sensitive files out of its scans, the chat's sources and the patch prompt. The rules go by the file's name. They leave out `.env` and `.env.*`, `id_rsa` and `id_ed25519`. They also leave out files that end in `.pem`, `.key`, `.p12`, `.pfx`, `.sqlite` or `.db`, and files named `secrets.*` or `credentials.*`. Nothing inside a file that is sent is redacted. What you type or paste is sent as you wrote it.
- **The doctor's line**. `codelead doctor` has the row "Remote models consent (CODELEAD_ALLOW_REMOTE_MODELS)". Without consent and with a local default, it says: `not given (the default provider is local; nothing is sent off this machine)`.
- **Unattended runs**. A run that nobody watches is refused when its model is remote and the consent is missing. It is also refused when the configuration would ask before each call. See [Unattended runs and the run lock](unattended-runs.md).
- **A modernization's documents**. [`/modernize --plan --redacted`](../chat-commands/modernize.md) writes a copy of the survey with every line of source removed, for a reader who may not see the code. See [The dossier](../modernization/dossier.md).

> [!NOTE]
> **For team leads:** to keep all code on your premises, leave every role on a local provider and leave `allowRemoteModels` at `false`. A remote call then cannot happen by accident: it is refused with words.

## Related content

- [`codelead config`](../cli/codelead-config.md) and [`codelead doctor`](../cli/codelead-doctor.md)
- [`codelead init`](../cli/codelead-init.md) and [`codelead setup`](../cli/codelead-setup.md)
- [`codelead interactive`](../cli/codelead-interactive.md)
- [Models and roles](models-and-roles.md)
- [Set up remote models](../how-to/set-up-remote-models.md)
- [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md)
- [Settings](../extension/settings.md)
- [Environment variables](../reference/environment-variables.md)
- [Glossary](glossary.md)
