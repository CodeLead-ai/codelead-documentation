---
title: codelead pkb embed
description: Make the embeddings of a project's pieces in the PKB store, so that a search can find text by its meaning. A text is embedded once.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb embed

**Applies to:** CodeLead 0.1 and later

`codelead pkb embed` makes the embeddings of a project's pieces in the store of the [PKB service](codelead-pkb.md). An embedding is a list of numbers that stands for what a text means. With them, [`codelead pkb search`](codelead-pkb-search.md) finds a piece that says the same thing in other words.

> [!NOTE]
> **New to this?** An embedding model reads a text and gives back a list of numbers, for example 768 of them. Two texts that mean the same thing get lists that point the same way. The search compares the question's list with each piece's list.

## Name

`codelead pkb embed` - Makes the embeddings of a project's pieces that have none.

## Synopsis

```console
codelead pkb embed [--project <ID>] [--dir <FOLDER>] [--dry-run] [--confirm-remote] [--by <NAME>] [--json]
codelead pkb embed -h|--help
```

## Description

`embed` reads every current piece of the project that has no embedding by the model that `pkb.json` names, with its prefixes. It sends their texts to the embeddings server, and stores the numbers with each piece.

- A text that has an embedding already, in another piece, takes that embedding. It is not sent again.
- Each embedding must have the length that `embeddings.dimensions` says. When the length is wrong, `embed` writes nothing and says the fix.
- `embed` also embeds the pieces of `codelead-shared`, the project of CodeLead's own knowledge. Every search of a project reads them too.
- Only one sync or `embed` of a project runs at a time. `embed` waits for no one: it stops and says who holds the project.

### It also runs after each sync

[`codelead pkb sync`](codelead-pkb-sync.md) runs the same step at its end. There it never fails the sync. When no model is named, or the server does not answer, the sync says how many pieces wait. You run `codelead pkb embed` later. `sync --no-embed` skips the step.

### The embeddings server

`pkb.json` names the server in its `embeddings` section:

| Key | What it holds | Default |
|---|---|---|
| `baseUrl` | The server's address. It must answer `POST <baseUrl>/embeddings`, the OpenAI-compatible route. | `http://localhost:1234/v1`, where LM Studio listens |
| `model` | The embedding model to ask for. | empty: nothing is embedded |
| `dimensions` | How many numbers the model gives for each text. | `768` |
| `remote` | `true` when the server is outside your machines. | `false` |
| `documentPrefix` | Text put before each piece's text when it is embedded. | empty |
| `queryPrefix` | Text put before each question when it is embedded. | empty |

A model of 768 numbers fits the default. Examples are `nomic-embed-text` v1.5, `bge-base-en-v1.5` and `all-mpnet-base-v2`. A model with another length needs another `dimensions`. Set it, then run [`codelead pkb store up`](codelead-pkb-store.md) to make the vector index again at that size.

> [!IMPORTANT]
> One store holds one set of embeddings for each piece. When you change `model` or a prefix, every piece is embedded again.

### The task prefixes

Some embedding models learned with a short label before each text. The label says what the text is for. The `nomic-embed-text` family is one of them. It expects `search_document: ` before each stored text and `search_query: ` before each question. Such a model finds better matches when it gets its labels.

Set the pair in the `embeddings` section of `pkb.json`. Keep the space after each colon:

```json
"embeddings": {
  "baseUrl": "http://localhost:1234/v1",
  "model": "text-embedding-nomic-embed-text-v1.5",
  "dimensions": 768,
  "documentPrefix": "search_document: ",
  "queryPrefix": "search_query: "
}
```

Then run `codelead pkb embed`. What happens:

- The store keeps each piece's numbers under a key: the model's name, and the two prefixes when they are set. For example: `text-embedding-nomic-embed-text-v1.5 [prefixes "search_document: " "search_query: "]`.
- A new pair of prefixes is a new key. So every piece waits again, and `embed` embeds it again.
- A search makes the question's numbers with the query prefix. It compares them only with pieces of the same key. Until `embed` runs, the search answers by words and says why.
- With both prefixes empty, the key is the model's name alone. Embeddings made before the prefixes existed keep it.

Other model families use other labels, or none. Read the model's own page before you set them.

### Privacy

The texts of the pieces leave the store only for the embeddings server.

- A server on this machine or on your network is fine. That is the default.
- A server with another address counts as remote, whatever `remote` says.
- A remote server gets the texts only when `privacy.allowRemoteModels` is `true` and you add `--confirm-remote`. Each send is then a line of the disclosure log, in the store and in each repository of the project.
- When `pkb.json` says `paused`, no model is called. A scoring run that owns the model server sets it.

## Options

- **`--project <ID>`**

  The project of `pkb.json`. You can leave it out when `pkb.json` names one project only.

- **`--dir <FOLDER>`**

  The PKB's folder, with `pkb.json`. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--dry-run`**

  Counts what would be sent, and sends and writes nothing. When the store answers and holds the project, it counts the pieces that the store holds. Otherwise it reads the project into a store in memory, as `codelead pkb sync --dry-run` reads it. Its first line says which it counted, and why.

