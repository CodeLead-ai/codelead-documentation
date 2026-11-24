---
title: codelead pkb store
description: Start, stop and check the store of the PKB service on this machine, MongoDB Community with its search process, from one compose file.
kind: reference
area: cli
date: 2026-11-24
---

# codelead pkb store

**Applies to:** CodeLead 0.1 and later

`codelead pkb store` runs the store of the [PKB service](codelead-pkb.md) on this machine. The store is MongoDB Community with its search process, mongot, in two Docker containers. `up` starts it, `down` stops it, and `status` says what runs.

> [!IMPORTANT]
> [`codelead pkb sync`](codelead-pkb-sync.md) writes a project's knowledge into the store. Nothing reads it back yet: the service and `ask` come later.

## Name

`codelead pkb store` - Starts (`up`), stops (`down`) and checks (`status`) the store of the PKB service.

## Synopsis

```console
codelead pkb store up [--dir <FOLDER>] [--json]
codelead pkb store down [--dir <FOLDER>] [--json]
codelead pkb store status [--dir <FOLDER>] [--json]
codelead pkb store -h|--help
```

## Description

| Subcommand | What it does |
|---|---|
| `up` | Writes the store's files, starts the two containers, sets up the database and its users, and makes the collections and indexes. |
| `down` | Stops the two containers. The data stays. |
| `status` | Says whether the containers run, how many documents each collection holds, and whether the search indexes answer. |

All three need Docker: Docker Desktop or Colima, with the compose plugin. Each one checks Docker first. When Docker is missing or does not answer, the command stops before it writes anything.

> [!NOTE]
> **New to this?** Docker runs programs in containers, which are small Linux systems on your computer. A compose file lists the containers that work together. A volume is a folder that Docker keeps, so the data stays when a container stops.

### What up does

1. **It reads `pkb.json`** in the store's folder. On the first run the file does not exist, and `up` uses the defaults.
2. **It checks the store's address.** The store listens on `127.0.0.1` only. `up` refuses a `store.uri` with another host.
3. **It checks Docker**, and that no other program holds the store's port.
4. **It writes the files.** It writes `pkb.json` once and the passwords once. It writes the compose file and the two configuration files on every run.
5. **It starts mongod**, the database process, and waits until it answers.
6. **It sets up mongod**: a replica set of one member, and three users. A replica set is how MongoDB keeps copies of its data. mongot reads the changes from it.
7. **It starts mongot**, the search process.
8. **It makes the collections and the indexes.** mongot builds its two search indexes a moment later.

You can run `up` again at any time. It keeps `pkb.json` and the passwords, and it changes nothing that is set up already.

### When a container cannot start

When Docker is quit while the store runs, it can leave a stopped container behind that never starts again. Docker then says, for example, `RWLayer of container … is unexpectedly nil`. A container holds none of the store's data: the data is in the two volumes.

So when mongod or mongot does not start, `up` looks at the store's own containers:

1. It looks for a container of that service, in the store's compose project, that is stopped (`created`, `exited` or `dead`).
2. It removes that one container by its id, with `docker rm`. It never removes a volume, a running container, or a container of another project.
3. It starts the service once more, and says what it removed:

   <!-- described, not captured -->
   ```output
   mongod could not start from its stopped container (Docker said: Error response from daemon: RWLayer of container 3f2a9c1b7d4e… is unexpectedly nil). That container (3f2a9c1b7d4e) was removed and a new one started; the store's data is in its volumes, which were not touched.
   ```

When the service still does not start, or no stopped container is there, `up` stops with Docker's words and a second line with the way out:

<!-- described, not captured -->
```output
docker compose could not start mongod: Error response from daemon: Mounts denied
Its stopped container (3f2a9c1b7d4e) was removed first (Docker had said: Error response from daemon: RWLayer of container 3f2a9c1b7d4e… is unexpectedly nil), and a new one does not start either. If Docker was quit or restarted while the store ran, quit Docker Desktop fully, start it again, then run codelead pkb store up again. The store's data is in the volumes codelead-pkb_mongod-data and codelead-pkb_mongot-data, never in a container.
```

### Security

- The store's port is published on `127.0.0.1` only. Another computer cannot reach it.
- mongot's ports are not published at all. Only mongod reaches mongot, inside Docker.
- `up` makes the passwords and the keyfile on this machine, on its first run. They are in `.env` only, and only your user can read that file.
- `pkb.json` holds the store's address without the password.

### The users

| User | Role | What for |
|---|---|---|
| `pkb-admin` | `root` | Sets up the store. Only `up` uses it. |
| `pkb-service` | `readWrite` and `dbAdmin` on the store's database | The PKB service's own user. |
| `pkb-mongot` | `searchCoordinator` | mongot reads the changes with it. |

