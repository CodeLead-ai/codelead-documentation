---
title: codelead pkb serve
description: Run the PKB service, so that people and agents ask, list and change a project's knowledge through one address.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb serve

**Applies to:** CodeLead 0.1 and later

`codelead pkb serve` runs the **PKB service**: a small web server over the store of the [PKB](codelead-pkb.md). The other `codelead pkb` commands then go through it, and so can a script or a page. The service listens on `127.0.0.1` by default, so only this machine reaches it. It refuses every write that comes from another site.

> [!IMPORTANT]
> Start the store first with [`codelead pkb store up`](codelead-pkb-store.md), or use `--memory` for a trial. The service never runs code and never writes into a repository. It calls a model only to answer and to make embeddings, under the same rules as the commands.

## Name

`codelead pkb serve` - Runs the PKB service: its routes over the store, and its stream of events.

## Synopsis

```console
codelead pkb serve [--dir <FOLDER>] [--memory] [--static <FOLDER>] [--json]
codelead pkb serve -h|--help
```

## Description

`serve` reads `pkb.json` in the PKB's folder. It listens on the address of its `bind` key and the port of its `port` key. The defaults are `127.0.0.1` and `7415`. Then it waits for requests until you stop it with Ctrl+C.

Each route of the service calls the same code as the command of the same name. So `codelead pkb ask` through the service prints what it prints without the service. The routes are listed in the [PKB service API](../reference/pkb-service-api.md).

> [!NOTE]
> **New to this?** A service is a program that keeps running and answers requests. You start it once. Then each command sends its request to the service, instead of opening the store itself.

### How the commands find the service

A command goes through the service when one of these names it:

- the environment variable `CODELEAD_PKB_URL`, such as `http://127.0.0.1:7415`;
- the `pkb.url` key of the workspace's `.codelead/config.json`.

The `pkb.project` key of the same file names the project that the commands ask. `--dir` on a command keeps it on the store of that folder, without the service. When the service named does not answer, the command says so and stops. It never tries another store by itself.

```json
{ "pkb": { "url": "http://127.0.0.1:7415", "project": "tally" } }
```

### What the service refuses

The service guards every request in this order:

