---
title: codelead implement
description: Let CodeLead work the board's cards without a person, one session for each card, with caps on cards, time and the clock.
kind: reference
area: cli
date: 2026-11-20
---

# codelead implement

**Applies to:** CodeLead 0.1 and later

`codelead implement` works the cards of the repository's board without a person. Each card is one session: CodeLead implements the card's request, repairs it when a check fails, and writes the result on the card.

## Name

`codelead implement` - Works the board's cards unattended.

## Synopsis

```console
codelead implement [<TARGET>] --card <ID> [--repair-rounds <N>] [--dry-run] [--json]
codelead implement [<TARGET>] --queue [--max-cards <N>] [--max-minutes <N>] [--until <TIME>]
                   [--repair-rounds <N>] [--dry-run] [--json]
codelead implement -h|--help
```

## Description

The [board](../pinboard/index.md) holds cards: each card is a piece of work with a title, a description and a thread. A person gives a card to CodeLead with the tag `for:codelead`. `codelead implement` then takes the card and works it like this:

1. It starts a session and types [`/implement --card <id>`](../chat-commands/implement.md). The card's text is the request.
2. When the [validation](../concepts/validation-and-the-gate.md) fails, it runs up to `--repair-rounds` rounds of [`/repair`](../chat-commands/repair.md), [`/edit`](../chat-commands/edit.md) and [`/validate`](../chat-commands/validate.md).
3. It types `/exit`. The session writes the card's trail as the participant `codelead`: "Started", then "Landed" with the evidence record, or "Stopped" with the reason.

A card that landed moves to **For review**, where a person reviews it.

> [!NOTE]
> **New to this?** *Unattended* means that nobody answers the questions. CodeLead answers them itself, and it stops a card when it cannot continue. A person reviews the result on the board.

### One card or the queue

- `--card <id>` works the one card, wherever it stands, then ends.
- `--queue` takes the cards tagged `for:codelead`: the top of **Backlog** first, then the other columns in the board's order. It never takes a card from **In progress**, **For review** or **Done**. CodeLead reads the board again between cards, so a card whose tag a person removed is not taken.

### Caps and stops

A queue run ends when no tagged card is left, or at the first cap: `--max-cards`, `--max-minutes`, `--until`. It also ends when the stop file `.codelead/STOP` appears, or on Ctrl+C. The card that is running always finishes first.

One run at a time works in a clone: the run takes a lock, and a second run is refused while the lock is held.

### Remote models

Nobody can answer a consent question in an unattended run. A run whose model is remote is refused unless `privacy.allowRemoteModels` is `true` or `CODELEAD_ALLOW_REMOTE_MODELS=1` is set. See [Privacy and remote models](../concepts/privacy-and-remote-models.md).

> [!TIP]
> **For experienced users:** on macOS, the run keeps the machine awake with `caffeinate -i -s` while it runs. An idle Mac can sleep after a minute, and a run on a sleeping Mac becomes very slow.

Every card that lands writes an [evidence record](../concepts/evidence-records.md), and its id is on the card. A team lead reviews the work from the board. See [Work with agents](../pinboard/work-with-agents.md).

## Arguments

- **`<TARGET>`**

  The repository whose board the run works. The default is the current folder.

## Options

- **`--card <ID>`**

  Works this card (`42` or `#42`), wherever it stands, then ends.

- **`--queue`**

  Takes the cards tagged `for:codelead`, top of Backlog first, then the other columns in the board's order. Never from In progress, For review or Done. The old column "Queued for CodeLead" still counts, after the tagged cards.

- **`--repair-rounds <N>`**

  The repair rounds that a card's session may take (`/repair`, `/edit`, `/validate`). A whole number, 0 or more. The default is 2.

- **`--max-cards <N>`**

  With `--queue`: the most cards that the run works.

- **`--max-minutes <N>`**

  With `--queue`: no card starts once the run has gone on this many minutes.

- **`--until <TIME>`**

  With `--queue`: no card starts at or after this time. Give `HH:MM` (the next such time of day) or an ISO time such as `2026-11-07T06:00:00Z`.

- **`--dry-run`**

  Checks, then lists what would run: the models, the queue in order, the caps and the lock. Nothing is run or written.

  The checks include the model server. CodeLead asks it for its list of models, with no generation call. Each model that a card's session can call must answer. When one does not answer, the dry run is refused with exit code 2.

- **`--json`**

  Prints the summary, or the dry run, as one JSON object.

## Examples

