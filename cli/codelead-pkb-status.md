---
title: codelead pkb status
description: See what runs for the PKB service, in words. The service, the store, the chat and embeddings servers, paused and the projects.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb status

**Applies to:** CodeLead 0.1 and later

`codelead pkb status` says what runs for the [PKB service](codelead-pkb.md). It names the service when one is named, and the store with its search process. It says whether the chat and embeddings servers answer, and whether `pkb.json` holds every model call. It ends with the projects and their counts.

## Name

`codelead pkb status` - Says what runs: the service, the store, the endpoints, `paused` and the projects.

## Synopsis

```console
codelead pkb status [--dir <FOLDER>] [--json]
codelead pkb status -h|--help
```

## Description

`status` goes through the service that `CODELEAD_PKB_URL` or `pkb.url` names, as the other commands do. See [How the commands find the service](codelead-pkb-serve.md#how-the-commands-find-the-service). Without a service, or with `--dir`, it reads the store on this machine itself.

| Line | What it says |
|---|---|
| The service | Where it listens, whether only this machine reaches it, and whether each request needs the shared key. |
| The store | Whether MongoDB answers, its database, and whether its search process (mongot) answers. Or a store in memory. |
| Chat | The chat server, its model, and whether it answers. |
| Embeddings | The embeddings server, its model, and whether it answers. |
| Paused | Whether `pkb.json` says `paused`. Then no model is called. |
| Projects | The projects of `pkb.json`, then each project in the store with its counts. |

`status` never asks a model to run. It asks a server only for its list of models, for at most 2 seconds. It asks nothing while `paused` holds, and it never asks a remote server.

## Options

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. `status` then reads the store on this machine, not the service. The default is `<home>/.codelead/pkb`, or the folder of `CODELEAD_PKB_HOME`.

- **`--json`**

  Prints the result as one JSON document: `serving` (through the service), `store`, `chat`, `embeddings`, `paused`, `configured_projects` and `projects`.

## Examples

These examples use the sample project `tally`, through a service on port `7425`. Its chat server is not running.

- See what runs through the service:

  ```console
  codelead pkb status
  ```

  ```output
  The PKB service answers at http://127.0.0.1:7425: it listens on 127.0.0.1:7425 (this machine only), and needs no shared key.
  The store: in memory, in the service itself. Nothing is kept when the service stops.
  Chat: http://localhost:18080/v1, model a-chat-model: does not answer (ECONNREFUSED).
  Embeddings: http://localhost:18080/v1, no model named (embeddings.model in pkb.json).
  Paused: no.
  Projects of pkb.json: tally.
  - codelead-shared in the store: 3 entries (3 reported), 1 source, 0 evidence records, 3 chunks, 0 questions.
  - tally in the store: 14 entries (9 evidence-backed, 1 confirmed, 3 reported, 1 downgraded), 12 sources, 2 evidence records, 30 chunks, 1 question.
  ```

- Read the store on this machine, without the service. Here the store is not set up, so the exit code is `1`:

  ```console
  codelead pkb status --dir ./pkb
  ```

  ```output
  No PKB service is named (CODELEAD_PKB_URL, or pkb.url in .codelead/config.json): the commands use the store on this machine.
  The store: MongoDB does not answer (the PKB store is not set up). Start it with codelead pkb store up.
  Chat: http://localhost:18080/v1, model a-chat-model: does not answer (ECONNREFUSED).
  Embeddings: http://localhost:18080/v1, no model named (embeddings.model in pkb.json).
  Paused: no.
  Projects of pkb.json: tally.
  ```

- A service that needs the shared key, asked without it. The exit code is `2`:

  ```output
  Refused: this service needs its shared key, and the request did not send it (or sent another).
  Set CODELEAD_PKB_KEY to the key of the service's .env (PKB_SHARED_KEY), then run this again.
  ```

- A service named that does not run. The exit code is `1`:

  <!-- described: the words of the other commands, captured with codelead pkb entries -->
  ```output
  The PKB service at http://127.0.0.1:7499 (named by CODELEAD_PKB_URL) does not answer (ECONNREFUSED).
  Start it with codelead pkb serve on the machine that holds the store, then run this again. To use a store on this machine instead, name its folder with --dir.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The store answers, and for MongoDB its search process answers too. |
| `1` | The store, its search process or the service named does not answer. |
| `2` | Refused: no `pkb.json`, a problem in the settings that name the service, or the shared key. |

## See also

- [`codelead pkb serve`](codelead-pkb-serve.md): run the service.
- [`codelead pkb store`](codelead-pkb-store.md): `store status` says what the containers of the store do.
- [PKB service API](../reference/pkb-service-api.md): `GET /api/health`, what `status` reads.
- [`codelead pkb`](codelead-pkb.md)
