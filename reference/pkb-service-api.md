---
title: PKB service API
description: Every route of the PKB service, what it takes and answers, its stream of events, and the guards that refuse a request.
kind: reference
area: reference
date: 2026-11-24
---

# PKB service API

**Applies to:** CodeLead 0.1 and later

[`codelead pkb serve`](../cli/codelead-pkb-serve.md) runs the PKB service. The service answers HTTP requests under `/api/`, in JSON, on `127.0.0.1:7415` by default. Each route calls the same code as a `codelead pkb` command. This page lists the routes, the shape of every answer, the stream of events and the guards.

## Every answer

Each answer is one JSON object. It holds the JSON that the command prints with `--json`, and two fields more:

| Field | What it holds |
|---|---|
| `words` | What the command prints without `--json`, line by line. |
| `exit` | The command's exit code: `0` done, `1` it started and failed, `2` refused. |

A refusal or a failure also holds `error`, and its HTTP status says the kind:

```json
{ "error": { "code": "not_found", "message": "The PKB store holds no entry e-none: codelead pkb entries lists a project's entries with their ids.", "wayOut": "codelead pkb entries lists a project's entries with their ids." } }
```

| `error.code` | Status | What it means |
|---|---|---|
| `invalid` | 400 | The request lacks something, or holds a wrong value or a field that the route does not take. |
| `key` | 401 | The shared key is needed, and the request did not send it. |
| `cross_site`, `host`, `consent` | 403 | A request from another site, a `Host` that is not the service's, or the remote rule. |
| `not_found`, `route` | 404 | No such project, entry, question, answer or chunk, or no such route. |
| `method` | 405 | The route answers another method. |
| `not_configured`, `refused`, `conflict` | 409 | `pkb.json` lacks something, a rule refuses it, or another sync runs. |
| `too_large` | 413 | The body is larger than the limit. |
| `media_type` | 415 | A write that is not JSON. |
| `failed` | 500 | It started and failed. |
| `upstream` | 502 | The chat or embeddings server did not answer. |
| `paused`, `unavailable`, `busy` | 503 | `pkb.json` says `paused`, the store does not answer, or the service is busy. |
| `timeout` | 504 | It took longer than the deadline. Its work goes on. |

No answer holds a secret, a password, a key or a path of this machine. A file is named as `<repository>:<path>`.

## The routes

