---
title: codelead board serve
description: Serve the board's page on this machine only, live as the files change, on port 7414 or a free port, until you stop it.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board serve

**Applies to:** CodeLead 0.1 and later

`codelead board serve` starts the board's page on your machine. Open the address that it prints in a browser. The page shows the board's files as they change, and writes them through the same rules as the commands.

## Name

`codelead board serve` - Serves the board's page on this machine.

## Synopsis

```console
codelead board serve [--port <PORT>] [--open] [--board <DIR>] [--json]
codelead board serve -h|--help
```

## Description

`serve` answers only on `127.0.0.1`, so only your machine can open the page. It tries port 7414 first, so that a bookmark keeps working. When that port is taken, it takes a free one and prints it. It runs until you press Ctrl+C, then prints `Pinboard stopped.`

The page writes as the board's first person in `board.json`, and the top bar shows that name. Changes that an agent or an editor makes to the files show on the page as they happen. A card file changed by hand gets its event by `unknown` once the file has been quiet for a moment.

Leave it running in a terminal for as long as you like, days included. If it stops, the same command starts it again.

> [!NOTE]
> **New to this?** In VS Code you do not need this command. The [Board view](../../extension/board-view.md) starts the page's server for you, and stops it with the window.

### Started by another program

When another program starts `serve` with a pipe as its input, `serve` stops when that pipe closes. The CodeLead extension for VS Code uses this, so a closed window takes its server with it. A terminal never stops it this way.

## Options

- **`--port <PORT>`**

  The port. The default is 7414, or a free port when 7414 is taken. `0` asks the system for a free port.

- **`--open`**

  Opens the page in your browser.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints one line, `{ "url", "port", "board", "person" }`, and nothing when it stops.

## Examples

- Start the page and open it yourself:

  ```console
  codelead board serve
  ```

  ```output
  Pinboard: http://127.0.0.1:7414/
  The board in docs/codelead/board, as Sam. The page writes the board's files through the same rules as these commands; Ctrl+C stops it.
  ```

  Press Ctrl+C to stop it:

  ```output
  Pinboard stopped.
  ```

- Start the page and open it in the browser:

  ```console
  codelead board serve --open
  ```

- Start it on a free port and read the address as JSON, as the VS Code extension does:

  ```console
  codelead board serve --port 0 --json
  ```

  ```output
  {"url":"http://127.0.0.1:63559/","port":63559,"board":"<your project>/docs/codelead/board","person":"sam"}
  ```

- Give a port that is not one:

  ```console
  codelead board serve --port 99999
  ```

  ```output
  "99999" is not a port: give a whole number from 1 to 65535, or 0 for a free one the system picks.
  ```

- Start it where there is no board:

  ```console
  codelead board serve
  ```

  ```output
  No board at <your project>/docs/codelead/board (no board.json there). Run `codelead board init` in the repository first.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The page was served, then stopped. |
| `2` | No board, a port that is not a number from 0 to 65535, or a port that cannot be used. |

## Files

`serve` reads every file of the board. It writes what the page asks for: cards, events and cursors, through the board's rules. It also writes `.local/preferences.json` when you choose a theme or a project. See [Files and events](../files-and-events.md).

> [!TIP]
> **For experienced users:** besides the board and its cards, the page's process answers `GET` and `PUT /api/preferences`, and `POST /api/cards/batch` for one change to several cards. The page loads nothing from the network.

## See also

- [Use the board](../use-the-board.md): what you do on the page.
- [Projects, themes and bulk operations](../projects-themes-and-bulk.md)
- [The Board view](../../extension/board-view.md): the page inside VS Code.
- [`codelead board init`](board-init.md): create the board first.
- [Pinboard overview](../index.md)
