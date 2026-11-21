---
title: Specification files
description: The format of a spec in plain words under docs/codelead/specs - the title, the Who line, the steps, the variations, what a step can check - and every file that CodeLead writes beside it.
kind: reference
area: reference
date: 2026-11-20
---

# Specification files

**Applies to:** CodeLead 0.1 and later

A **spec** is a markdown file that says, in plain words, what a person does in the application and what they then see. CodeLead turns it into a [journey](journeys-format.md) and walks it on the running application. The spec is the test: nobody keeps test code for it. This page is the format of the file, and of the files that CodeLead writes beside it.

## The files

| Path | Written by | In git | What it holds |
|---|---|---|---|
| `docs/codelead/specs/<name>.md` | you, from [`codelead spec new <name>`](../cli/codelead-spec.md) | Yes | The spec. |
| `docs/codelead/specs/<name>.journeys.json` | `codelead spec draft` | Yes | The drafted journey, `codelead.journeys/1`. See [Journeys format](journeys-format.md). |
| `docs/codelead/specs/<name>.journeys.md` | `codelead spec draft` | Yes | The draft in words: the steps as they will run, and what to confirm. |
| `docs/codelead/specs/<name>.walk.json`, `.walk.md` | `codelead spec walk` | Yes | The last walk's report, `codelead.walk/1`. |
| `docs/codelead/specs/<name>.screenshots/` | `codelead spec walk --screenshots` | Your choice | A screenshot after each step. |
| `docs/codelead/specs/coverage.json`, `coverage.md` | `codelead spec coverage` | Yes | The screens and forms that no spec names, `codelead.spec-coverage/1`. |
| `docs/codelead/specs/README.md` | `codelead spec report`, and again after `walk` and `coverage` | Yes | The library page of every spec, its check and its last walk. |
| `.codelead/spec-report.html` | the same | No | The tester's page, in the style of the run report. |
| `.codelead/specs/<name>/*.walk.json` | [`/implement --spec`](../chat-commands/implement.md) | No | The walks of the spec after a landing. |

`<name>` is the file's name without `.md`. It is the journey's id too. A name is lower-case letters, digits and hyphens, and starts with a letter or a digit, for example `export-invoice`.

## A spec

A spec for the Basic Store sample:

```markdown
# Show the stock on the order form

Who: a clerk

## Steps

1. Open the first screen and choose "Orders", then "Create New". I see "Create Order".
2. Choose "Coffee beans" in "Product". I see "In stock: 12".

## Also

- Out of stock: open the first screen, choose "Orders", then "Create New", then choose "Green tea" in "Product". I see "Out of stock" and I do not see "In stock:".
```

The check:

```console
codelead spec check docs/codelead/specs/stock-on-order-form.md
```

```output
docs/codelead/specs/stock-on-order-form.md: usable (0 notes)
```

## The parts of a spec

| Part | How you write it | Required |
|---|---|---|
| The title | The first heading, `# <what the person gets done>`. | Yes |
| The person | A line `Who: <who>`. | Yes |
| The steps | A heading `## Steps`, then a numbered list. | Yes |
| The variations | A heading `## Also`, then one bullet for each variation. | No |
| Notes | Every other line. | No |

Every other line is prose. CodeLead keeps it as you wrote it, and never checks it. The help lines that `codelead spec new` writes are prose too, so the new file is usable at once.

### Who

| You write | The walk |
|---|---|
| `Who: a new user` (or `a new person`) | Registers someone new through the screens. |
| `Who: a visitor` (or `anyone`, `anybody`) | Neither registers nor signs in: the application has no accounts. |
| `Who: a clerk`, `Who: an administrator` | Signs in as that role. The role must be one that the application's screens know. |

The heading may be `**Who:**` too. The article `a`, `an` or `the` is not part of the role.

### Steps

One step per line, numbered `1.` or `1)`. First what the person does: open, click, choose, fill in, sign in. Then what they see. A heading `### Steps` works too.

### What a step can check

