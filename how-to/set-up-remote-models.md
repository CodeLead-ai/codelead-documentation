---
title: Set up remote models
description: Connect CodeLead to a model provider on the internet - the key in .env, the provider and model settings, the consent before the first call, and the cost of each call.
kind: how-to
area: how-to
date: 2026-11-20
---

# Set up remote models

This guide connects CodeLead to a remote provider: a company that runs models on the internet and bills each call. With a remote provider, the parts of your code that CodeLead selects leave your machine for each model call. So CodeLead refuses remote models until you allow them, and asks before the first call.

**In short**
- You put the provider's key in the project's `.env` file. CodeLead never prints it.
- You point a [role](../concepts/models-and-roles.md) (a kind of work), or every role, at the provider and a model.
- You allow remote models, and you answer `y` to the consent question before the first call.
- CodeLead records the tokens of each call, and its cost when the provider's price is known.

## Before you begin

- [CodeLead is installed](../get-started/install.md), and you ran [`codelead init`](../cli/codelead-init.md) in your project.
- You have an account with a provider and a key for its API. `codelead init` writes two remote providers: `anthropic` (the Anthropic API) and `openrouter` (OpenRouter, an OpenAI-compatible service).
- The owner of the code agrees that it may be sent to that provider. Read [Privacy and remote models](../concepts/privacy-and-remote-models.md).

> [!WARNING]
> A remote provider bills each call. A modernization or an unattended run makes many calls. Read the caps of a run before you start it. See [Unattended runs and the run lock](../concepts/unattended-runs.md).

## Put the key in .env

1. Open the file `.env` in the project folder, or create it.
2. Add the key under the name that the provider reads. For example:

   ```text
   ANTHROPIC_API_KEY=<your key>
   ```

   For OpenRouter, the name is `OPENROUTER_API_KEY`.

3. Check that git ignores the file. `codelead init` adds `.env`, `.env.local` and `.env.*.local` to `.gitignore`.

In the [extension](../extension/settings.md), the Settings page does the same: choose the provider in **Model**, paste the key in **API key**, and select **Save**. The page writes the key to `.env`, and then shows only **set**.

## Point the roles at the provider

Choose one of two ways.

- **Every role on the provider.** Put the provider and the model in `.env`:

  ```text
  CODELEAD_PROVIDER=anthropic
  CODELEAD_MODEL=<model id>
  ```

- **Some roles on the provider.** Name the provider and the model in the role's entry of `.codelead/config.json`. For example, only the `deep` role:

  ```json
  "deep": { "provider": "anthropic", "model": "<model id>" }
  ```

To see the model ids, run [`codelead models`](../cli/codelead-models.md) `--provider anthropic`. Without a key, it prints the models that CodeLead knows, and says that the API could not be asked.

Another OpenAI-compatible provider works too. Add an entry under `providers`, with `"type": "openai-compatible"`, its `baseUrl`, the name of its key variable in `apiKeyEnv`, and `"isRemote": true`.

## Allow remote models

1. In `.codelead/config.json`, set `allowRemoteModels` to `true` under `privacy`:

   ```json
   "privacy": {
     "allowRemoteModels": true,
     "confirmBeforeSendingRepoContextToRemote": true,
     "redactSensitiveFiles": true
   }
   ```

   In the extension, turn on **Allow remote models** in the **Privacy** section of the Settings page.

2. Start a session. Before the first call to the provider, CodeLead asks:

   <!-- described, not captured -->
   ```output
   Remote model selected: anthropic / <model id>

   CodeLead may send repository snippets to this provider.

   Continue? [y/n]
   ```

3. Type `y` and press **Enter**. Enter alone gives nothing, because the step is hard to undo: what was sent cannot be called back. A yes holds for that provider until the process ends.

With `allowRemoteModels` set to `false`, a call to a remote provider is refused before anything is sent. The words are `Remote model blocked by privacy settings.`, the provider, and the setting to change.

> [!IMPORTANT]
> A run that nobody watches cannot answer the question. Give it the consent with `CODELEAD_ALLOW_REMOTE_MODELS=1` in `.env`, once the code's owner has agreed. That variable also turns the question off.

