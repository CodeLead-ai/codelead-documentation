---
title: Set up local models
description: Connect CodeLead to a model server on your machine or your network - its address, the model id, the context length and the roles - and check it with codelead doctor.
kind: how-to
area: how-to
date: 2026-11-20
---

# Set up local models

This guide connects CodeLead to a local model server: a program on your machine, or on a machine of your network, that runs a model. With a local server, no code leaves your network, and a call costs nothing. CodeLead uses a local server by default, at `http://localhost:1234/v1`.

**In short**
- You start the model server with a model loaded, and you tell CodeLead its address when it is not the default.
- You choose the model id for each [role](../concepts/models-and-roles.md) (a kind of work), and you tell CodeLead the model's context length.
- [`codelead doctor`](../cli/codelead-doctor.md) checks that the server answers and that it serves the model that you named.

## Before you begin

- [CodeLead is installed](../get-started/install.md), and you ran [`codelead init`](../cli/codelead-init.md) in your project. It writes `.codelead/config.json`.
- You have a model server. LM Studio works without setup. Another server works too when it offers an OpenAI-compatible API, such as llama-server or mlx_lm.server.
- You downloaded a model in that server.

> [!NOTE]
> **New to this?** A *model server* runs a model and answers requests from other programs. A *model id* is the exact name that the server gives a model. A *role* is a kind of work, such as the preflight (the analysis before a change) or the repair. See [Models and roles](../concepts/models-and-roles.md).

## Start the server

1. Open your model server and load a model.
2. Set the context length when you load the model. The context length is the number of tokens that the model can read in one request. Choose the largest one that your machine can hold. CodeLead's prompts carry files and build output, so they are large.
3. Start the server. In LM Studio, start it on the **Developer** tab. Its address is `http://localhost:1234/v1`.
4. For a server on another machine, allow access from the network on that machine. In LM Studio, turn on **Serve on Local Network**.

## Tell CodeLead the address

CodeLead's local provider is named `lmstudio`, whatever server answers at its address. When your server runs at `http://localhost:1234/v1`, you set nothing.

For another address, choose one way:

- Put the address in the project's `.env` file. It wins over the configuration file:

  ```text
  CODELEAD_BASE_URL=http://<host>:1234/v1
  ```

- Or change `providers.lmstudio.baseUrl` in `.codelead/config.json`.
- Or, in the [extension](../extension/settings.md), open the Settings page, choose the provider in **Model**, type the **Address**, and select **Save**.

A second kind of local server can stay beside the first one. Add an entry under `providers` in `.codelead/config.json`, with `"type": "openai-compatible"`, its `baseUrl`, and `"isRemote": false`. A role then names it as its `provider`.

## Choose the model

1. List the models that the server offers, with [`codelead models`](../cli/codelead-models.md):

   ```console
   codelead models
   ```

   The command prints one `- <model id>` line for each model. It asks the server for its list. It makes no model call.

2. Name the model in `.codelead/config.json`, in the entry of each role:

   ```json
   "modelRoles": {
     "default":   { "provider": "lmstudio", "model": "<model id>", "contextTokens": 32768 },
     "fast":      { "provider": "lmstudio", "model": "<model id>", "contextTokens": 32768 },
     "preflight": { "provider": "lmstudio", "model": "<model id>", "contextTokens": 32768 },
     "architect": { "provider": "lmstudio", "model": "<model id>", "contextTokens": 32768 },
     "deep":      { "provider": "lmstudio", "model": "<model id>", "contextTokens": 32768 },
     "critic":    { "provider": "lmstudio", "model": "<model id>", "contextTokens": 32768 }
   }
   ```

   An empty `model` sends no model name. The server then answers with the model that it has loaded.

3. Set `contextTokens` to the context length that you loaded the model with. CodeLead sizes its patch, retry and repair prompts from this number, so that a prompt fits the model.

> [!TIP]
> To pin every role to one model for one run, put `CODELEAD_MODEL=<model id>` in `.env`. To change one role, use `CODELEAD_MODEL_<ROLE>`, for example `CODELEAD_MODEL_FAST`. These variables win over the file. See [Environment variables](../reference/environment-variables.md).