### The collections

The store's database is `codelead_pkb` by default. It holds nine collections:

| Collection | What it holds |
|---|---|
| `projects` | The projects, each with its repositories. A project is private by default. |
| `sources` | Where knowledge came from: a file, a report, a card. Each source has a hash, a version and a default trust level. |
| `entries` | The knowledge entries, as in the encyclopedia: type, summary, body, trust level, history and version. |
| `chunks` | Short pieces of text that a search finds. Each one names what it belongs to, and has that owner's trust level. A piece whose text changed is superseded: it stays, and a search never finds it. |
| `evidence` | CodeLead's evidence records, as they were written. |
| `questions` | The questions asked, each with who asked it and when. |
| `answers` | Each answer to a question. An answer is never changed. It is only marked when its evidence changed. |
| `disclosures` | A log of what was sent and where to, as ids, file hashes and counts only. |
| `locks` | The project that a sync holds, so that only one sync of a project runs at a time. |

### The indexes

- **Plain indexes** keep each entry and each question once per project. They also find a project's sources, chunks and answers quickly.
- **`chunks_text`** is a search index on the words of each chunk.
- **`chunks_embedding`** is a vector search index on the embedding of each chunk, with cosine similarity.
- You can filter both search indexes by project, by what a chunk belongs to, and by trust level. Both leave out a superseded chunk.

An embedding is a list of numbers that a model makes from a text. Texts with a similar meaning get similar numbers. The number of values comes from `embeddings.dimensions` in `pkb.json`, 768 by default. When you change it, the next `up` changes the vector index.

## Options

These options belong to each of `up`, `down` and `status`.

- **`--dir <FOLDER>`**

  The store's folder, with `pkb.json`, the env file and the compose file. The default is `<home>/.codelead/pkb`. The environment variable `CODELEAD_PKB_HOME` changes the default.

- **`--json`**

  Prints the result as one JSON document. The default is text.

## Examples

- Start the store. The first run downloads the two images, about 750 MB:

  ```console
  codelead pkb store up
  ```

  <!-- described, not captured -->
  ```output
  The PKB store is up on 127.0.0.1:27415: MongoDB 8.3.17 with its search process mongot 1.70.6 (compose project codelead-pkb).
  Files in <home>/.codelead/pkb: pkb.json written, .env written, compose.yaml written, mongod.yml written, mongot.yml written. The passwords and the keyfile were made now, in .env (readable by you only).
  Replica set pkb: initiated now. Users: pkb-admin (root), pkb-service (the PKB's own: readWrite and dbAdmin on codelead_pkb), pkb-mongot (searchCoordinator); made now: pkb-admin, pkb-service, pkb-mongot.
  Collections: 9 in codelead_pkb (made now: 9); plain indexes: 10 (made now: 10).
  Indexes: 10 plain; search: chunks_text (search) PENDING; chunks_embedding (vectorSearch, 768 dimensions) PENDING.
  Connect: mongodb://pkb-service:<password>@127.0.0.1:27415/?authSource=admin&directConnection=true, database codelead_pkb (the password is PKB_SERVICE_PASSWORD in <home>/.codelead/pkb/.env).
  ```

  `PENDING` means that mongot is still building the search index. `status` shows `READY` a moment later.

- Run it when Docker does not run. Nothing is started or written:

  ```console
  codelead pkb store up
  ```

  ```output
  Docker does not answer (Cannot connect to the Docker daemon at unix://<home>/.docker/run/docker.sock. Is the docker daemon running?). Start Docker Desktop (or run colima start), then run this again.
  Nothing was started or written.
  ```

  Start Docker Desktop, or run `colima start`, and run `up` again. The exit code is `2`.

- Check what runs:

  ```console
  codelead pkb store status
  ```

  <!-- described, not captured -->
  ```output
  The PKB store in <home>/.codelead/pkb (compose project codelead-pkb, 127.0.0.1:27415): mongod running (healthy); mongot running.
  Collections in codelead_pkb: projects 0, sources 0, entries 0, chunks 0, evidence 0, questions 0, answers 0, disclosures 0, locks 0.
  Indexes: 10 plain; search: chunks_text (search) READY, queryable; chunks_embedding (vectorSearch, 768 dimensions) READY, queryable.
  ```

- Stop the store and keep its data:

  ```console
  codelead pkb store down
  ```

  <!-- described, not captured -->
  ```output
  The PKB store is down: its containers and its network are removed. Its data stays in the volumes codelead-pkb_mongod-data and codelead-pkb_mongot-data; codelead pkb store up starts it again with that data.
  ```

