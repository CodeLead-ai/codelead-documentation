---
title: Passwords
description: Where the Analyst, the Explorer and the replay read the passwords of the accounts they sign in as, how to set them, and where a password never goes.
kind: how-to
area: analyst
date: 2026-11-20
---

# Passwords

The Analyst signs in to an application as the roles that you name. This page says which commands need a password, where each one reads it, and how you set it. A password is never typed into a command line of the Analyst, and never written anywhere by it.

**In short**
- A role's password comes from the environment variable `CODELEAD_WALK_PASSWORD_<ROLE>`: the role's key in capitals. For the role `alpha`, it is `CODELEAD_WALK_PASSWORD_ALPHA`.
- The Explorer and the replay can also read a credentials file outside the folder where they run, which `CODELEAD_WALK_CREDENTIALS` names.
- In VS Code, the Settings page keeps the passwords in the project's `.env` file, and shows only **set** or **not set**.
- A password never goes into a recording, a journeys file, a log or a report. No Analyst command calls a model, so no password reaches a prompt.

## Before you begin

- You know the roles that you will use, and the account of each one. See [Roles, journeys and steps](concepts.md#roles-and-accounts).
- You have the passwords from the application's owner. Use test accounts where you can.

> [!NOTE]
> **New to this?** An *environment variable* is a named value that a program reads when it starts. Setting a password there keeps it out of commands, files and screenshots.

## Which commands need a password

| Command | How users sign in | Password needed |
|---|---|---|
| [`record`](reference/analyst-record.md) | Its own sign-in form | None. You type it in the role's window. The recorder masks it. |
| `record` | Windows sign-in, as this computer's account | None. The browser uses this computer's own session. |
| `record` | Windows sign-in, as another account | `CODELEAD_WALK_PASSWORD_ALPHA`, or typed in the control window's setup |
| [`explore`](reference/analyst-explore.md) | A visitor | None |
| `explore` | Given accounts | `CODELEAD_WALK_PASSWORD_<ROLE>` for each role that `--as` names, or the credentials file |
| `explore` | Windows sign-in | Optional: `CODELEAD_WALK_PASSWORD_ALPHA`, or the credentials file. Without it, this computer's session. |
| [`replay`](reference/analyst-replay.md) | New users | None. The walk makes up its own user names and passwords. |
| `replay` | Given accounts | `CODELEAD_WALK_PASSWORD_<USER>`, or the credentials file |
| `replay` | Windows sign-in | Optional, from the same places. Without it, this computer's session. |
| [`draft`](reference/analyst-draft.md), [`check`](reference/analyst-check.md), [`rules`](reference/analyst-rules.md), [`facts`](reference/analyst-facts.md) | Any | None. They read files only. |

A wrong-password step needs the account's password too. The walk builds a wrong password that is as long as the account's password and never equal to it.

## Name the variable

The variable's name is `CODELEAD_WALK_PASSWORD_` and the role's key in capitals:

| Role key | Variable |
|---|---|
| `alpha` | `CODELEAD_WALK_PASSWORD_ALPHA` |
| `clerk` | `CODELEAD_WALK_PASSWORD_CLERK` |
| `night_shift` | `CODELEAD_WALK_PASSWORD_NIGHT_SHIFT` |

A role's key starts with a letter (A to Z, a to z), then letters, digits or underscores. `record`, `explore`, `replay` and `check` use this one rule. Two keys that differ only in case, such as `Clerk` and `clerk`, read the same variable. So the Explorer refuses two `--as` roles with such keys. Under Windows sign-in, the one role is `alpha` unless you name another with `--role`.

## Set a password in the terminal

Set the variable in the terminal where you run the command. The value stays in that terminal only. On macOS or Linux:

```console
export CODELEAD_WALK_PASSWORD_ALPHA='<ALPHA_PASSWORD>'
codelead analyst explore --base http://localhost:5080/ --given-accounts --as alpha=store.clerk
```

On Windows, in PowerShell:

```console
$env:CODELEAD_WALK_PASSWORD_ALPHA = '<ALPHA_PASSWORD>'
codelead analyst explore --base http://localhost:5080/ --given-accounts --as alpha=store.clerk
```

> [!TIP]
> **For experienced users:** put a space before the command in bash or zsh, when your shell is set to keep such commands out of its history. Or read the value from your password manager's command line into the variable.

## Set a password in VS Code

1. In the [Analyst view](../extension/analyst-view.md), select the gear. The Settings page opens at **Analyst and Explorer**.
2. Under **Sign-in passwords**, find the role, or select **Add a role** and type its key.
3. Type the password, and select **Save**.

The row says **set** or **not set**, and the name of its variable. It never shows the value. **Save** writes `CODELEAD_WALK_PASSWORD_<ROLE>` to the project's `.env` file. **Remove** deletes it. See [Settings](../extension/settings.md#analyst-and-explorer).

`codelead analyst` reads the `.env` file of the folder where it runs, when it starts. A variable that is already set in the terminal wins over the same variable in `.env`.

> [!WARNING]
> `.env` holds the passwords in plain text. [`codelead init`](../cli/codelead-init.md) adds `.env` to `.gitignore`. Do not commit it, and do not send it with a session export.

## Use a credentials file

The Explorer and the replay can read the passwords from a JSON file instead. The file maps each **account**, not each role, to its password:

```json
{
  "store.clerk": "<ALPHA_PASSWORD>",
  "CORP\\store.admin": "<ADMIN_PASSWORD>"
}
```

Name the file with `CODELEAD_WALK_CREDENTIALS`. The file must be outside the repository and outside the folder where you run the command, where nothing can commit it:

```console
export CODELEAD_WALK_CREDENTIALS="$HOME/.config/basic-store/walk-credentials.json"
```

- A file inside the folder is refused before a browser opens, with the exit code 2. The Explorer says:

  ```output
  CODELEAD_WALK_CREDENTIALS names a file inside the folder the Explorer runs in: the passwords are kept in a file outside it, where nothing commits them. Name with CODELEAD_WALK_CREDENTIALS a file outside this folder (or set CODELEAD_WALK_PASSWORD_<ROLE>, the role in capitals), then explore again. Nothing was explored.
  ```

- The walk finds an account by its name as written, then by its name in any case, as Windows compares names.
- `CODELEAD_WALK_PASSWORD_<USER>` wins over the file when both hold a password.
- The Explorer reads the file in the same way as the replay.

## Where a password never goes

| Place | What CodeLead does |
|---|---|
| The recording, `recording.md` and `index.json` | A typed password is written as a placeholder, such as `{alpha.password}`, with `"masked": true`. Its user name is written `{alpha.name}`. |
| The journeys file | Under given accounts, a password field holds only `{alpha.password}` or `{alpha.wrongPassword}`. The walk refuses a journeys file that holds a password. |
| The terminal and the logs | The recorder, the Explorer and the walk replace every spelling of the password, and the HTTP credentials made from it, with the placeholder. |
| The site map, the forms, the stories, the rules and the facts | They are made from the masked recordings, so they hold the placeholder only. |
| A prompt | No Analyst command calls a model. Nothing is sent anywhere, except the browser's own requests to the application. |

The recorder does not even keep a hash or the length of a password. It keeps only one word beside a masked value: `"own"` when the user signed in with that password in the recording, `"other"` when not.

> [!CAUTION]
> A screenshot shows the page as it was. A password field shows only dots, but a user name that the page displays is in the screenshot.

## When a password is missing

The Explorer reads every password before a browser opens. It names each role that has none, writes nothing, and exits with code 2. The message also says whether `CODELEAD_WALK_CREDENTIALS` is set (here it is not):

```output
The explorer signs in as each account with its password, read from CODELEAD_WALK_PASSWORD_<ROLE> (the role in capitals), else the file CODELEAD_WALK_CREDENTIALS names (each account mapped to its password; not set here), and has none for these: CODELEAD_WALK_PASSWORD_CLERK (clerk, store.clerk), CODELEAD_WALK_PASSWORD_ADMIN (admin, store.admin). Set them in this terminal, or name with CODELEAD_WALK_CREDENTIALS a file outside this folder that maps each account to its password (the explorer reads a password, and never writes or prints one), then explore again. Nothing was explored.
```

The replay says the same before it walks anything, and exits with code 2:

```output
Nothing was walked: the walk signs in as the accounts it is given, each with its password, and has none for clerk (store.clerk): set CODELEAD_WALK_PASSWORD_CLERK, or name with CODELEAD_WALK_CREDENTIALS a file outside the repository that maps each account to its password.
```

Under Windows sign-in, a missing password is not an error. The browser signs in with this computer's session. When you named another account, the recorder and the Explorer say which account the browser signs in as instead.

## Check that it worked

- In VS Code, the role's row says **set**.
- In a terminal, check that the variable exists without printing it:

  ```console
  printenv CODELEAD_WALK_PASSWORD_ALPHA > /dev/null && echo set
  ```

  ```output
  set
  ```

- The Explorer starts and prints `signed in` lines, or the replay passes its sign-in step.

## Details

- **The variables**:

  | Variable | Read by | Holds |
  |---|---|---|
  | `CODELEAD_WALK_PASSWORD_<ROLE>` | `explore`, `replay`, `record` under Windows sign-in, and [`codelead walk`](../cli/codelead-walk.md) | The password of one role or user |
  | `CODELEAD_WALK_CREDENTIALS` | `explore`, `replay` and `codelead walk` | The path of a JSON file outside the repository, from account to password |

- **The control window's setup**: a password typed there for Windows sign-in travels only to the recorder on this machine. The recorder keeps it in memory for the session, and empties the field once the session starts.
- **A continued recording** under Windows sign-in needs the recording's own account. On a computer signed in as another account, set that role's variable before `record --continue`.
- See [Environment variables](../reference/environment-variables.md) for every variable that CodeLead reads.

## See also

- [Explore an application](explore-an-application.md)
- [Record a journey](record-a-journey.md)
- [From a recording to a specification](from-recording-to-spec.md)
- [The Analyst view](../extension/analyst-view.md)
- [Settings](../extension/settings.md#analyst-and-explorer)
- [The walk and parity](../modernization/walk-and-parity.md)
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
