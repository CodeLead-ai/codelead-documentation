---
title: codelead board schedules
description: A schedule is a person's standing ask - at its times it puts one card in Ready, and the board gives that card to an agent like any other.
kind: reference
area: pinboard
date: 2026-11-25
---

# codelead board schedules

**Applies to:** CodeLead 0.1 and later

`codelead board schedules` makes and runs the board's **schedules**. A schedule is your standing ask. At its times it puts one card in the **Ready** column, and the board gives that card to an agent like any other Ready card.

## Name

`codelead board schedules` - Lists, makes, changes, pauses, removes and runs the board's schedules.

## Synopsis

```console
codelead board schedules [list] [--board <DIR>] [--json]
codelead board schedules add <NAME> --as <PERSON> <RULE> <CARD> [--zone <ZONE>] [--dry-run]
codelead board schedules change <NAME> --as <PERSON> [<RULE>] [<CARD>]
codelead board schedules pause|resume|remove <NAME> --as <PERSON>
codelead board schedules run [--board <DIR>] [--json]
codelead board schedules -h|--help
```

`<RULE>` is one of `--every <INTERVAL> [--from <LOCAL_TIME>]`, `--daily --at <HH:MM>`, or `--weekly <DAYS> --at <HH:MM>`.

`<CARD>` is `--title <TEXT>`, with `--field` for the three things Ready asks for, or `--from-card <ID>`.

## Description

A schedule belongs to the board, not to an agent. So every agent gets scheduled cards, whichever it is. And every scheduled run leaves a card with its record: nothing runs unseen. The board still starts no agent and calls no model.

**Only a person** makes, changes, pauses or removes a schedule. The commands refuse an agent.

**The card must pass the Ready check when you make the schedule.** It says three things:

- how to tell it is done (`definitionOfDone`);
- what it may touch (`mayTouch`);
- its limit (`limitMinutes`, `limitMoney`, or both).

So no schedule can make a card the board would not accept.

**When it is due** is one of three rules:

- **Every so often:** `--every 30m`, `--every 2h`, `--every 1d`. It counts from `--from`, a local time, or from now.
- **Every day at a time:** `--daily --at 07:30`.
- **On days of the week at a time:** `--weekly mon,thu --at 09:00`.

Times are local to the machine. The schedule writes its time zone down when you make it, and reads its times in that zone from then on. `--zone` names another zone. Every time is shown in that zone and in UTC.

> [!NOTE]
> **New to this?** A scheduled card is the one card that enters Ready without your move at that moment. It is allowed because you made the schedule: the card is your ask. You are its author, and its file names the schedule.

**Running what is due.** `codelead board schedules run` makes, once, the card of each schedule that is due now. The page's process does the same by itself while it runs: at its start, and then every minute. So schedules run only while something runs them. That is the page, an app that hosts the board, or this command from your system's own scheduler.

Three rules keep the cards few:

- **One card for missed times.** Say nothing ran for three days. The next run makes one card, not three. Its description lists the times it stands for.
- **Not while the last one is open.** A schedule makes no new card while its last one is still in Ready or In progress. It says why. Once that card is in For review or Done, the next time makes the next card.
- **A paused schedule makes nothing.** The times it misses while paused are not made up when you resume it.

Two processes that run at the same moment make one card, not two.

## Options

- **`--as <PERSON>`**

  Who makes, changes, pauses, resumes or removes the schedule. It must be a person of the board. Required for those commands.

- **`--every <INTERVAL>`**, **`--from <LOCAL_TIME>`**

  Every N minutes, hours or days: `30m`, `2h`, `1d`. `--from "2026-11-23 09:00"` says where the count starts. The default is now.

- **`--daily`**, **`--weekly <DAYS>`**, **`--at <HH:MM>`**

  Every day, or on the named days (`mon,thu`), at the local time `--at` gives.

- **`--zone <ZONE>`**

  The time zone the rule is read in, such as `Europe/Berlin`. The default is this machine's zone.

- **`--title <TEXT>`**, **`--body <MARKDOWN>`**, **`--body-file <PATH>`**, **`--type <TYPE>`**, **`--tag <TAG>`**

  The card the schedule makes. The type is `task` unless you say another.

- **`--field <NAME=VALUE>`**

  One of the fields Ready asks for: `definitionOfDone`, `mayTouch`, `limitMinutes`, `limitMoney`. Give `mayTouch` a JSON list for several places. Repeatable.

- **`--for <AGENT>`**

  The agent the card is for. It adds the tag `for:<AGENT>`. The default is any agent.

- **`--from-card <ID>`**

  Takes the card from a card of the board: its type, title, description, tags and Ready fields. The other options change them.

- **`--dry-run`**

  Checks everything and writes nothing.

- **`--board <DIR>`**

  The board folder. The default is `docs/codelead/board` of the repository you are in.

- **`--json`**

  Prints the answer as JSON.

## Examples

These outputs come from a board in a scratch folder. The card numbers and times are examples.