- See what a queue run would do, without running it. The model server is running, and the roles name no model, so the server's loaded model counts:

  ```console
  codelead implement --queue --dry-run
  ```

  <!-- described, not captured -->
  ```output
  Dry run of codelead implement --queue: the checks passed; nothing is run or written.
  Board: docs/codelead/board.
  Model: the default role, the preflight role, the fast strategy, the repair role, the clarify strategy, the architect strategy, the deep strategy: lmstudio (the server's loaded model).
  No card is tagged for:codelead outside In progress, For review and Done: a run would end at once.
  Each card: one session driven with /implement --card <id>, then up to 2 repair rounds, then /exit; the session writes the card's trail as codelead.
  Caps: no caps; up to 2 repair rounds per card; the stop file .codelead/STOP and Ctrl+C stop the run after the card in flight.
  Lock: free: a run started now takes it.
  ```

- A dry run with two tagged cards and two caps. This output comes from CodeLead's tests, and the card numbers are examples:

  ```console
  codelead implement --queue --dry-run --max-cards 3 --until 2026-11-07T06:00:00Z
  ```

  ```output
  Dry run of codelead implement --queue: the checks passed; nothing is run or written.
  …
  The queue (the cards tagged for:codelead, top of Backlog first), in order: 2 cards.
    1. #2 "Invoices carry a total" (Backlog)
    2. #1 "Export an invoice as PDF" (Backlog)
  …
  Caps: at most 3 cards, no card starts at or after 2026-11-07T06:00:00.000Z; up to 2 repair rounds per card; …
  Lock: free: a run started now takes it.
  ```

- Run the dry run when no model server answers. Here the address names a port where nothing runs. CodeLead refuses the dry run with exit code 2, and names the model, the address and the way out:

  ```console
  CODELEAD_BASE_URL=http://127.0.0.1:9/v1 codelead implement --queue --dry-run
  ```

  ```output
  The run is refused; nothing was run:
  - The model of the default role, the preflight role, the fast strategy, the repair role, the clarify strategy, the architect strategy, the deep strategy does not answer: lmstudio (http://127.0.0.1:9/v1) answered no list of its models (Could not connect to lmstudio at http://127.0.0.1:9/v1. If this is LM Studio, make sure it is open and the Local Server is running.). Start the model server (or name a model that answers in .codelead/config.json), then run the dry run again.
  ```

  Start the model server, then run the dry run again. `--card <ID> --dry-run` checks the models in the same way.

- Work the queue overnight, and start no card after 06:00:

  ```console
  codelead implement --queue --until 06:00
  ```

  <!-- described, not captured -->
  ```output
  #2 "Invoices carry a total" (Backlog): started, driven with /implement --card 2.
  #2 "Invoices carry a total": landed, evidence record ev-…
  #1 "Export an invoice as PDF" (Backlog): started, driven with /implement --card 1.
  #1 "Export an invoice as PDF": landed, evidence record ev-…
  Implement run (the queue): 2 cards worked, 2 landed …
  Stopped: no card is tagged for:codelead outside In progress, For review and Done.
  ```

- Run it in a repository that has no board. CodeLead refuses and says how to make one:

  ```console
  codelead implement --queue --dry-run
  ```

  ```output
  The run is refused; nothing was run:
  - No board in <your project>/docs/codelead/board: the run takes its cards from the repository's board. `codelead board init` makes one; then tag a card for:codelead (or name one with --card).
  ```

- Run it with neither `--card` nor `--queue`:

  ```console
  codelead implement
  ```

  ```output
  The run is refused; nothing was run:
  - Nothing to work: name a card (--card <id>), or take the queue (--queue: the cards tagged for:codelead, top of Backlog first).
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Every card worked landed. An empty queue is also `0`. |
| `1` | A card did not land. |
| `2` | Nothing could run: no board, no card named, an unknown card, a lock held by another run, a remote model that is not allowed. With `--dry-run`, also a model that does not answer. |
| `130` | Ctrl+C. The card that was running finished first. |

## Files

| File | What `implement` does with it |
|---|---|
| `docs/codelead/board/` | Reads the cards. The sessions add comments and events, and move landed cards to For review. |
| `.codelead/STOP` | Ends the run after the card that is running when this file appears. |
| The run lock under `.codelead/` | Held for the run and given back at its end. |
| `.codelead/evidence/ev-*.json` | Each card's session writes its evidence records. |
| `.codeleadsessions/<session id>/` | Each card's session writes its log. |

## See also

- [Work with agents](../pinboard/work-with-agents.md)
- [Tutorial: work with agents on the board](../tutorials/work-with-agents-on-the-board.md)
- [`/implement`](../chat-commands/implement.md): the command that each card's session runs.
- [`codelead board`](codelead-board.md): make the board and tag the cards.
- [`codelead pr`](codelead-pr.md): open a pull request for a landed card.
- [Stop and resume work](../how-to/stop-and-resume.md)
- [Results and stops](../concepts/results-and-stops.md)
