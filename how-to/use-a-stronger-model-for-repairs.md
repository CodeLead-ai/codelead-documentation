---
title: Use a stronger model for repairs
description: Send CodeLead's repairs to a stronger model with the repair role, keep the other work on your usual model, and turn on the two switches that use it at the end of a new app's build.
kind: how-to
area: how-to
date: 2026-11-20
---

# Use a stronger model for repairs

This guide sets the **repair role**: the model that fixes what [the gate](../concepts/validation-and-the-gate.md) (your project's own build and tests) refused. A common setup keeps a local model for all the other work, and gives the repairs a stronger model. The stronger model then works only on the hard part, so it costs less than running everything on it.

**In short**
- You add a `repair` entry to `modelRoles`, or you set `CODELEAD_PROVIDER_REPAIR` and `CODELEAD_MODEL_REPAIR`.
- The repair model can be remote, or a second local server named `repair-local`.
- Two switches for new apps also use it: **Polish on the repair model** and **Review pass after the build**.
- [`codelead doctor`](../cli/codelead-doctor.md) shows where the repairs go, and checks that the repair server answers.

## Before you begin

- The other roles work. See [Set up local models](set-up-local-models.md).
- For a remote repair model: its key is in `.env`, and remote models are allowed. See [Set up remote models](set-up-remote-models.md).
- You know what a [repair round](../concepts/repair.md) is: a second, smaller change that fixes what the first one broke.

> [!NOTE]
> **New to this?** Without a repair role, a repair goes to the same model as the patch. That is a fine start. Add a repair role when the repairs of your model often fail, or go in circles.

## Choose where the repairs go

The repair role answers these calls:

- [`/repair`](../chat-commands/repair.md), and the repair rounds of a pin, an in-place transform and a rebuild;
- the validation repair of a new app's step or increment;
- one more attempt of a new app's step, after the patch check refused its patch twice for the same problem.

Choose one of two kinds of model:

- **A remote model.** Use the provider `anthropic`, `openrouter`, or another remote provider of your configuration.
- **A second local server.** Use the provider `repair-local`. It is an OpenAI-compatible server at `http://localhost:8080/v1` by default. `CODELEAD_REPAIR_BASE_URL` moves it, and `CODELEAD_REPAIR_API_KEY` holds its key when it needs one.

## Set the repair role

1. Open `.codelead/config.json`.
2. Add a `repair` entry under `modelRoles`, beside the other roles. For a remote model:

   ```json
   "repair": { "provider": "anthropic", "model": "<model id>" }
   ```

   For a second local server:

   ```json
   "repair": { "provider": "repair-local", "model": "<model id>" }
   ```

3. Save the file.

Each key that the entry leaves out is taken from the role of the patch. An entry with a provider and no model asks the repair server for the patch's model, which it may not serve. So name the model too.

> [!TIP]
> **For experienced users:** for one run, set the role in the environment: `CODELEAD_PROVIDER_REPAIR=repair-local` and `CODELEAD_MODEL_REPAIR=<model id>`. They win over the file. A generic `CODELEAD_MODEL` is refused while the repair role uses another provider than the default role. Use `CODELEAD_MODEL_REPAIR` for the repair role.

In the [extension](../extension/settings.md), open the Settings page. In **Model roles**, choose the provider and the model in the row `repair`, and select **Save**.

## Use the repair model at the end of a new app's build

Two switches put the repair role's model on the end of a [new app's](../concepts/new-apps.md) build. Both are off by default. Both need the repair role. Without it, the build runs as if the switch were off.

| Switch on the Settings page | Key in `.codelead/config.json` | What it does |
|---|---|---|
| **Polish on the repair model** | `greenfield.polishOnRepairRole` | The plan's last increment, the polish, has its patches written by the repair role's model. |
| **Review pass after the build** | `greenfield.reviewPassAfterBuild` | After the last increment, the repair role's model reviews the app for code quality and ease of use, and makes one change set through the usual checks. |

To turn them on in the file:

```json
"greenfield": {
  "polishOnRepairRole": true,
  "reviewPassAfterBuild": true
}
```

`CODELEAD_POLISH_ON_REPAIR` and `CODELEAD_REVIEW_PASS_AFTER_BUILD` win over the file. Each takes `1`, `true` or `yes` for on, and `0`, `false` or `no` for off. When the review pass does not land, the build still counts as complete, and CodeLead says why.

## Check that it worked

1. Run [`codelead config`](../cli/codelead-config.md). The list of roles ends with the repair role:

   ```output
   Model Roles
     default: lmstudio / (provider default)
     fast: lmstudio / (provider default)
     preflight: lmstudio / (provider default)
     architect: lmstudio / (provider default)
     deep: lmstudio / (provider default)
     critic: lmstudio / (provider default)
     repair: repair-local / larger-local-model
   ```

   The two switches are in `codelead config --json`, under `greenfield`.

2. Run `codelead doctor`. The line `Repair role` names the provider, the address and the model. When the repair server is another server, a second line checks that it answers. Here no server runs at the repair address:

   ```output
   ok       Repair role                                           repair-local at http://127.0.0.1:9/v1, model larger-local-model
   FAIL     Repair server at http://127.0.0.1:9/v1                Could not connect to repair-local at http://127.0.0.1:9/v1.  → Start the repair server on the address CODELEAD_REPAIR_BASE_URL names, or unset CODELEAD_PROVIDER_REPAIR to send the repairs where the patch calls go.
   ```

   A remote repair provider has no server for you to start. When its key is not set, the second line names the key variable:

   ```output
   ok       Repair role                                           anthropic, model claude-sonnet-5-5
   FAIL     Repair server at anthropic                            the anthropic provider's key is not set  → Set ANTHROPIC_API_KEY in .env (the key is never printed), or unset CODELEAD_PROVIDER_REPAIR to send the repairs where the patch calls go.
   ```

   Without a repair role, the line says `unset: the repairs go where the patch calls go (CODELEAD_PROVIDER_REPAIR and CODELEAD_MODEL_REPAIR split them off)`.

3. After a repair, read the session log. The repair's model events name the role `repair`, the provider and the model. See [Read a session in the Session Log Viewer](read-a-session.md).

## Details

- **What the repair sees.** The failure, the files that it names, and the change so far. The repair's patch goes through the same patch check, the same question and the same gate.
- **When a stronger model does not help.** A failure of the environment, such as a missing tool, is not a model problem. CodeLead says so, and runs no repair round for it. See [Fix a stop](fix-a-stop.md).
- **A loop that goes in circles.** A repair can bring a file back to a state that an earlier round tried. CodeLead then stops the repair loop. See [Repair](../concepts/repair.md).
- **The pins of a modernization.** The pins write their patch on the repair role when it is remote and remote models are allowed. See [Pins](../modernization/pins.md).
- **The rounds.** A pin or an in-place transform runs its repair rounds by itself. `modernization.repair.rounds` (the default is 2) counts the rounds that did not move the failure. A round whose failure differs from every earlier one earns one more, up to twice the number. For example, round 1 fixes a compiler error and round 2 moves the failure again, so a third round runs. A rebuild part's session counts every round, up to its own cap, `repairsPerSession` in the [run profile](../reference/run-profile.md).

## See also

- [Models and roles](../concepts/models-and-roles.md#the-repair-role)
- [Repair](../concepts/repair.md)
- [`/repair`](../chat-commands/repair.md)
- [Set up local models](set-up-local-models.md) and [Set up remote models](set-up-remote-models.md)
- [Settings](../extension/settings.md) in the extension
- [Configuration file](../reference/configuration.md)