1. **The address.** A request must name the service's own address in its `Host` header. On `127.0.0.1`, that is `127.0.0.1`, `localhost` or `[::1]` with the port. A page of another site that reaches the service through a name of its own is refused.
2. **The preflight.** A browser asks first before it sends a request from another site. The service refuses that question, so the browser never gets permission.
3. **The origin.** A request from a browser names its site in its `Origin` header. Only the service's own site may use the routes. A request that a browser marks as coming from another site is refused too.
4. **The shared key.** When a key is set, every route needs it. See [The shared key](#the-shared-key).
5. **The body of a write.** A write must be JSON, with the header `Content-Type: application/json`. A plain form of another site cannot send that. The body holds at most 2 MB, or 32 MB for an import.

A refused write writes nothing. Each refusal answers with a code, a message and the way out.

### The shared key

A key is needed when the service listens on the team's network. Then `bind` holds the machine's address, such as `192.168.1.20`, or `0.0.0.0`. Without a key, `serve` refuses to start.

1. Make a key of at least 32 random characters, such as the output of `openssl rand -hex 32`.
2. Put it in the PKB folder's `.env` file, as `PKB_SHARED_KEY=<key>`. The store's passwords live in that file too.
3. Give the key to each client, as `CODELEAD_PKB_KEY`. Put it in the client's environment, or in the `.env` file of the client's workspace.

A client sends the key in the header `Authorization: Bearer <key>`. The service never prints the key, never logs it and never answers with it. It never listens on a public address, with or without a key.

> [!TIP]
> **For experienced users:** on the network, the clients may reach the service by a name, such as `pkb.lan`. List each name in the `hosts` key of `pkb.json`, or the service refuses the request.

### Time and load

- A request must arrive in 60 seconds, and its headers in 10 seconds.
- A route answers in 60 seconds. A route that may call a model answers in 600 seconds.
- At most two requests that may call a model run at once. A third one is told that the service is busy.
- A route that takes too long answers with the code `timeout`. Its work goes on, and an answer may still be stored.

So a model server that does not answer never stops the other routes.

### The page's folder

`--static <FOLDER>` also serves the files of a built page from that folder, such as a web app. It is off by default. The page of the PKB service comes in a later release. Without `--static`, the address `/` answers that the service serves no page.

## Options

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json` and the `.env` file. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--memory`**

  Serves a store in memory instead of the MongoDB store. Nothing is kept when the service stops. Use it for a trial, or to check an export by importing it. Sync or import a project into it first.

- **`--static <FOLDER>`**

  Also serves the built files of a page from this folder. Only files inside the folder are served.

- **`--json`**

  Prints where the service listens as one JSON document, then keeps running.

## Examples

These examples use a small sample project, `tally`, an invoicing application. Its `pkb.json` sets the port to `7425`. Its chat server is not running.

- Start the service over a store in memory:

  ```console
  codelead pkb serve --dir ./pkb --memory
  ```

  ```output
  The PKB service listens at http://127.0.0.1:7425 (this machine only).
  The store: in memory, in the service itself. Nothing is kept when it stops: sync or import a project into it.
  A write from another site is refused. The routes: codelead pkb status, or GET /api/health.
  Stop it with Ctrl+C.
  ```

- In a second terminal, sync the project through the service, and see what runs:

  ```console
  export CODELEAD_PKB_URL=http://127.0.0.1:7425
  codelead pkb sync --by person-a
  codelead pkb status
  ```

  See [`codelead pkb status`](codelead-pkb-status.md#examples) for the result.

- With a key in the `.env` file, the first line says that each request needs it:

  ```output
  The PKB service listens at http://127.0.0.1:7427 (this machine only; every request must send the shared key).
  ```

- Listen on the team's network without a key. The service does not start, and the exit code is `2`:

  ```console
  codelead pkb serve --dir ./net --memory
  ```

  ```output
  pkb.json's bind is 192.168.1.20: on the network, the service needs a shared key, and the PKB's folder holds none.
  Put a key of at least 32 random characters in the PKB folder's .env as PKB_SHARED_KEY=<key> (openssl rand -hex 32 makes one), then give it to each client as CODELEAD_PKB_KEY. Or set bind to 127.0.0.1.
  Nothing was started.
  ```

- Send a write from another site with `curl`. The service refuses it, and nothing is written:

  ```console
  curl -s -X POST http://127.0.0.1:7425/api/entries/e-ffa9d7b67c42d3734e22/downgrade \
    -H "Origin: http://evil.example" -H "Content-Type: application/json" \
    -d '{"by":"someone","reason":"a page of another site"}'
  ```

  ```output
  {"ok":false,"refused":"Refused: a request from another site (its Origin is another site's); nothing was written. Call the service from the command line, or from its own page.","error":{"code":"cross_site","message":"Refused: a request from another site (its Origin is another site's); nothing was written.","wayOut":"Call the service from the command line, or from its own page."},"words":"Refused: a request from another site (its Origin is another site's); nothing was written.\nCall the service from the command line, or from its own page.","exit":2}
  ```

  The status is `403`. See the [PKB service API](../reference/pkb-service-api.md#the-guards) for the other refusals.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The service ran and was stopped. |
| `2` | It did not start: no `pkb.json`, a bind on the network without the shared key, a public address, the store does not answer, or the port is taken. |

## Files

| File | What `serve` does with it |
|---|---|
| `<home>/.codelead/pkb/pkb.json` | Reads `bind`, `port`, `hosts` and the projects when it starts. Each route reads it again, so a change of `paused` holds at once. |
| `<home>/.codelead/pkb/.env` | Reads the store's password and the shared key, `PKB_SHARED_KEY`. |

The service writes one line to standard error for each request: the method, the path, the status and the time. The line never holds the body, the key or a question's words.

## Details

- Every answer is JSON, with the command's words in `words` and its exit code in `exit`. A refusal or a failure adds `error`, with `code`, `message` and `wayOut`.
- No answer holds a path of this machine. A last scan replaces the PKB's folder, the checkouts, the home folder and the temporary folder in every answer.
- No answer carries an `Access-Control-Allow-Origin` header.
- The service keeps one connection to the store, and closes it when it stops.

## See also

- [PKB service API](../reference/pkb-service-api.md): the routes, what each one takes and answers, and the guards.
- [`codelead pkb status`](codelead-pkb-status.md): what the service and the store say.
- [`codelead pkb store`](codelead-pkb-store.md): start the store that the service serves.
- [`codelead pkb`](codelead-pkb.md): the commands of the PKB service.
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