An automated build (`/create-app --auto`, `/build-incremental`) and the modernization program in unattended mode ask the question at their start, while you are there. They ask once for each remote provider that the roles would use, before the first model call. Type `y` for each one. A no ends the command before it starts, with the outcome `stopped`. The words are `The automated build did not start: anthropic was not allowed (the answer was no), and the repair role uses it.` A consent that still comes up later in such a run stops it, because nobody is there to answer. <!-- described, not captured -->

## Read the cost of the calls

CodeLead records the tokens of every call. It also records the cost, when it is known:

- OpenRouter reports the cost of each call. CodeLead keeps it as the provider reported it.
- The Anthropic API reports no cost. CodeLead prices the tokens from its own price table, and marks the cost `computed`. For a model that is not in the table, it says once `No Anthropic price for model …`, and records the calls without a cost.

You find the cost in three places:

- **After a chat answer**: one dim line with the provider, the model, the tokens and the cost, for example `anthropic <model id> · 12,480 in, 912 out tokens, $0.0512 (computed)`. <!-- described, not captured -->
- **In the session log**: each `model.call.completed` event carries `usage`, with `costUsd` when the cost is known. See [Read a session in the Session Log Viewer](read-a-session.md).
- **In the summary of an unattended run**: [`codelead rebuild run`](../cli/codelead-rebuild-run.md) and [`codelead implement`](../cli/codelead-implement.md) add up the cost of their calls. `codelead rebuild run --max-cost-usd <x>` stops a rebuild run at a cost.

## Check that it worked

1. Run [`codelead doctor`](../cli/codelead-doctor.md) in the project folder.
2. Read the key's line. With the key in `.env`, the Anthropic line says `set in this folder's .env (value not shown)`. Without it, the line says:

   ```output
   missing  Anthropic API key (ANTHROPIC_API_KEY)                 not set  → Create a key in the Anthropic Console and put ANTHROPIC_API_KEY=<key> in the repository's .env (gitignored); needed for CODELEAD_PROVIDER=anthropic.
   ```

3. Read the consent's line. With a remote default provider and no `CODELEAD_ALLOW_REMOTE_MODELS`, it is `warn`: `not given, and the default provider is remote: an interactive session asks before sending, an unattended run is refused`. That is expected for a session that you watch.
4. Run [`codelead config`](../cli/codelead-config.md). The roles show the provider and the model, for example `deep: anthropic / <model id>`. The key is never shown. With the key set, the provider's block says where it comes from:

   ```output
     anthropic
       Type: anthropic
       Base URL: (the SDK's default endpoint)
       API key env: ANTHROPIC_API_KEY
       Default API key: (none)  (the key sent comes from ANTHROPIC_API_KEY)
       Remote: true
   ```
5. Optional: `codelead doctor --probe-model` sends one tiny request to the default role's model, and prints the time and the cost. The provider bills that request.

## Details

- **Where the key is read.** CodeLead reads `.env` in the folder where it runs, and the environment. An environment variable wins over `.env`.
- **What leaves the machine.** Each call sends the prompt: CodeLead's rules, the request, the plan, the selected files and build output. The session log keeps every prompt under `model/`. See [Privacy and remote models](../concepts/privacy-and-remote-models.md#details).
- **The extension's chat.** A remote role with remote models not allowed stops the chat's session before it starts, and names the setting. With remote models allowed, the chat asks the same consent question.
- **Runs that start with nobody there.** [`codelead rebuild run`](../cli/codelead-rebuild-run.md) and [`codelead implement`](../cli/codelead-implement.md) do not ask. They refuse before the start when a remote role has no consent. See [Unattended runs and the run lock](../concepts/unattended-runs.md).
- **The run report of a new app** says when a remote provider was in use, and what left the machine.

## See also

- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
- [Models and roles](../concepts/models-and-roles.md)
- [Set up local models](set-up-local-models.md)
- [Use a stronger model for repairs](use-a-stronger-model-for-repairs.md)
- [`codelead doctor`](../cli/codelead-doctor.md), [`codelead config`](../cli/codelead-config.md) and [`codelead models`](../cli/codelead-models.md)
- [Configuration file](../reference/configuration.md) and [Environment variables](../reference/environment-variables.md)