## Give the roles different models

Each role can use its own model. For example, a small model for the `fast` role, and a larger one for `architect` and `deep`. The `critic` role reviews the plan of a new app, and takes the `architect` role's model when it has no entry. To send the repairs to a stronger model, see [Use a stronger model for repairs](use-a-stronger-model-for-repairs.md).

Check which model each role uses with [`codelead config`](../cli/codelead-config.md). Right after `codelead init`, every role is on `lmstudio` with no model:

```console
codelead config
```

```output
…
Model Roles
  default: lmstudio / (provider default)
  fast: lmstudio / (provider default)
  preflight: lmstudio / (provider default)
  architect: lmstudio / (provider default)
  deep: lmstudio / (provider default)
  critic: lmstudio / (provider default)
…
```

`(provider default)` means that no model is named: the server's loaded model answers.

## Check that it worked

1. Run the doctor in the project folder:

   ```console
   codelead doctor
   ```

2. Read the two model lines. When the server answers, the line `Model server at …` is `ok` and lists the models. The line `Configured model is served` is `ok` and names your model.
3. When the server does not answer, the line says `FAIL` and how to fix it. Here the address points to a port where no server runs:

   ```output
   warn     Model settings overridden by the environment          lmstudio's base URL is http://127.0.0.1:9/v1 from CODELEAD_BASE_URL (the file says http://localhost:1234/v1)  → If it is not meant for this folder, remove CODELEAD_BASE_URL from the shell's profile or the program that starts CodeLead, then open a new terminal. codelead config shows every value with its source.
   FAIL     Model server at http://127.0.0.1:9/v1                 Could not connect to lmstudio at http://127.0.0.1:9/v1.  → Start LM Studio's server (Developer tab); for a server on another machine turn on "Serve on Local Network" there and set CODELEAD_BASE_URL=http://<host>:1234/v1 here.
   …
   1 required check failed; fix the lines marked FAIL or MISSING and run codelead doctor again.
   ```

   Start the server, or correct the address, and run `codelead doctor` again. Its exit code is `0` when every required check passed.

4. Optional: run `codelead doctor --probe-model`. It sends the default role's model one tiny request, and prints the answer and the time.

## Details

- **Where the values come from.** Environment variables win over `.env`, and `.env` wins over `.codelead/config.json`. `codelead config --json` says the source of each value. See [Configuration file](../reference/configuration.md).
- **The key of a local server.** The `lmstudio` provider sends the placeholder key `lm-studio` when `CODELEAD_LMSTUDIO_API_KEY` is not set. A server that needs a real key reads it from that variable.
- **The prompt budget.** With `contextTokens`, a prompt may hold `contextTokens × 3.5 − 24,000` characters, and never fewer than 20,000. Without it, the patch and repair prompts may hold 200,000 characters. The retry prompt then reads the window that the server reports, such as LM Studio's `loaded_context_length`, else it holds 90,000 characters. `CODELEAD_PROMPT_MAX_CHARS` wins over all of these.
- **A prompt that is too large.** The server refuses it. CodeLead then says `Model call failed: context length exceeded.` and `No files were changed.` Load the model with a larger context length, or ask for a smaller change. See [Fix a stop](fix-a-stop.md#fit-the-prompt-to-the-model).
- **The override rule.** A generic `CODELEAD_MODEL` is refused when a role uses another provider than the default role. See [Models and roles](../concepts/models-and-roles.md#details).

## See also

- [Models and roles](../concepts/models-and-roles.md)
- [`codelead models`](../cli/codelead-models.md), [`codelead config`](../cli/codelead-config.md) and [`codelead doctor`](../cli/codelead-doctor.md)
- [Set up remote models](set-up-remote-models.md)
- [Use a stronger model for repairs](use-a-stronger-model-for-repairs.md)
- [Configuration file](../reference/configuration.md) and [Environment variables](../reference/environment-variables.md)
- [Settings](../extension/settings.md) in the extension