- **`--confirm-remote`**

  Confirms that the texts may go to a remote embeddings server. It has an effect only when `privacy.allowRemoteModels` is `true`.

- **`--by <NAME>`**

  Who embeds. The lock on the project and a remote disclosure name it. The default is `codelead pkb embed`.

- **`--json`**

  Prints the result as one JSON document. The default is text.

## Examples

These examples use the sample project `orchard` of [`codelead pkb search`](codelead-pkb-search.md#examples). Its `pkb.json` sets the two prefixes of the nomic family.

- See what would be sent, with no store set up and without sending anything:

  ```console
  codelead pkb embed --dry-run
  ```

  ```output
  Dry run: the PKB store is not set up; counted in the project read into a store in memory, as codelead pkb sync --dry-run reads it.
  12 chunks wait for an embedding by text-embedding-nomic-embed-text-v1.5 at http://localhost:1234/v1 with the prefixes "search_document: " and "search_query: ": 9 of the project orchard, 3 of the project codelead-shared.
  12 texts (4,440 characters) would be sent in 2 requests; 0 chunks would take the embedding of a known text. Nothing was sent or written.
  ```

  The characters count the prefix of each text: with no prefixes, the same run counts 4,236.

- See what would be sent when the store runs. The dry run then counts the pieces that the store holds:

  ```console
  codelead pkb embed --dry-run
  ```

  <!-- described, not captured -->
  ```output
  Dry run: counted in the PKB store (database codelead_pkb), as it holds the project now.
  …
  ```

- Run it when `pkb.json` names no model. Nothing is sent, and the exit code is `2`:

  ```console
  codelead pkb embed --dry-run
  ```

  ```output
  Dry run: the PKB store is not set up; counted in the project read into a store in memory, as codelead pkb sync --dry-run reads it.
  12 chunks wait for their embeddings: pkb.json names no embeddings model (embeddings.model).
  Name one in pkb.json (an embedding model the embeddings server has loaded, of embeddings.dimensions numbers), then run codelead pkb embed.
  Nothing was sent.
  ```

- Run it while a scoring run owns the model server (`"paused": true`):

  ```console
  codelead pkb embed --dry-run
  ```

  ```output
  Dry run: the PKB store is not set up; counted in the project read into a store in memory, as codelead pkb sync --dry-run reads it.
  12 chunks wait for their embeddings: pkb.json says paused: no model is called while a scoring run owns the model server.
  When the run ends, set "paused": false in pkb.json, then run codelead pkb embed.
  Nothing was sent.
  ```

- Embed the project's pieces. The store must run, and the embeddings server must have the model loaded:

  ```console
  codelead pkb embed
  ```

  <!-- described, not captured -->
  ```output
  9 chunks of the project orchard embedded by text-embedding-nomic-embed-text-v1.5 at http://localhost:1234/v1 with the prefixes "search_document: " and "search_query: " (768 numbers each); none waits.
  3 chunks of the project codelead-shared embedded by text-embedding-nomic-embed-text-v1.5 at http://localhost:1234/v1 with the prefixes "search_document: " and "search_query: " (768 numbers each); none waits.
  Sent: 12 texts in 2 requests.
  ```

- Run it again. Every piece has its embedding, so nothing is sent:

  ```console
  codelead pkb embed
  ```

  <!-- described, not captured -->
  ```output
  Every chunk of the project orchard and of codelead-shared has its embedding by text-embedding-nomic-embed-text-v1.5 with the prefixes "search_document: " and "search_query: " (12 chunks).
  ```

- Run it when the store is not set up:

  ```console
  codelead pkb embed
  ```

  ```output
  The PKB store is not set up: the PKB's folder holds no password for it (.env). codelead pkb store up sets it up.
  Nothing was sent.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Done, nothing to do, or a dry run. |
| `1` | It stopped part-way: the server stopped answering, or the model gives embeddings of another length. What was embedded is kept. |
| `2` | Refused before anything was sent: no `pkb.json`, no such project, no model named, `paused`, a remote server without consent, the store does not answer, or a sync of the project runs. |

## Files

| File | What `embed` does with it |
|---|---|
| `<home>/.codelead/pkb/pkb.json` | Reads the project, and the `embeddings` (with its two prefixes), `privacy` and `paused` settings. |
| `<home>/.codelead/pkb/.env` | Reads the store's password. |
| `.codelead/disclosures.jsonl`, in each repository of the project | Adds one line for each send to a remote server. A server on your machines adds none. |

## See also

- [`codelead pkb`](codelead-pkb.md): the commands of the PKB service.
- [`codelead pkb search`](codelead-pkb-search.md): search by words, by meaning and in the code.
- [`codelead pkb sync`](codelead-pkb-sync.md): send a project's knowledge into the store. It runs this step at its end.
- [`codelead pkb store`](codelead-pkb-store.md): start the store, and make its vector index again after a change of `dimensions`.
- [Privacy and remote models](../concepts/privacy-and-remote-models.md)
