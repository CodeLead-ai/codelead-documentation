---
title: codelead analyst record
description: Record a running web application while a person uses it - the control window, roles, journeys, Expect this - into recording.json, recording.md and a screenshot for each step.
kind: reference
area: analyst
date: 2026-11-20
---

# codelead analyst record

**Applies to:** CodeLead 0.1 and later

`codelead analyst record` opens a control window and a visible browser. A person uses the application, and the Analyst cuts each journey into steps and records each step. The command calls no model.

## Name

`codelead analyst record` - Records a running web application while a person uses it.

## Synopsis

```console
codelead analyst record [--base <URL>] [--app <NAME>] [--sign-in form|windows] [--given-accounts]
                        [--integrated <ACCOUNT>] [--role <KEY>] [--api <ORIGIN>] [--out <FOLDER>]
                        [--keep-query] [--bar] [--no-window]
codelead analyst record --continue <FOLDER> [--role <KEY>] [--bar] [--no-window]
codelead analyst help record
```

## Description

The command needs a person and a screen. It opens Playwright's Chromium, so CodeLead must be installed with it. The folder must have the pro plan. See [`codelead analyst`](../../cli/codelead-analyst.md).

### The control window

The **control window** is a small window of its own, beside the browser. It drives the whole session. It is served on `127.0.0.1` only, under a random key in its address, and nothing done in it is recorded.

With no options, the control window opens on its **setup**: the application's address, its name, and how it signs its users in. The session starts when you select **Start session**. The options fill the setup in advance. When they say everything, the session starts at once: an address, a sign-in mode, and under Windows sign-in an account.

With `--no-window`, the terminal asks what is missing instead: the address, then `form` or `windows`, then the account under Windows sign-in.

### Roles, journeys and steps

1. **Start role** opens a window for a role, at the application's address. Each role has its own cookies. Its window's title begins with the role's name.
2. **Start journey** names a journey and takes the role's window to the first page.
3. The person uses the application. The recorder cuts a step at four points. The page goes to another address, a write is sent and answered, a dialog opens or closes, or a form is sent.
4. **Expect this** marks a text on the page that the step must show. **End journey** ends the journey.
5. **Finish session** ends the open journey, writes the files and closes the browser.

The control window then stays open for **Draft journeys**, **Check** and **Replay** on the recording. Close it, or press Enter in the terminal, to end the command.

In a role's window, **Alt+Shift+S** (**Option+Shift+S** on a Mac) starts or ends a journey, and **Alt+Shift+E** is **Expect this**. In the terminal, `role <KEY>` starts a role, a line of text starts a journey with that name, an empty line ends it, and `finish` ends the session.

See [Record a journey](../record-a-journey.md) for each control.

### What is captured

The recording keeps each action and the values typed into ordinary fields. It keeps the requests that the pages send, with their statuses, the headings and alerts after each step, and a screenshot. An address can be refused (401, 403, or a redirect to the sign-in page) or fail (404, a server error, no answer). Then its `navigate` action keeps the status. A value typed into a password field is written as a placeholder, such as `{clerk.password}`. So is a value typed into a field that says PIN, secret, token or key. Request bodies, headers and cookies are never kept. See [Passwords](../passwords.md).

### Continue a recording

`--continue <FOLDER>` reopens a recording. The application, the address, the sign-in mode, the roles and the journeys are the recording's own, and new steps join the same `recording.json`. **Continue journey** replays a saved journey with the walk, then records on from where it ended.

## Options

- **`--base <URL>`**

  The application's address, `http` or `https`. Without it, the control window asks it, or the terminal with `--no-window`. With `--continue`, the recording's own address is used.

- **`--app <NAME>`**

  What you call the application, for example `"Basic Store"`. It is optional.

- **`--sign-in <MODE>`**

  How the application signs its users in. `form`: its own sign-in pages, with many roles, each in its own window. `windows`: the browser's own session, Windows authentication, with one role. Without it, the control window asks.

- **`--given-accounts`**

  The application's users exist already, for example in a directory or a single sign-on. Each user's sign-in name is kept as its account, and a replay signs in as it.

- **`--integrated <ACCOUNT>`**

  Windows sign-in as this account, for example `'CORP\store.clerk'`. With the password in `CODELEAD_WALK_PASSWORD_<ROLE>`, the browser answers the application's challenge as that account. With no password, it signs in with this computer's session, and says so when that is another account. The default role is `alpha`, so the variable is `CODELEAD_WALK_PASSWORD_ALPHA`. The password is read, never written or printed.

