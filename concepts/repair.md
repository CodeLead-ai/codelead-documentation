---
title: Repair
description: What a repair round is, what the model sees in it, how CodeLead remembers earlier rounds, and why the loop stops when it starts to go in circles.
kind: concept
area: concepts
date: 2026-11-20
---

# Repair

A **repair** is a second, smaller change that fixes what the first change broke. When the [gate](validation-and-the-gate.md) refuses a change on the code, a **repair round** starts. It asks a model for a small fix, checks it, applies it after your yes, and runs the gate again. CodeLead remembers every round of the task, and it stops the loop when a round would repeat an earlier one.

**In short**
- One repair round is three commands: [`/repair`](../chat-commands/repair.md), then [`/edit`](../chat-commands/edit.md), then [`/validate`](../chat-commands/validate.md).
- The model sees the failure first, the files that the failure names, the files that this task changed, and what the earlier rounds tried.
- A round that would bring a file back to an earlier state is a **cycle**. At the first cycle CodeLead tries once more with a wider view. At the second, it stops and hands the task to you.
- A failure of the environment, such as a missing tool, is never repaired: a patch cannot fix it.

## Why it exists

A first patch often fails on a detail. A `using` line is missing, a test expects another value, or the old compiler does not know a language feature. The fastest fix is a small patch aimed at the error, not a new start.

Repair loops have a known danger. A model can undo its own last fix, then do it again, and spend money on each round. CodeLead gives every round the history of the task, and it compares the files with their earlier states. A loop that goes in circles stops, and the result says what each round tried.

## How it works

### One round

1. `/validate` fails on the code. Its result says `next: /repair`.
2. `/repair` reads the last failed validation and asks a model for the smallest patch that makes it pass. CodeLead runs the [patch check](the-patch-check.md) on it. Nothing changes yet.
3. `/edit` asks `Apply this patch? [Y/n]`. A yes applies it.
4. `/validate` runs the gate again. A pass lands the change. A failure starts the next round.

Here is the end of each command of a round. These lines come from a session on the Basic Store sample, in the third round of a task:

```output
✔ Repair round 3 — the repair for dotnet test is ready to apply (1 file); nothing is changed yet; next: /edit
✔ Repair round 3 — applied the repair to 1 file; not validated yet; next: /validate
✔ Validation passed after repair round 3: 4 commands
```

### What the model sees

- **The failure, first.** Every failing test and every compiler error is counted and given before the rest, within 20,000 characters.
- **The files that the failure names**. These are the paths in the errors. They are also the files that declare the types and members that the messages and the stack name. For example, "X does not implement I" finds the file of `I` too.
- **The files that this task changed.** The patches of this session, and the earlier increments of the same chain of evidence records. Other changes in your working tree are listed apart, so that they do not confuse the model.
- **The history, from the second round on**. A section named "Earlier repair rounds of this task" holds each earlier round. It gives the round's diagnosis, its diff and the verdict that followed it. It ends with a rule: "Do not return a file to a state an earlier round already tried: that state failed as shown."

### Cycles

CodeLead keeps, for each file that a round touched, the state that the round left. A state is the file's code with comments and spacing left out. So a round that writes the same code with other comments leaves the same state. CodeLead also keeps a fingerprint of each failure, without times, line numbers or machine paths.

- **The first cycle.** When a repair would bring a file back to an earlier state, CodeLead escalates once. The next prompt names the cycle, widens the scope, and lifts the plan's non-goals.
- **The second cycle.** The loop stops for you. The result says what each round tried.
- **After that.** `/repair` asks no model while the failure stays the same.

<!-- described, not captured -->
```output
✖ The repair loop stopped: this repair returns webapp.Characterization/Fakes/FakeApplicationSignInManager.cs to its state after repair 1, which failed with Expected 302, got 200; next: Fix it by hand, then /validate
```

## What it is not

- **Not automatic in a session that you drive.** In [`/implement`](../chat-commands/implement.md), a failed gate ends with the way on `/repair`. You decide whether to repair, to fix the code by hand, or to stop.
- **Not a new attempt from zero.** A repair changes as little as it can, and it starts from the files as they are now.
- **Not a way around the gate.** A repair lands only when the gate passes. `/validate --override` is a manual pass, recorded as manual.
- **Not for the environment.** A missing SDK, a database that does not answer or a timeout is not a code error. See [Validation and the gate](validation-and-the-gate.md).

> [!TIP]
> **For experienced users:** set the [repair role](models-and-roles.md) to a stronger model, and keep a local model for the rest. See [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md).

## Details

- **Which model.** The `repair` role when the configuration sets one. Otherwise the role of the patch strategy.
- **Rounds that run by themselves.** A pin or an in-place transform repairs itself, up to `modernization.repair.rounds` rounds (default 2). It asks once first: `The gate refused the pins: <failure>. Repair it? [Y/n]`. [`codelead implement`](../cli/codelead-implement.md) takes `--repair-rounds` (default 2).
- **New apps.** A build step uses [`/build-step --repair`](../chat-commands/build-step.md). An increment of [`/build-incremental`](../chat-commands/build-incremental.md) has at most 3 attempts. Each new attempt carries the failure, for example "Fix ONLY the compiler errors below".
- **The evidence.** Each round's validation writes a record with `command: "repair"`, `follows` (the record before it), `repair_round` and `repair_rounds[]`. A cycle is kept in `repair_rounds[].cycle`, and a stopped loop has the stop reason `repair_cycle`. See [Evidence records](evidence-records.md).
- **The file limit**. CodeLead ranks the files. First come the files that the errors name by path, then the files that the failure's names lead to. Then come the task's files that use them, then the rest. A large file counts as one slot and a small file as a quarter. A file left out says why.
- **Non-goals.** A repair may cross a non-goal of the plan only for a file that this task itself wrote.

## Related content

- [`/repair`](../chat-commands/repair.md), [`/edit`](../chat-commands/edit.md), [`/validate`](../chat-commands/validate.md)
- [`/build-step`](../chat-commands/build-step.md) and [`/build-incremental`](../chat-commands/build-incremental.md)
- [`codelead implement`](../cli/codelead-implement.md)
- [Validation and the gate](validation-and-the-gate.md)
- [Models and roles](models-and-roles.md)
- [Evidence records](evidence-records.md)
- [Use a stronger model for repairs](../how-to/use-a-stronger-model-for-repairs.md)
- [Tutorial: change existing code with evidence](../tutorials/change-existing-code.md)
