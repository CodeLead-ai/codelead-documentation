---
title: codelead implement
description: Let CodeLead work the board's cards without a person, one session for each card, with caps on cards, time and the clock; the cards in Ready each on their own branch and worktree.
kind: reference
area: cli
date: 2026-11-25
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
codelead implement [<TARGET>] --ready [--max-cards <N>] [--max-minutes <N>] [--until <TIME>]
                   [--repair-rounds <N>] [--dry-run] [--json]
codelead implement -h|--help
```

## Description

The [board](../pinboard/index.md) holds cards: each card is a piece of work with a title, a description and a thread. A person gives a card to CodeLead in one of two ways: with the tag `for:codelead`, or by moving the card to the column **Ready**. `codelead implement` then takes the card and works it like this:

1. It starts a session and types [`/implement --card <id>`](../chat-commands/implement.md). The card's text is the request.
2. When the [validation](../concepts/validation-and-the-gate.md) fails, it runs up to `--repair-rounds` rounds of [`/repair`](../chat-commands/repair.md), [`/edit`](../chat-commands/edit.md) and [`/validate`](../chat-commands/validate.md).
3. It types `/exit`. The session writes the card's trail as the participant `codelead`: "Started", then "Landed" with the evidence record, or "Stopped" with the reason.

A card that landed moves to **For review**, where a person reviews it.

> [!NOTE]
> **New to this?** *Unattended* means that nobody answers the questions. CodeLead answers them itself, and it stops a card when it cannot continue. A person reviews the result on the board.

### One card, the queue, or Ready

- `--card <id>` works the one card, wherever it stands, then ends. A card in **Ready** is the exception: the board gives such a card, so `--card` refuses it and names the two ways out.
- `--queue` takes the cards tagged `for:codelead`: the top of **Backlog** first, then the other columns in the board's order. It never takes a card from **Ready**, **In progress**, **For review** or **Done**. CodeLead reads the board again between cards, so a card whose tag a person removed is not taken.
- `--ready` asks the board for CodeLead's card from **Ready**, works it, hands it back, and asks again, until the board has nothing for it. See the next section.

### The cards in Ready

A card in Ready says three things. It says how to tell it is done (`definitionOfDone`) and what it may touch (`mayTouch`). It names its limit (`limitMinutes`, `limitMoney`, or both). Only the person moves a card there. See [Hand a card to CodeLead through Ready](../pinboard/work-with-agents.md#hand-a-card-to-codelead-through-ready).

`--ready` works each card like this:

1. It asks the board for its card, as every agent does ([`codelead board take`](../pinboard/cli/board-take.md)). CodeLead does not choose. The board moves the card to In progress and holds it for CodeLead with a claim.
2. It makes the branch `card/<id>-<title>` from your HEAD, and a git worktree for it beside your checkout, in the folder `<repository>-cards/<id>-<title>`. Nothing lands on the branch you have checked out, and your own changes are never touched.
3. It runs the card's session in that worktree. The session's checkpoint commits go to the card's branch.
4. The card's three things steer the session:
   - The "done when" words go into the request, marked as the card's own.
   - A patch that touches a file outside `mayTouch` is refused at the patch check, in words that name the file and the card's `mayTouch`. It counts as a failed attempt, like any refusal.
   - When `limitMinutes` is up, no model call starts, and the session ends at its next step. `limitMoney` stops the priced calls in the same way. With both, the first one reached stops it.
5. It renews the claim while the session works. When the person moves the card away, the next renewal is refused, and the session ends: the person's move wins.
6. It hands the card back to the person in **For review**, with one comment, landed or stopped. It never puts the card back in Ready, so a card is not taken again in a loop. Nothing is merged.

The worktree is removed when it holds nothing that is not committed, and the branch is kept. When it holds work that is not committed, it stays, and the card's comment names it.

Nobody answers in the session. It follows the rule of every unattended run: it takes the recommended answer, and it stops at a step that is hard to undo.

> [!NOTE]
> **New to this?** `mayTouch` should be paths: `src/auth/`, `test/auth.test.ts`. A repository's name or a sentence bounds nothing, because the board reads it as the whole repository. The card's `Started:` comment says so.

### Caps and stops

A queue run ends when no tagged card is left, or at the first cap: `--max-cards`, `--max-minutes`, `--until`. It also ends when the stop file `.codelead/STOP` appears, or on Ctrl+C. The card that is running always finishes first.

A `--ready` run ends when the board has nothing for CodeLead, with the board's own words, or at the same caps. The stop file and Ctrl+C end the card that is running at its session's next step. The card then goes back to the person with that reason.

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

  Works this card (`42` or `#42`), wherever it stands but in Ready, then ends.

- **`--queue`**

  Takes the cards tagged `for:codelead`, top of Backlog first, then the other columns in the board's order. Never from Ready, In progress, For review or Done. The old column "Queued for CodeLead" still counts, after the tagged cards.

- **`--ready`**

  Asks the board for CodeLead's card from Ready, works it on its own branch in its own worktree, and hands it back in For review. Then it asks again, until the board has nothing for CodeLead. A board with no Ready column is refused: [`codelead board add-ready`](../pinboard/cli/board-add-ready.md) gives it one.

- **`--repair-rounds <N>`**

  The repair rounds that a card's session may take (`/repair`, `/edit`, `/validate`). A whole number, 0 or more. The default is 2.

- **`--max-cards <N>`**

  With `--queue` or `--ready`: the most cards that the run works.

- **`--max-minutes <N>`**

  With `--queue` or `--ready`: no card starts once the run has gone on this many minutes.

