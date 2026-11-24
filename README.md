# CodeLead documentation

This is the official documentation of **CodeLead**.

CodeLead lets AI models write code for you, under rules that you control. A model proposes a change. CodeLead checks it. You approve it. Your own build and tests judge it. CodeLead then records what happened, so that you can prove it later.

**Start here: [index.md](index.md)**

## Where to start

| You are | Read first | Then |
|---|---|---|
| New to CodeLead | [Install CodeLead](get-started/install.md) | [Quickstart: CodeLead in VS Code](get-started/quickstart-vscode.md) |
| Already a user of coding agents | [How CodeLead works](concepts/how-codelead-works.md) | [Quickstart: change existing code](get-started/quickstart-existing-code.md) |
| A team lead or an architect | [Evidence records](concepts/evidence-records.md) | [Modernization overview](modernization/index.md) |

Not sure which one you are? Read [Choose your path](get-started/choose-your-path.md).

## What is in this repository

| Folder | What it holds | Pages |
|---|---|---|
| [get-started/](get-started/index.md) | Install CodeLead and run your first task. | 6 |
| [tutorials/](tutorials/index.md) | Lessons that you follow from start to end on a sample. | 7 |
| [how-to/](how-to/index.md) | Short answers to "how do I…?" | 15 |
| [concepts/](concepts/index.md) | How CodeLead works, and why. | 22 |
| [cli/](cli/index.md) | Every command of the `codelead` command line. | 40 |
| [chat-commands/](chat-commands/index.md) | Every command that you type in a session, such as `/implement`. | 30 |
| [modernization/](modernization/index.md) | How CodeLead modernizes an old application, stage by stage. | 11 |
| [extension/](extension/index.md) | CodeLead for Visual Studio Code. | 15 |
| [pinboard/](pinboard/index.md) | The board between you and your coding agents. | 22 |
| [analyst/](analyst/index.md) | The Analyst and the Explorer, which read a running application. | 15 |
| [reference/](reference/index.md) | File formats, settings, messages and exit codes. | 20 |
| `media/` | The pictures that the pages show. | |

[index.md](index.md) is the hub. [toc.yml](toc.yml) lists every page in reading order.

## How to read these pages

- **You can read them here, on GitHub.** Every link is a relative link, so it works in the browser.
- **Each page starts with "In short".** It gives the main points in a few lines. Read on only when you need more.
- **Two kinds of boxes help different readers.** "New to this?" explains a word or an idea. "For experienced users" gives the detail that an expert looks for.
- **The examples use one sample**, a small store application called the Basic Store, with clients, products and orders. You do not need it to read the pages.
- **Placeholders are in angle brackets.** `<your project>` means the folder of your own project. `<your key>` means your own key. Never copy a placeholder as it is.
- **Output is real when it can be.** Most output blocks were captured from a real run. A block that was written by hand is marked so in the page's source.

## Which versions these pages describe

- CodeLead 0.1.0
- CodeLead for VS Code 0.7.16

A page says so when it is about a later or an earlier version.

<!-- copied:start -->
Last copied on **2026-11-24**: 205 pages, 40 pictures.
<!-- copied:end -->

## How the pages are kept true

- **They are written from the product.** The writers ran each command and copied what it printed.
- **A checker reads every page before it is published.** It refuses a broken link, a command or an option that has no line in its page, a sentence of more than 25 words, and any private name, machine path or key.
- **The pages live beside CodeLead's code.** When CodeLead changes, its page changes in the same step. This repository is a copy of those pages. It is published from there.

For this reason, please do not edit a page here. A change made here is lost at the next copy. Tell us about the problem instead.

## Found a mistake?

Open an issue in this repository. Three things help us fix it fast:

1. The page, for example `cli/codelead-doctor.md`.
2. The sentence or the output that is wrong.
3. What you saw when you ran the command.

Please do not put a key, a password or private code in an issue.

## Licence

MIT (see [LICENSE](LICENSE)). You can use these pages, change them and share them. Keep the copyright notice.

## Build a website from these pages

The pages are plain Markdown files with a small header (title, description, kind, area, date). `toc.yml` gives the order and the names of the pages, in the format that documentation site builders such as docfx read. No website is built from this repository yet.