- Run a second store for a trial, beside the first one. Give it its own folder, port and project name. First write `pkb-trial/pkb.json`:

  ```json
  {
    "store": {
      "uri": "mongodb://pkb-service@127.0.0.1:27499/?authSource=admin&directConnection=true",
      "database": "codelead_pkb",
      "compose_project": "codelead-pkb-trial"
    }
  }
  ```

  Then start it, and stop it when the trial ends:

  ```console
  codelead pkb store up --dir pkb-trial
  codelead pkb store down --dir pkb-trial
  ```

> [!CAUTION]
> `down` keeps the data. To delete the data of a store, remove its two volumes after `down`, for example `docker volume rm codelead-pkb-trial_mongod-data codelead-pkb-trial_mongot-data`. You cannot undo it.

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Done. For `status`: the store answers, and both search indexes answer. |
| `1` | The command ran and failed, and the words say why and the way out. For `status`: no store is set up in the folder, mongod does not run, or the search indexes do not answer yet. |
| `2` | Refused before anything ran: Docker is missing or does not answer, `store.uri` is not on `127.0.0.1`, the port is taken, `pkb.json` is not JSON, or an option is wrong. |

## Files

All files are in the store's folder, `<home>/.codelead/pkb/` by default:

| File | Written | What it holds |
|---|---|---|
| `pkb.json` | by the first `up`, then it is yours | The configuration of the PKB service. See [The configuration](#the-configuration). |
| `.env` | by every `up`, with the passwords of the first | The store's port and database, the user names, the passwords and the keyfile. Only your user can read it. Never copy it into a repository. |
| `compose.yaml` | by every `up` | The two containers, their volumes and the published port. |
| `mongod.yml` | by every `up` | The configuration of mongod. |
| `mongot.yml` | by every `up` | The configuration of mongot. |

Docker keeps the data in two volumes, `<project>_mongod-data` and `<project>_mongot-data`. Here `<project>` is `store.compose_project`.

### The configuration

`pkb.json` holds the whole configuration of the PKB service. This release reads `store`, `embeddings`, `projects`, `privacy` and `paused`. The other keys are for the service, which comes later. A key that the file leaves out takes its default.

| Key | Default | What it does |
|---|---|---|
| `store.uri` | `mongodb://pkb-service@127.0.0.1:27415/?authSource=admin&directConnection=true` | The store's address and port, without the password. The host must be `127.0.0.1` or `localhost`. |
| `store.database` | `codelead_pkb` | The database of the nine collections. |
| `store.compose_project` | `codelead-pkb` | The name that Docker gives the containers and the volumes. |
| `embeddings.dimensions` | `768` | The number of values in every embedding, and in the vector index. |
| `embeddings.baseUrl`, `embeddings.model`, `embeddings.remote` | `http://localhost:1234/v1`, no model, `false` | The embeddings server of [`codelead pkb embed`](codelead-pkb-embed.md#the-embeddings-server) and [`codelead pkb search`](codelead-pkb-search.md). |
| `embeddings.documentPrefix`, `embeddings.queryPrefix` | empty, empty | The text put before each stored text and each question when they are embedded. See [the task prefixes](codelead-pkb-embed.md#the-task-prefixes). |
| `bind`, `port` | `127.0.0.1`, `7415` | The service's own address. Not used yet. |
| `chat` | a local model server, no model | The chat model of the service. Not used yet. |
| `projects` | none | The projects, each with its repositories and their checkouts. [`codelead pkb sync`](codelead-pkb-sync.md#the-project-in-pkbjson) reads them. |
| `privacy.allowRemoteModels`, `paused` | `false`, `false` | Whether texts may go to a remote model, and whether every model call waits. `embed` and `search` follow them. |

## Details

- The images are pinned: `mongodb/mongodb-community-server:8.3.17-ubi9` and `mongodb/mongodb-community-search:1.70.6`.
- Each container writes its secret into a folder in memory when it starts. No secret is on the container's disk or on a command line.
- `up` sets up the users with `mongosh` inside the mongod container. It makes the first user through the localhost exception of MongoDB.
- `up` waits up to 90 seconds for mongot before it makes the search indexes.
- mongot follows mongod a moment later. A search can miss a document for a short time after it is written.

## See also

- [`codelead pkb`](codelead-pkb.md): the commands of the PKB service, and what does not exist yet.
- [`codelead pkb sync`](codelead-pkb-sync.md): send a project's knowledge into the store.
- [The knowledge base](../concepts/knowledge-base.md)
- [`codelead doctor`](codelead-doctor.md): checks that Docker is installed and answers.
- [Environment variables](../reference/environment-variables.md#folders)
- [Files and folders](../reference/files-and-folders.md#your-home-folder)