| Route | What it does | The command |
|---|---|---|
| `GET /api/health` | What runs: the service, the store, the endpoints, `paused`, the projects. Always `200`. | [`status`](../cli/codelead-pkb-status.md) |
| `GET /api/projects` | The projects in the store, with their counts, and whether `pkb.json` names them. | |
| `GET /api/projects/<p>/entries` | The ledger. Takes `type`, `trust`, `stale=1` and `q` in the query. | [`entries`](../cli/codelead-pkb-entries.md) |
| `GET /api/entries/<id>` | One entry, with its history and the stored answers that cite it (`cited_by`). | [`entries <id>`](../cli/codelead-pkb-entries.md) |
| `POST /api/entries/<id>/confirm` | A person's confirm. Body: `{ "by", "reason" }`. | [`confirm`](../cli/codelead-pkb-confirm.md) |
| `POST /api/entries/<id>/downgrade` | A person's downgrade. Body: `{ "by", "reason" }`. | [`downgrade`](../cli/codelead-pkb-downgrade.md) |
| `GET /api/chunks/<id>` | What a citation opens: the text, the file and lines, or the section, and the commit. | |
| `GET /api/projects/<p>/sources` | The project's sources, each with its kind, origin, hash and version. | |
| `POST /api/projects/<p>/ask` | Ask a question. Body: `{ "question", "by", "dryRun", "confirmRemote" }`. | [`ask`](../cli/codelead-pkb-ask.md) |
| `GET /api/projects/<p>/questions` | The stored questions, each with its answers and marks. | [`asks`](../cli/codelead-pkb-asks.md) |
| `GET /api/questions/<id>` | One question with all its answers. | [`asks <id>`](../cli/codelead-pkb-asks.md) |
| `POST /api/questions/<id>/reask` | Ask a stored question again. Body: `{ "by", "dryRun", "confirmRemote" }`. | [`reask`](../cli/codelead-pkb-reask.md) |
| `POST /api/answers/<id>/promote` | Make an answer an entry. Body: `{ "by" }`. | [`promote`](../cli/codelead-pkb-promote.md) |
| `POST /api/projects/<p>/search` | Search a project. Body: `{ "question", "dryRun", "confirmRemote" }`. | [`search`](../cli/codelead-pkb-search.md) |
| `POST /api/projects/<p>/sync` | Sync a project from the repositories that the service's `pkb.json` names. Body: `{ "by", "dryRun", "embed" }`. | [`sync`](../cli/codelead-pkb-sync.md) |
| `POST /api/projects/<p>/embed` | Make the embeddings that wait. Body: `{ "by", "dryRun", "confirmRemote" }`. | [`embed`](../cli/codelead-pkb-embed.md) |
| `GET /api/projects/<p>/export` | The files of an export, as `files: [{ path, content }]`. Takes `format=md` or `format=json`. | [`export`](../cli/codelead-pkb-export.md) |
| `POST /api/projects/<p>/import` | Rebuild a project from its export. Body: `{ "files": [{ "path", "content" }] }`, the JSON files and the manifest. | [`import`](../cli/codelead-pkb-import.md) |
| `GET /api/stream` | Server-sent events. See [The stream](#the-stream). | |

You can leave out every field of a body that the command does not require. The required ones are `question`, `by` for a promotion, and `by` and `reason` for a trust change. A write that creates something answers `201`, and `200` otherwise.

The consent rule and `paused` hold for a route as for its command. `paused` is read from `pkb.json` at each request, so it holds at once.

## The stream

`GET /api/stream` keeps the connection open and sends events as they happen. A browser reads it with `EventSource`.

| Event | When | What it holds |
|---|---|---|
| `ready` | First, on each connection | The id of the last event, and the time between heartbeats. |
| `sync` | A sync starts, after each reader, and at its end | The project, the stage (`started`, `reader`, `done` or `failed`), the reader and the counts. |
| `entry` | An entry changed: a confirm, a downgrade, a promotion, a new version at a sync | The entry's id, project, summary, trust, version, and who. |
| `answers` | Stored answers were marked | The entry or source, how many answers, and the words of the mark. |
| `answer` | An answer was stored | The project, the question's id, the answer's id, and whether it is "not in the knowledge base". |
| `heartbeat` | Every 15 seconds | The time, and how many clients read the stream. |

Each event but `ready` and `heartbeat` has an id. A client that comes back with the header `Last-Event-ID` gets the events that it missed, from the last 200. At most 50 clients read the stream at once.

## The guards

The service checks each request in this order. A refused request writes nothing.

1. **`Host`**: the service's own address and port. On `127.0.0.1`, that is `127.0.0.1`, `localhost` or `[::1]`. On the network, it is the `bind` address, and each name of the `hosts` key of `pkb.json`.
2. **`OPTIONS`**: refused. No answer of the service ever carries an `Access-Control-Allow-Origin` header. So a browser never lets a page of another site read or write.
3. **`Origin`**: when a request names one, it must be `http://` and an allowed host. A request whose `Sec-Fetch-Site` is not `same-origin` or `none` is refused too. Both hold for reads as for writes.
4. **`Authorization: Bearer <key>`**: needed on each route when the PKB folder's `.env` holds `PKB_SHARED_KEY`. The service compares the key in constant time.
5. **`Content-Type: application/json`** on each write, and a body of one JSON object. A plain HTML form cannot send it.
6. **Limits**: a body of at most 2 MB, or 32 MB for an import; headers in 10 seconds; the whole request in 60 seconds.
7. **Deadlines**: a route answers in 60 seconds, or 600 seconds when it may call a model. At most two requests that may call a model run at once.

> [!NOTE]
> **For team leads:** the guards keep pages of other sites out, even on this machine. They do not tell people apart. Each person who reaches the service, with the key when one is set, can read every project and confirm or downgrade under any name. Accounts and roles come in a later release.

## Examples

These examples use `curl` with the sample project `tally`, through a service on port `7425`.

- See what runs:

  ```console
  curl -s http://127.0.0.1:7425/api/health
  ```

  The answer holds `serving`, `store`, `chat`, `embeddings`, `paused`, `configured_projects`, `projects`, `words` and `exit`.

- Ask a question that the project does not know. No model is called:

  ```console
  curl -s -X POST http://127.0.0.1:7425/api/projects/tally/ask \
    -H "Content-Type: application/json" \
    -d '{"question":"How are employees paid for overtime?","by":"agent-1"}'
  ```

  The status is `201`. `words` begins with these lines:

  ```output
  Ask · tally: "How are employees paid for overtime?"
  Not in the knowledge base.
  Model: none was called, because nothing cleared the floor.
  ```

- Downgrade an entry, with a name and a reason:

  ```console
  curl -s -X POST http://127.0.0.1:7425/api/entries/e-91a3e6deb0d285d7b045/downgrade \
    -H "Content-Type: application/json" \
    -d '{"by":"person-b","reason":"The list answers again since the customer is optional"}'
  ```

- Read the stream for 8 seconds while another terminal confirms and downgrades:

  ```console
  curl -sN --max-time 8 http://127.0.0.1:7425/api/stream
  ```

  ```output
  retry: 2000

  event: ready
  data: {"last":13,"heartbeat_ms":15000}

  id: 14
  event: entry
  data: {"id":"e-ffa9d7b67c42d3734e22","project":"tally","summary":"The web project targets .NET Framework 4.8","trust_level":"confirmed","version":2,"what":"confirmed by person-a","by":"person-a"}

  id: 15
  event: entry
  data: {"id":"e-91a3e6deb0d285d7b045","project":"tally","summary":"Walk finding silent-failure at step bravo-list: GET /api/invoice/list 500 and nothing on the page said so","trust_level":"downgraded","version":2,"what":"downgraded by person-b","by":"person-b"}
  ```

- Send a form from another site. The status is `415`:

  ```console
  curl -s -X POST http://127.0.0.1:7425/api/entries/e-ffa9d7b67c42d3734e22/downgrade \
    -H "Content-Type: text/plain" -d 'by=someone&reason=a+form'
  ```

  ```output
  {"ok":false,"refused":"A write is sent as JSON (Content-Type: application/json): nothing was written. Send the body as JSON with Content-Type: application/json.","error":{"code":"media_type","message":"A write is sent as JSON (Content-Type: application/json): nothing was written.","wayOut":"Send the body as JSON with Content-Type: application/json."},"words":"A write is sent as JSON (Content-Type: application/json): nothing was written.\nSend the body as JSON with Content-Type: application/json.","exit":2}
  ```

- Reach the service by a name that is not its own. The status is `403`:

  ```console
  curl -s http://127.0.0.1:7425/api/health -H "Host: evil.example:7425"
  ```

  ```output
  {"ok":false,"refused":"Refused: this service answers only to its own address. Reach it at http://127.0.0.1:7425.","error":{"code":"host","message":"Refused: this service answers only to its own address.","wayOut":"Reach it at http://127.0.0.1:7425."},"words":"Refused: this service answers only to its own address.\nReach it at http://127.0.0.1:7425.","exit":2}
  ```

## See also

- [`codelead pkb serve`](../cli/codelead-pkb-serve.md): run the service, and the shared key.
- [`codelead pkb`](../cli/codelead-pkb.md): the commands that call these routes.
- [Environment variables](environment-variables.md): `CODELEAD_PKB_URL` and `CODELEAD_PKB_KEY`.
- [Files and folders](files-and-folders.md): `pkb.json` and its `.env` file.