- Make a schedule: every Monday at 09:00, check the dependencies.

  ```console
  codelead board schedules add check-deps --as sam --weekly mon --at 09:00 --zone Europe/Berlin --title "Check the dependencies" --body 'Run `npm outdated`. Update what is safe.' --field definitionOfDone="npm outdated lists nothing that is safe to update; npm test passes." --field mayTouch='["package.json", "package-lock.json"]' --field limitMinutes=20
  ```

  ```output
  Made the schedule check-deps: every Monday at 09:00 (Europe/Berlin).
  Its first card is due Mon 2026-11-30 09:00 Europe/Berlin (2026-11-30 08:00 UTC), in Ready.
  docs/codelead/board/schedules/check-deps.json
  ```

- List the schedules:

  ```console
  codelead board schedules
  ```

  ```output
  1 schedule, each time in its zone and in UTC:

  check-deps: every Monday at 09:00 (Europe/Berlin), made by sam on 2026-11-25 14:01 UTC
    next: Mon 2026-11-30 09:00 Europe/Berlin (2026-11-30 08:00 UTC)
    last card: none yet
  ```

- Run what is due. Here a second schedule, `tidy-notes`, was due:

  ```console
  codelead board schedules run
  ```

  ```output
  The schedule tidy-notes put #5 in Ready: Tidy the notes (due Wed 2026-11-25 15:02 Europe/Berlin, 2026-11-25 14:02 UTC).
  docs/codelead/board/items/0005-tidy-the-notes.md
  ```

  The card is tagged `scheduled:tidy-notes`, and Sam is its author. Its description ends with the schedule's own line:

  ```output
  Made by the schedule `tidy-notes` (Sam's): every minute from Wed 2026-11-25 15:02, Europe/Berlin. Due Wed 2026-11-25 15:02 Europe/Berlin (2026-11-25 14:02 UTC).
  ```

- Run again when nothing is due:

  ```console
  codelead board schedules run
  ```

  ```output
  Nothing is due. The next is tidy-notes, Wed 2026-11-25 15:03 Europe/Berlin (2026-11-25 14:03 UTC).
  ```

- Leave out the three things that Ready asks for. The schedule is refused, in the card check's own words:

  ```console
  codelead board schedules add check-deps --as sam --weekly mon --at 09:00 --title "Check the dependencies"
  ```

  ```output
  The schedule's card cannot go to Ready yet: it does not say how to tell it is done (definitionOfDone), it does not name the repository, folder or files it may touch (mayTouch), and it names no limit in minutes or money (limitMinutes, limitMoney). Nothing was written: set what is missing, then make the schedule again.
  ```

- Try it as an agent:

  ```console
  codelead board schedules add check-deps --as codelead --weekly mon --at 09:00 --title "Check the dependencies"
  ```

  ```output
  Refused: codelead is an agent and may not make a schedule: a schedule is the person's standing ask, which puts a card in Ready at its times. Ask the person in a card's thread. Nothing was written.
  ```

- Pause a schedule, then resume it:

  ```console
  codelead board schedules pause check-deps --as sam
  codelead board schedules resume check-deps --as sam
  ```

  ```output
  Paused the schedule check-deps: it makes no card until it is resumed, and the times it misses meanwhile are not made up.
  Resumed the schedule check-deps: the times it missed while paused are not made up. Its next card is due Mon 2026-11-30 09:00 Europe/Berlin (2026-11-30 08:00 UTC).
  ```

- Remove a schedule. The cards it made stay:

  ```console
  codelead board schedules remove tidy-notes --as sam
  ```

  ```output
  Removed the schedule tidy-notes (schedules/tidy-notes.json): it makes no more cards; the cards it made stay on the board.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | Done. `run` always exits `0`, also when nothing is due. |
| `1` | Refused: an agent asked, the card does not pass the Ready check, the board has no Ready column, or the name is taken. |
| `2` | No board, no schedule of that name, or the options are wrong: no `--as`, no rule, two rules, a bad time or zone. |

## Files

Each schedule is one file: `schedules/<NAME>.json` in the board folder, beside `items/`. Git carries it like a card.

A card that a schedule made has the key `schedule` in its file: the schedule's name, its maker, and the time it was due for.

The commands append `schedule.added`, `schedule.changed`, `schedule.paused`, `schedule.resumed` and `schedule.removed` to `events.jsonl`. A run appends `schedule.ran` with the card it made, or `schedule.skipped` with the reason. See [Files and events](../files-and-events.md).

## See also

- [`codelead board take`](board-take.md): how an agent gets a Ready card, scheduled or not.
- [`codelead board add-ready`](board-add-ready.md): give an older board its Ready column.
- [`codelead board serve`](board-serve.md): the page's process, which runs the schedules while it runs.
- [`codelead implement --ready`](../../cli/codelead-implement.md): CodeLead takes its cards from Ready.
- [Rules for agents](../rules-for-agents.md#ready-is-the-persons)