| You write | The walk checks |
|---|---|
| `I see "Order saved"` | The text is on the page. |
| `I do not see "Error"` (also `I cannot see`, `I no longer see`) | The text is not on the page. |
| `I see a "Download" link` | A link with that text. |
| `I am on /orders` (also `I see the page /orders`, `the address is /orders`) | The address's path. |

- Put what the person sees **in quotes**: straight quotes `"…"`, curly quotes `“…”`, or single quotes `'…'` around words. A quoted word is never read as part of the step's own words.
- Join two checks with `and`, or a comma: `I see "Saved" and I do not see "Error"`.
- An apostrophe, as in `the task's title`, is not a quote.

### Also

Each bullet is one variation: a short name, a colon, then the steps. Each variation is its own journey: the same person, starting again from the first screen.

```markdown
## Also

- An empty form: open the first screen, choose "Register", then choose "Create account" without filling anything in. I see "Enter a name" and I do not see "Welcome".
```

## The template

`codelead spec new <name>` writes one filled example with a line of help under each part:

```console
codelead spec new add-a-product
```

```output
Written: docs/codelead/specs/add-a-product.md
Edit it, then: codelead spec check docs/codelead/specs/add-a-product.md
```

It never writes over a file that is there. Its example is a new user who registers, signs in and sees their name. Change it into your own.

## Gaps and notes

`codelead spec check` reads each spec and says what stops the walk from using it:

- **A gap** means that the spec cannot be used until you fix it. `spec check` then ends with exit code 1.
- **A note** means that the spec can be used, and that the change would let the walk judge more.

Each finding says what is wrong, how to fix it, and a line that you could write. A spec without a `Who:` line and without steps:

```output
docs/codelead/specs/broken.md: the spec does not say who the person is. Fix: add a Who: line: "a visitor" (an application with no accounts: nobody registers or signs in), "a new user" (the walk registers a fresh person through the screens) or "a <role>" (the walk signs in as that role). Example: Who: a clerk
docs/codelead/specs/broken.md: the spec has no ## Steps. Fix: add a ## Steps heading, and under it a numbered list of what the person does and sees. Example: 1. Open the invoice list from the menu. I see "Invoices".
docs/codelead/specs/broken.md: 2 gaps, 0 notes; fix the gaps, then draft it
```

A spec whose steps check nothing:

```output
docs/codelead/specs/vague.md:7: note: step 1 says nothing the walk can check. Fix: add what the person sees after it, so the walk can tell the step worked. Example: 1. Open the first screen and choose "Products". I see "…".
docs/codelead/specs/vague.md:8: step 2, the last step, says nothing the walk can check. Fix: end it with what the person sees: I see "…", I do not see "…", I see a "…" link, or I am on /path. Example: 2. Fill in the name, then save. I see "…".
docs/codelead/specs/vague.md: 1 gap, 1 note; fix the gap, then draft it
```

A step in the middle that checks nothing is a note. The last step must check something: without it, the walk cannot tell that the spec worked.

## From a spec to a walk

1. `codelead spec check` until the spec is usable.
2. `codelead spec draft` turns it into `<name>.journeys.json`, with a model call, and writes `<name>.journeys.md` for you to read.
3. `codelead spec walk --base <url>` walks it on the running application and writes `<name>.walk.json` and `.walk.md`.
4. After a change, `/implement --spec docs/codelead/specs/<name>.md` walks it again. CodeLead starts the application from [`.codelead/run.json`](run-file.md), or from a rebuild's run profile.

The walk is evidence beside the gate: a failing walk is a warning, and the change stays landed. See [Evidence record format](evidence-record-format.md#the-fields-of-a-walk).

## See also

- [`codelead spec`](../cli/codelead-spec.md) and [`/spec`](../chat-commands/spec.md)
- [`/implement`](../chat-commands/implement.md)
- [Journeys format](journeys-format.md)
- [Run file for specification runs](run-file.md)
- [From a recording to a specification](../analyst/from-recording-to-spec.md)
