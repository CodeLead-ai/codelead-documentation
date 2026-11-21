---
title: Waive a security finding
description: Decide each security finding that changes who may do what - waive it with a reason, or fix it on purpose - then record the choice so that the in-place stage goes on.
kind: how-to
area: how-to
date: 2026-11-20
---

# Waive a security finding

This guide decides the security findings that the modernization program leaves to a person. Most findings of the dossier's security scan are fixed by the in-place transform `S`, under the pins. Two classes are different: their fix changes who may do what. For each such finding, you **waive** it with a reason, or you **fix it on purpose**.

**In short**
- A finding of the class `unauthorized-write-action` or `permissive-cors` is a decision for a person. The Upgrade in place stage waits on it.
- You waive a finding with [`/modernize --waive`](../chat-commands/modernize.md#--waive) `<id> --reason <why>`, or you fix it with [`/modernize --transform`](../chat-commands/modernize.md#--transform) `S --security <id>`.
- You record the choice with [`codelead modernize answer`](../cli/codelead-modernize-answer.md) `security=decided`, or let the stage decide all of them with `fix-all` or `fix-none`.
- Every waiver, with its reason, goes into `docs/codelead/security-waivers.json` and the audit pack.

## Before you begin

- The program ran up to the Upgrade in place stage, and the stage waits. Or the dossier exists, and you work by hand. See [The modernization program](../modernization/the-program.md).
- You know what the application is meant to allow. For example, the Basic Store sample is a store that accepts writes without sign-in, by design.

> [!NOTE]
> **New to this?** The [journeys](../analyst/concepts.md) recorded the application as it is today. A fix that adds a sign-in requirement refuses callers that the journeys saw succeed. That can be right, or it can break what the store is for. So a person decides.

## Read the decisions

1. Run [`codelead modernize status`](../cli/codelead-modernize-status.md). The stage **Transform in place** waits, with a problem such as `9 security findings are decisions for a person`. That is the count on the Basic Store sample: nine write actions, S-003 to S-011.
2. Read the problem's reason. It says why the fix changes who may do what, and lists each finding with its two ways on:

   <!-- described, not captured -->
   ```output
   S-003 (Create accepts a write without [Authorize], WebApplication1/Controllers/ClientsController.cs:63): waive it with a reason: /modernize --waive S-003 --reason "<why it is meant to be so>"; or fix it on purpose: /modernize --transform S --security S-003
   ```

3. For each finding, decide: is it meant to be so?

The two classes, and what their fix does:

| Class | What the fix does |
|---|---|
| `unauthorized-write-action` | It adds an authorization requirement: a caller who may write today without signing in is refused. |
| `permissive-cors` | It names the origins that may call with the user's credentials: a site that calls today from another origin is refused. |

## Waive a finding that is meant to be so

1. Open the chat in the old application's folder, or run [`codelead interactive`](../cli/codelead-interactive.md) there.
2. Waive the finding by its id, with the reason after `--reason`:

   ```console
   /modernize --waive S-003 --reason the store accepts writes without sign-in by design
   ```

   <!-- described, not captured -->
   ```output
   Waived S-003 Create accepts a write without [Authorize] — the store accepts writes without sign-in by design. Recorded in docs/codelead/security-waivers.json (1 waiver(s)); the audit pack lists it under 5-security/.
   ```

3. Write the reason for a reviewer. It is the record of why the application keeps the exposure.

You can type the reason with quotes or without them. When the whole reason is in one pair of quotes, `"…"` or `'…'`, CodeLead takes the quotes off. A waiver without a reason is refused: `--waive requires --reason <why>: a waiver without a reason is not recorded.`

## Fix a finding on purpose

When a finding is not meant to be so, fix it with the transform `S`, named by its id:

```console
/modernize --transform S --security S-004
```

CodeLead composes the fix and runs it as a change: the patch, the patch check, your yes, then the gate with the pins. Naming a finding by its id fixes it even when it was waived before.

> [!CAUTION]
> A fix that adds a sign-in requirement changes what your journeys recorded. The rebuild's walk then sees a refusal where the old application said yes. Record the journeys again, or update them, after such a fix.

## Record the choice

1. When every decision is waived or fixed, record it:

   ```console
   codelead modernize answer security=decided
   ```

   <!-- described, not captured -->
   ```output
   Answers recorded: security=decided.
   Then: codelead modernize run in-place (Transform in place waits).
   ```

2. Run the line that it names: `codelead modernize run in-place`. The transforms that landed before are not run again. The stage ends `done`.

S is not run again. The stage knows a landed S by the findings that it covered, so the fixes of the first run stay as they are.

Two other answers decide all the findings at once:

- `security=fix-all`: the stage fixes them all on purpose, by their class. Only the open findings are taken, never a waived one.
- `security=fix-none`: the stage leaves them open. Waive each with its reason, so that the audit pack says why.

A value that the program does not take is refused, with the three that it takes.

## Check that it worked

- `docs/codelead/security-waivers.json` lists each waiver with its `key`, `reason`, `by` and `date`. `by` is your git user name.
- `codelead modernize status` shows **Transform in place** as `done`. When you fixed some findings on purpose, its summary says, for example, `2 security findings that change who may do what left as the person decided them (security=decided)`. A waived finding is not counted there.
- After the Hand over stage, the audit pack's `5-security/DISPOSITIONS.md` lists each finding as fixed, open or waived, with the reason of each waiver.

## Details

- **The other classes.** Their fix changes nothing that a legitimate caller does or sees. Examples are output encoding, a parameterized query, a token on a form and a cookie flag. The in-place stage fixes those by itself, under the pins.
- **The selectors of S.** `all` means all open findings, and a class name means the open findings of that class. A waived finding is never taken by them. An id or a key names one finding on purpose.
- **The key** of a finding is `class|file|title`, from `security-exposure.json`. `/modernize --waive` takes the id, such as `S-003`, or the key.
- **A second waiver** of the same finding replaces the first, and the output says so.
- **Without a security report**, a waiver by id is refused: run `/modernize --plan` first, or waive by key.
- **The file** has the schema `codelead.security-waivers/1`. See [Extraction dispositions and security waivers](../reference/dispositions-and-waivers.md).

## See also

- [Transforms](../modernization/transforms.md)
- [`/modernize`](../chat-commands/modernize.md#--waive)
- [`codelead modernize answer`](../cli/codelead-modernize-answer.md)
- [The modernization program](../modernization/the-program.md)
- [The audit pack](../modernization/audit-pack.md)
- [Extraction dispositions and security waivers](../reference/dispositions-and-waivers.md)