- **`--role <KEY>`**

  A role to start at once, with its window open from the start. A key starts with a letter (A to Z, a to z), then letters, digits or underscores. Under Windows sign-in, it names the one role. The default there is `alpha`.

- **`--user <KEY>`**

  The earlier name of `--role`.

- **`--api <ORIGIN>`**

  The origin of the application's server, when it is not the address's origin. The requests to it are the ones listed.

- **`--out <FOLDER>`**

  Where the recording is written. The default is `./analyst-recordings/<TIME>/`.

- **`--keep-query`**

  Keeps the query part of addresses, so that the draft asserts it. Without it, the recording says where it dropped a query.

- **`--continue <FOLDER>`**

  Reopens the recording in this folder, the one that holds `recording.json`. A flag that says something else than the recording, such as another `--base`, is refused.

- **`--bar`**

  Also shows the Analyst's bar along the bottom of each role's window. It is off by default, because the control window drives the session.

- **`--no-window`**

  No control window. The terminal, the keys, and the bar with `--bar`, drive the session.

## Examples

- Start the recorder, and fill the setup in the control window:

  ```console
  codelead analyst record
  ```

- Record the Basic Store sample with its own sign-in pages. The session starts at once:

  ```console
  codelead analyst record --base http://localhost:5080/ --app "Basic Store" --sign-in form
  ```

  <!-- described, not captured -->
  ```output
  The control window's page: http://127.0.0.1:53237/2MlrZ2…/ (served on this machine alone; its address holds its key).
  Form sign-in: each role signs in through the application's own pages, in a window of its own with its own cookies.
  …
  Recorded 9 steps into analyst-recordings/2026-11-20T09-30-00Z: recording.json, recording.md and the screenshots under steps/.
  Next: codelead analyst draft analyst-recordings/2026-11-20T09-30-00Z/recording.json
  ```

  The last line names the command that you ran, so you can run it as it is.

- Record an application behind Windows authentication, as this computer's account:

  ```console
  codelead analyst record --base http://localhost:5080/ --integrated 'CORP\store.clerk'
  ```

- Add journeys to a recording made before:

  ```console
  codelead analyst record --continue analyst-recordings/2026-11-20T09-30-00Z
  ```

- Give a sign-in mode that does not exist. Nothing opens:

  ```console
  codelead analyst record --sign-in kerberos
  ```

  ```output
  --sign-in is form or windows, not "kerberos".
  ```

- Give a role key with a space:

  ```console
  codelead analyst record --role "two words"
  ```

  ```output
  --role: a role's name starts with a letter (A to Z), then letters, digits or underscores, such as manager or clerk_2, and "two words" is not one.
  ```

- Give a role key that starts with a digit:

  ```console
  codelead analyst record --role 1clerk
  ```

  ```output
  --role: a role's name starts with a letter (A to Z), then letters, digits or underscores, such as manager or clerk_2, and "1clerk" does not start with a letter.
  ```

- Give an account for Windows sign-in with the form sign-in mode:

  ```console
  codelead analyst record --sign-in form --integrated 'CORP\store.clerk'
  ```

  ```output
  --integrated names the account of Windows sign-in, and --sign-in form says the application's own sign-in pages: give one or the other.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The recording was written, and the control window was closed. |
| `1` | The recording stopped on an error, such as Chromium not installed. Also an option that the command does not know, and a folder without the pro plan. |
| `2` | Refused before anything was recorded: an option with a wrong value, a setup that was given up, or a folder that `--continue` cannot reopen, such as one whose application does not answer. |

## Files

| File | What `record` does with it |
|---|---|
| `analyst-recordings/<TIME>/recording.json` | Writes the recording, `codelead.recording/1`, after every step. |
| `analyst-recordings/<TIME>/recording.md` | Writes the same recording, to read. |
| `analyst-recordings/<TIME>/index.json` | Writes the roles and journeys of the session, for `--continue`. |
| `analyst-recordings/<TIME>/steps/s01.png` | Writes a screenshot when each step ends. |
| `.env` | Reads `CODELEAD_WALK_PASSWORD_<ROLE>` under Windows sign-in, when the variable is not set in the terminal. |

See [Files the Analyst writes](../files.md).

## See also

- [Record a journey](../record-a-journey.md)
- [`codelead analyst draft`](analyst-draft.md): the next command.
- [`codelead analyst explore`](analyst-explore.md): explore without a person.
- [Roles, journeys and steps](../concepts.md)
- [Passwords](../passwords.md)
- [The Analyst view](../../extension/analyst-view.md)
- [Tutorial: record journeys with the Analyst](../../tutorials/record-journeys-with-the-analyst.md)
