---
title: codelead board tidy
description: After an import, remove from each imported card's description the opening repetition of its title in bold, with a dry run that shows every change first.
kind: reference
area: pinboard
date: 2026-11-20
---

# codelead board tidy

**Applies to:** CodeLead 0.1 and later

`codelead board tidy --imported` cleans the cards that [`codelead board import`](board-import.md) made. The import copies an item's bold opening into both the title and the description. `tidy` removes that repetition from the description.

## Name

`codelead board tidy` - Removes the repeated bold title from imported cards.

## Synopsis

```console
codelead board tidy --imported [--as <PARTICIPANT>] [--dry-run] [--board <DIR>] [--json]
codelead board tidy -h|--help
```

## Description

`tidy` looks only at the cards listed in the `imports` of `board.json`. A card whose description starts with its title in bold loses that opening. A card whose bold opening is not its title is left as it is, and the output says so.

Each change is a normal write: the card's file changes, and an `item.updated` event names its description. Run it with `--dry-run` first: it prints every change, before and after, and writes nothing.

`--imported` is required. It is the only tidy there is.

> [!NOTE]
> **New to this?** You need `tidy` only after an import. A board that you started empty never needs it.

## Options

- **`--imported`**

  Tidies the cards that the import made. Required.

- **`--as <PARTICIPANT>`**

  Who changes the cards. The default is the board's first person.

- **`--dry-run`**

  Prints every change and writes nothing.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the result as JSON.

## Examples

- See every change first:

  ```console
  codelead board tidy --imported --dry-run
  ```

  ```output
  Tidy of the imported cards as sam: 3 of 5 began with their title in bold; dry run: nothing was written.

    #8    Give the test database a second client
          before: **Give the test database a second client.** The orders tests need two clients to check that one client…
          after:  The orders tests need two clients to check that one client cannot see the other's orders.

    #9    0a: Keep the Basic Store on .NET Framework 4.5 for one more release?
          before: **0a: Keep the Basic Store on .NET Framework 4.5 for one more release?** The new API waits on this answer.
          after:  The new API waits on this answer.

    #10   Which payment provider for the new checkout?
          before: **Which payment provider for the new checkout?** Sam's call: two providers are compared in…
          after:  Sam's call: two providers are compared in `webapp/Controllers/OrdersController.cs`.

  Left as they are (2):
    #11   its bold opening is not its title · P1. Orders list
    #12   its bold opening is not its title · P2. Stock
  ```

- Make the changes:

  ```console
  codelead board tidy --imported
  ```

  ```output
  Tidy of the imported cards as sam: 3 of 5 began with their title in bold; the repetition is gone.
  …
  ```

- Forget `--imported`:

  ```console
  codelead board tidy
  ```

  ```output
  tidy needs --imported: it strips from each imported card's description the opening repetition of its title in bold (run it with --dry-run first to see every change).
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Tidied, or a dry run was printed. |
| `1` | A rule refused a write, for example a card file with problems. |
| `2` | No board, or no `--imported`. |

## Files

`tidy` reads `imports` in `board.json`, changes the imported card files in `items/`, and appends `item.updated` events to `events.jsonl`.

## See also

- [`codelead board import`](board-import.md): make the cards first.
- [`codelead board show`](board-show.md): read a card after the tidy.
- [`codelead board validate`](board-validate.md)
- [Files and events](../files-and-events.md#imports)
