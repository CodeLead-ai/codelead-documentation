---
title: The audit pack
description: What the audit pack of a modernization holds, who reads it, and how to check that nothing in it was changed.
kind: concept
area: modernization
date: 2026-11-23
---

# The audit pack

The audit pack is one folder that proves what a modernization did. It holds every change that landed, with its request, its evidence record, its commit and its attempts. It also holds the gate, the survey before and after, the security findings, and every session with its secrets removed. CodeLead writes it in the last stage, Hand over.

**In short**
- You know what the audit pack holds, layer by layer, and which file to read first.
- You know who reads it, and how they check that no file in it was changed.
- You know how to run it, and how to keep secrets out of it.

## Why it exists

The people who review a modernization rarely watch it happen. A team lead, a security reviewer, an auditor or the application's owner checks the work afterwards. They do it without CodeLead, and without the person who ran it. The audit pack answers their questions from files: what changed, what proved each change, what was refused and why, and what is left.

## How it works

[`/modernize --audit-pack`](../chat-commands/modernize.md#--audit-pack) builds the pack in `docs/codelead/audit-pack/`. The program's Hand over stage runs the same action. It reads the repository's evidence records, its git history, the dossier, the declared gate and the sessions. It copies what it needs, removes secrets, and writes a manifest.

On the Basic Store sample, after the dossier and with no landed change yet:

```console
/modernize --audit-pack
```

```output
Audit pack
Audit pack: docs/codelead/audit-pack/ (47 files, 0.2 MB)
Increments: 0 landed; sessions: 1; stops: 0. Before `e41a806` → after `ab1dad7`.
Note: No landed evidence record; layer 2 is empty.

Read README.md first; manifest.json lists every file with its SHA-256.
```

After a full modernization, the pack holds one folder for each landed increment: each pin, each transform.

### The layers

| Folder | What it holds |
|---|---|
| `README.md` | The summary. It opens with the stages that a person skipped: who, when and why. Then the rebuild as it stands, when the program's Rebuild stage is done or skipped. Then what was done, the increments, the metrics before and after, the criteria and where each proof is. Read it first. |
| `0-dossier/` | The survey, the intake and the options that the work started from. |
| `1-program/` | The program's state file, and the evidence record of each skipped stage. When the Rebuild stage is done or skipped, also the rebuild's plan as it stands, `rebuild-plan.json`, and the evidence record of the person's `rebuild=enough` answer. |
| `2-increments/<nn>-<stage>-<id>/` | One folder for each landed increment: `request.txt`, `evidence-record.json`, `commit.txt` and `diff.patch` when a commit exists, and `attempts.md` with every record of the same request. |
| `3-baselines/` | The declared gate (`validation-gate.json`), the knowledge store, the branch log, and `metrics.json`: the survey run on the tree before and the tree after. |
| `4-sessions/<session>/` | The full session log of each session, with every prompt and response, and `INDEX.md` with the reason of each stop. |
| `5-security/` | The security scan on both trees, and each finding's state: fixed, open, waived or new (`DISPOSITIONS.md`). |
| `manifest.json` | Every file of the pack with its size and its SHA-256. |

### The rebuild as it stands

A person can end the Rebuild stage with parts left: `codelead modernize answer rebuild=enough` goes on with what is built. The pack then says what is not built. When the program's Rebuild stage is done or skipped, `README.md` holds the section `## The rebuild as it stands`. CodeLead reads it from the rebuild's plan and the stage's evidence record, with no model call. It holds:

- **How the stage ended.** After an answer: its date, what the stage waited for, and the answer's record. Or the skip, or the stage's own end.
- **How many parts a walk of their own steps accepted**, of how many. Beside it: how many are accepted on what regresses, satisfied, or dropped by the plan.
- **The parts accepted on what regresses**, listed apart. The plan gives them no step of their own, and says why.
- **The parts that landed and that no walk judged**, in a table of their own. They are not finished.
- **A table of each other part that is not finished**: its state, and why, in the plan's own words. A state is, for example, `parked for a person`, `planned, waiting` or `planned and never taken`.
- **The plan's blockers**, and its notes to a person.

The plan is copied as `1-program/rebuild-plan.json`, with its secrets removed. In this example from CodeLead's tests, the person answered `rebuild=enough` while a prerequisite was parked:

<!-- described, not captured -->
```output
## The rebuild as it stands

The rebuild stage ended by the person's answer `codelead modernize answer rebuild=enough` at 2026-11-24T08:00:…Z: go on with what is built. It waited: The rebuild waits for you: 4 parts of 6 are not accepted yet: identity-seam is parked (the answer's record: `1-program/ev-….json`).

From the rebuild's plan `../shop-rebuild/docs/codelead/rebuild/rebuild-plan.json`, copied as `1-program/rebuild-plan.json`: **1 of 7 parts accepted by a walk of its own steps** (1 accepted on what regresses, 1 dropped by the plan); 4 parts not finished:

Accepted on what regresses: the plan gives them no step of their own.

- `sign-in:endpoint`: no step of the spec covers it.

Landed, and no walk judged them: not finished.

| Part | Why, in the plan's words |
|---|---|
| orders.edit:screen | landed (ev-…), and its acceptance was not judged (own-steps-unreachable): its own step orders.edit-3 waits on orders.edit-2, which fails, a step of identity-seam (parked) |

| Part | State | Why, in the plan's words |
|---|---|---|
| identity-seam | parked for a person | parked after 2 attempts that did not land (the last: ev-seam-2, the patch check refused the same change twice): a later rule or a person reopens it |
| orders.index:screen | planned, waiting | no evidence record shows it landed; waits on identity-seam (parked for a person) |
| orders.close:screen | planned and never taken | no evidence record shows it landed |
```

A rebuild with nothing left ends the line with `every part is accepted, satisfied or dropped: nothing is left`. A part that landed and that no walk judged is never counted as finished. When CodeLead cannot read the plan, the section says so, with the plan's path relative to the old application's folder.

### Before and after

`README.md` compares the two trees with the same survey. On the Basic Store sample, the table reads:

```output
| Metric | Before `e41a806` | After `ab1dad7` |
|---|---|---|
| Projects SDK-style | 0 of 0 | 0 of 1 |
| Target frameworks | - | v4.5 |
| Runtimes out of support | 0 | 1 |
| Source files / lines / test files | 0 / 0 / 0 | 74 / 5,079 / 0 |
```

Here the "before" commit is the repository's first commit, which was empty, because nothing had landed yet. Give the commit before the work started with `--since`, so that "before" is the old application.

## Who reads it

- **A team lead** reads `README.md`: what landed, how many sessions, what stopped and why.
- **A reviewer** opens an increment's folder: the request, the diff, the validation commands and their exit codes in the evidence record.
- **A security reviewer** reads `5-security/DISPOSITIONS.md`: each finding fixed, open or waived, with the reason of each waiver.
- **An auditor** checks the pack itself: every file against its SHA-256 in `manifest.json`.

> [!NOTE]
> **For team leads:** the pack stands alone. Every number in `README.md` comes from files in the layers, so a reader can check it without CodeLead.

## Secrets

CodeLead removes secrets from every text file that it copies into the pack. It writes `<redacted>` in the place of each secret. The rest of the line stays, so that a reader can still follow it. A text file is a file whose name or content says text, such as `.json`, `.md`, `.env` or `.ini`.

### What the rules remove

The rules run in this order:

| Kind of rule | What it removes |
|---|---|
| Secret files | The contents of each file that you name with `--secret-file <PATH>`, as one text. Name a file that holds one secret, such as a password file. You can name more than one file. |
| Private keys | The lines of a private key, between its `BEGIN` and `END` lines. A certificate stays, because it is public. |
| Known tokens | A token with a known shape, wherever it stands. For example: model provider keys, AWS access key ids (`AKIA…`), GitHub (`ghp_…`), GitLab (`glpat-…`), Slack (`xoxb-…`), Stripe (`sk_live_…`), Google (`AIza…`) and JSON Web Tokens. |
| Authorization | The token after `Bearer`, and the encoded user and password after `Authorization: Basic`. |
| Addresses | The password in `scheme://user:password@host`, and the `sig=` part of a shared access signature. |
| Connection strings | The values of `Password`, `Pwd`, `AccountKey`, `SharedAccessKey`, `SharedAccessSignature` and `Client Secret`. The server, the database and the user stay. |
| Keys and values | A value whose key names a secret: `password`, `pwd`, `secret`, `token`, `api_key`, `client_secret`, `access_key`, `private_key`, in any case. A longer key counts too, such as `DB_PASSWORD`, `SMTP_PASS` or `JWT_SECRET`. |
| Command lines | The value after a flag such as `--password` or `--api-key`, after `sqlcmd -P`, after `mysql -p`, and the password in `curl -u user:password`. |
| Paths | The repository's path and the home folder become `<repo>` and `<home>`. |

The key and value rule reads environment files, `export` and `set` lines, INI and properties files, YAML, JSON, XML attributes, `<add key="…" value="…" />` and code with a quoted value. A key that ends in `KEY`, such as `LICENSE_KEY`, counts only when its value looks like a secret. Such a value has 12 characters or more, with two kinds of characters, such as letters and digits. So `PRIMARY_KEY=id` stays.

In this example of an environment file, the values are fake:

```text
DB_HOST=db.example
DB_PASSWORD=not-a-real-password
export API_TOKEN=example-token-0000
PRIMARY_KEY=id
```

```output
DB_HOST=db.example
DB_PASSWORD=<redacted>
export API_TOKEN=<redacted>
PRIMARY_KEY=id
```

In a `Web.config`, the password goes, and the rest of the connection string stays:

```text
<add name="Shop" connectionString="Server=db;Database=Shop;User ID=shop;Password=example-only" />
<add key="SmtpPassword" value="example-only" />
```

```output
<add name="Shop" connectionString="Server=db;Database=Shop;User ID=shop;Password=<redacted>" />
<add key="SmtpPassword" value="<redacted>" />
```

In JSON and on a command line, the quotes and the other options stay:

```text
{"apiKey": "example-key-0000", "user": "shop", "max_tokens": 4096}
mytool --password example-only --verbose
```

```output
{"apiKey": "<redacted>", "user": "shop", "max_tokens": 4096}
mytool --password <redacted> --verbose
```

### What stays

The rules leave the text readable. These stay as they are:

- the word "password" in a sentence, and a label such as `Password: stored as a hash`;
- the name of a variable with no value, such as `ANTHROPIC_API_KEY`;
- a placeholder, such as `${DB_PASSWORD}`, `<your password>` or `****`;
- code that reads a secret, such as `password = request.form["password"]`, and a type, such as `password: string;`;
- a commit hash, an id such as a GUID, a base64 test fixture, and a public key token;
- a database key, such as `PRIMARY_KEY=id`, and a count, such as `"max_tokens": 4096`.

### What the rules cannot promise

A secret in a shape that no rule knows can pass. For example, a password in a sentence, a password after `-p` for most commands, or a value whose name says nothing. Name each secret file with `--secret-file`. Then read the pack before you send it outside your team.

Read the sessions with care. A session holds the prompts and the model's answers, and they hold your code.

> [!NOTE]
> **For team leads:** the same rules guard each card that CodeLead writes on the board. They also guard your answers to CodeLead's questions in a session log.

## Details

- **Options**: `--since <COMMIT>`, the commit before the work started (the default is the parent of the first checkpoint commit); `--out <FOLDER>`; `--secret-file <PATH>`.
- **The program**: the Hand over stage runs it in the old application: `codelead modernize run audit`, or `/modernize --run audit`. Run `/modernize --audit-pack` in the rebuild's own chat too, for the rebuild's increments.
- **Waivers**: a security finding that you decided not to fix is recorded with [`/modernize --waive`](../chat-commands/modernize.md#--waive) in `docs/codelead/security-waivers.json`, and the pack lists it in `5-security/`.
- **The manifest**: `codelead.audit-pack-manifest/1`, with `head`, `since`, and each file's `path`, `bytes` and `sha256`.
- **The catalog**: the pack's `README.md` is added to the repository's document catalog, `docs/codelead/CATALOG.md`.

## What it is not

- **The audit pack is not a report that a model wrote**. CodeLead assembles it from records, with no model call.
- **The audit pack does not hide stops**. A refused or stopped request is kept with its reason, in the sessions' index.
- **The audit pack is not updated by itself**. Run it again after more work, and it is written again.

## Related content

- [The eight stages](the-eight-stages.md#8-hand-over)
- [`/modernize --audit-pack`](../chat-commands/modernize.md#--audit-pack)
- [Evidence records](../concepts/evidence-records.md) and [Evidence record format](../reference/evidence-record-format.md)
- [Sessions](../concepts/sessions.md)
- [Export a session](../how-to/export-a-session.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
