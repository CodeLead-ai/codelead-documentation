---
title: /help
description: List every chat command with one line each, or show the usage lines of one command.
kind: reference
area: chat-commands
date: 2026-11-20
---

# /help

**Applies to:** CodeLead 0.1 and later

## Name

`/help` - Lists every chat command, or shows the usage lines of one command.

## Synopsis

```console
/help
/help <NAME>
/help --command <NAME>
```

## Description

`/help` prints a box with one line for each form of each command, then a paragraph about the chat. `/help <NAME>` prints the usage lines of one command. The name can start with `/` or not: `/help validate` and `/help /validate` are the same.

The chat in Visual Studio Code reads the same table for its completion: when you type `/`, it offers the commands and their options. [`/status`](status.md), [`/exit`](exit.md) and [`/quit`](quit.md) are in the table too.

> [!NOTE]
> **New to this?** When you do not remember a command, type `/help`. When you remember the command but not its options, type `/help` and its name.

## Arguments

- **`<NAME>`**

  The name of one command, for example `validate`, `implement` or `create-app`.

## Options

- **`--command <NAME>`**

  The same as `/help <NAME>`.

## What you see

`/help` is informational: its output is the answer, so its result has no headline. A name that `/help` does not know prints `Unknown command for help: <NAME>`.

## Examples

- List every command:

  ```console
  /help
  ```

  ```output
  ╭──────────────────────────────────────────────────────────────────────────────╮
  │                                                                              │
  │   CodeLead Interactive Commands                                              │
  │                                                                              │
  │   <question>             Chat about this repository and CodeLead: how the    │
  │   code works, what CodeLead did to it and why, how to use its commands.      │
  │   Text without a leading / goes here. The chat never changes code: a         │
  │   change request is answered with the command to run.                        │
  │   /chat <question>       The same, explicitly; /chat --new starts the        │
  │   conversation over.                                                         │
  │   /ask <question>        The same as /chat <question>.                       │
  │   /help                  Show this help.                                     │
  …
  │   /map [--refresh]       Read the repository into its map                    │
  │   (docs/codelead/repository-map.md); --refresh rewrites it even when         │
  │   fresh.                                                                     │
  │   /exit, /quit           Exit CodeLead.                                      │
  │                                                                              │
  │   Chat:                                                                      │
  │   Type a question without a leading / and CodeLead's chat answers it with    │
  │   the smartest model configured (the run profile's intervention model,       │
  │   else the deep model; the cost is shown): how this repository's code        │
  …
  ╰──────────────────────────────────────────────────────────────────────────────╯
  ```

- Show the usage lines of one command:

  ```console
  /help validate
  ```

  ```output
  Help: /validate

  Usage:
    /validate
    /validate --scope repo
    /validate --scope current
    /validate --scope <path>
    /validate --override
    /validate --mark-passed
    /validate --repair
  ```

- The same with `--command`:

  ```console
  /help --command implement
  ```

  ```output
  Help: /implement

  Usage:
    /implement <request>
    /implement --from-clipboard
    /implement --force <request>
    /implement --lint <request>
    /implement --card <id>
    /implement #<id>
    /implement --spec <file> <request>  (--spec <file>  walk this spec on the running application after the landing (docs/codelead/specs/<name>.md); repeatable; the application is run from .codelead/run.json or the rebuild's run block)
    /implement --spec <file> --card <id>  (the flag wins over the card's spec attachments)
  ```

- Every command has help, with every form that it takes. For example, `/pr`:

  ```console
  /help pr
  ```

  ```output
  Help: /pr

  Usage:
    /pr
    /pr --draft
    /pr --base <branch>
    /pr --evidence <record id>
    /pr --title <text>
    /pr --dry-run
  ```

  `/help modernize` lists every form of `/modernize` in the same way. This reference has the full page: [`/modernize`](modernize.md).

- A name that is no command:

  ```console
  /help nope
  ```

  ```output
  Unknown command for help: nope
  ```

## Files

`/help` reads no file and writes none.

## See also

- [Chat command reference](index.md)
- [The chat](../extension/chat.md)
- [`codelead interactive`](../cli/codelead-interactive.md)