- **`--until <TIME>`**

  With `--queue` or `--ready`: no card starts at or after this time. Give `HH:MM` (the next such time of day) or an ISO time such as `2026-11-07T06:00:00Z`.

- **`--dry-run`**

  Checks, then lists what would run: the models, the queue in order, the caps and the lock. With `--ready`: what the board would give CodeLead now, or its words for why not, and each card in Ready. Nothing is run or written: the dry run reads the board, and takes no card.

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
  No card is tagged for:codelead outside Ready, In progress, For review and Done: a run would end at once.
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
  Stopped: no card is tagged for:codelead outside Ready, In progress, For review and Done.
  ```

- Work the cards in Ready. Sam put one there, with its three things. This output comes from CodeLead's tests; the times, ids and hashes are examples:

  ```console
  codelead implement --ready
  ```

  ```output
  12:11:44Z The Ready cards of docs/codelead/board, as the board gives them to codelead: no caps; up to 2 repair rounds per card; the stop file .codelead/STOP and Ctrl+C end the card in flight at its session's next step, and the run.
  12:11:44Z #1 "Show the count of free berths" (Ready): the board gave it to codelead, held until 2026-11-25T12:41:44Z. Done when: The bookings page shows the count of free berths; npm test passes. May touch: src/. Limit: 30 minutes.
  12:11:44Z #1 "Show the count of free berths" (Ready): started, driven with /implement --card 1 in its worktree ../shop-cards/1-show-the-count-of-free-berths.
  12:11:45Z #1 "Show the count of free berths": landed, evidence record ev-20261125121145-0lg3iy, 0 min; now in For review. Branch card/1-show-the-count-of-free-berths: 1 commit; worktree removed (clean).
  12:11:45Z The run stops: Nothing for codelead: Ready is empty. The person puts a card there when an agent may pick it up by itself.
  Implement run (the Ready cards): 1 card worked, 1 landed, 0 min. Stopped: Nothing for codelead: Ready is empty. The person puts a card there when an agent may pick it up by itself.
  ```

  The card is in For review, with one comment that starts `Landed: 8f90a6e on card/1-show-the-count-of-free-berths`. The comment lists the branch's commits, what the validation ran against the card's "done when", and the worktree's state. See [Hand a card to CodeLead through Ready](../pinboard/work-with-agents.md#hand-a-card-to-codelead-through-ready).

- A card whose time limit was reached. The session ended at its next step, before a repair round. From CodeLead's tests:

  ```output
  Stopped: the session stopped at the card's limit of 5 minutes (it had run 6 minutes): no model call started after the time was up, and the session was ended at its next step.
  ```

  The card is in For review, and the comment says how to go on. Raise `limitMinutes` and move the card to Ready again, or carry on by hand in its worktree.

- Name a card that stands in Ready:

  ```console
  codelead implement --card 1
  ```

  ```output
  The run is refused; nothing was run:
  - Card #1 is in Ready: the board gives a card in Ready to the agent it is for, and an agent does not take one by its name, so --card does not work it. Two ways out: `codelead implement --ready` asks the board for its card (this one, when it is the next for codelead); or the person moves the card out of Ready (then `codelead implement --card 1` works it), or works it in a session (/implement --card 1).
  ```

- Run `--ready` on a board made before Ready. CodeLead refuses with the board's words:

  ```console
  codelead implement --ready
  ```

  ```output
  The run is refused; nothing was run:
  - This board has no Ready column, so no card is given: Ready is where the person puts a card an agent may pick up by itself. `codelead board add-ready` gives the board its Ready column.
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
  - Nothing to work: name a card (--card <id>), or take the queue (--queue: the cards tagged for:codelead, top of Backlog first), or ask the board for the cards in Ready (--ready).
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Every card worked landed. An empty queue is also `0`, and so is a board with nothing for CodeLead. |
| `1` | A card did not land. |
| `2` | Nothing could run: no board, no card named, an unknown card, a card in Ready named with `--card`, a lock held by another run, a remote model that is not allowed. With `--ready`, also a board with no Ready column, or a target that is not a git repository with a commit. With `--dry-run`, also a model that does not answer. |
| `130` | Ctrl+C. The card that was running finished first. |

## Files

| File | What `implement` does with it |
|---|---|
| `docs/codelead/board/` | Reads the cards. The sessions add comments and events, and move landed cards to For review. |
| `.codelead/STOP` | Ends the run after the card that is running when this file appears. |
| The run lock under `.codelead/` | Held for the run and given back at its end. |
| `.codelead/evidence/ev-*.json` | Each card's session writes its evidence records. |
| `.codeleadsessions/<session id>/` | Each card's session writes its log. A Ready card's session writes it here too, though it runs in the card's worktree. |
| The branch `card/<id>-<title>` | With `--ready`: the card's branch, made from your HEAD. It is kept. |
| `../<repository>-cards/<id>-<title>/` | With `--ready`: the card's worktree. It is removed when it holds nothing that is not committed. |

## See also

- [Work with agents](../pinboard/work-with-agents.md)
- [Tutorial: work with agents on the board](../tutorials/work-with-agents-on-the-board.md)
- [`/implement`](../chat-commands/implement.md): the command that each card's session runs.
- [`codelead board`](codelead-board.md): make the board and tag the cards.
- [`codelead board take`](../pinboard/cli/board-take.md): how an agent gets its card from Ready.
- [`codelead pr`](codelead-pr.md): open a pull request for a landed card.
- [Stop and resume work](../how-to/stop-and-resume.md)
- [Results and stops](../concepts/results-and-stops.md)
