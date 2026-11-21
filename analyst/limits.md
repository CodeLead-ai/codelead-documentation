---
title: Limits
description: What the Analyst and the Explorer do not read, do not do and cannot prove, so that a gap is a known limit and not a surprise.
kind: concept
area: analyst
date: 2026-11-20
---

# Limits

The [Analyst](index.md) and the Explorer see an application only from its screens and from the requests that its pages send. This page lists what they do not read, what they do not do, and what a replay cannot prove. Read it before you rely on a recording, a site map or a journeys file.

**In short**
- The Analyst reads web applications in Chromium only. Pop-up windows, frames of another origin and some actions, such as dragging, are not recorded.
- The Explorer does not know what a screen is for. It stops at its limits, and it never clicks a control that signs out, deletes or removes.
- A replay proves only the outcomes that the journeys file names. It does not look at the database, the e-mails or the server.
- Rules and facts hold only as far as the steps that they cite. The Analyst reads no code, and calls no model.

## Why this page exists

The Analyst gives evidence for each statement that it makes. Evidence has edges. A reviewer must know where they are, so that a missing rule is not read as "the application has no such rule".

> [!NOTE]
> **New to this?** "Observed" means that the Analyst saw it happen in a step. Something that the Analyst did not observe can still be true. It was only not seen.

## Applications and screens

- **Web applications only.** The application must answer at an `http` or `https` address. Desktop and mobile applications are not read.
- **Chromium only.** The Analyst drives Playwright's Chromium. It does not record in other browsers.
- **Pop-up windows** that the application opens are not recorded. The Explorer never follows a link that opens another window.
- **Frames of another origin** are not recorded. A frame of the page's own origin is recorded like the page.
- **A page that refuses scripts**, with a strict content policy, runs none of the recorder's code. On that page, the recorder sees the addresses and the requests only. It sees no click and no typing, and the keys and the bar do not work there.
- **Requests to other origins** are counted, not listed.
- **The query part of addresses** is dropped unless you record with `--keep-query`. Without it, a step that ends on `/Products?active=true` expects `/Products` with any query.

## Actions that a recording does not keep

A recording keeps five kinds of action. They are going to an address, typing into a field, a click, a form sent with Enter, and a choice in a drop-down. So these are not recorded as actions:

- Dragging and dropping.
- Moving the mouse over a menu that opens on hover only.
- Choosing a file to upload.
- A right-click, or a key press other than typing and Enter.

A journey that needs one of these cannot be replayed as recorded. Record around it, or add the step to the journeys file by hand. See [Journeys format](../reference/journeys-format.md).

## Values that a recording keeps

The recording keeps the values that you type into ordinary fields, such as a name or a price. It also keeps a screenshot of each step, which shows what the page displayed. Only passwords and secret-looking fields are masked. Record on test data, never on real customers' data. See [Record a journey](record-a-journey.md#what-the-analyst-captures-and-what-it-never-captures).

## What the Explorer does not do

- **It does not know purpose.** It finds screens and forms, but not what a journey is for, and not what a step must show. Only a person's recording gives that.
- **It stops at its limits**: 200 screens and 8 moves from the first page for each role, by default. It visits at most three addresses of one pattern that names a record, such as `/Products/Details/{id}`.
- **It never clicks** a control that signs out, logs out, deletes or removes. The screens behind those controls are not in the site map.
- **It never follows** a download, an address of another origin, `mailto:`, `tel:`, `javascript:`, or a control with no name.
- **It opens one level of toggles.** What a menu reveals is read, but a menu inside that menu is not opened.
- **It signs in in one way.** It looks for a sign-in form on the first page, or behind one **Log in** link. Any other sign-in needs the sign-in page as the address.

> [!IMPORTANT]
> **Observe only** sends no form, but it clicks every other button outside a form. A button that writes with a script, such as **Approve**, is clicked. Explore a copy of the application.

**Try forms** leaves boxes to tick, files and passwords as they are. A made-up value such as `Explorer Name` can fail a field's own rule. Then the form is refused for that reason, and not for the one that you wanted to see.

## What a replay cannot prove

A replay checks only the outcomes that the journeys file names. They are the address after each step, the writes that the server accepted or refused, and the texts on the page. It does not check anything else:

- **Not the data behind the page.** It does not read the database, the files, the e-mails that the application sends, or the server's logs.
- **Not other users.** A drafted step checks the page of the user who acts. It does not check what another user's screen shows.
- **Not every record.** The draft makes each id general, so `^/Products/Details/[^/]+$` passes on any product's page.
- **Not a refusal with status 200.** Many old applications refuse a save by showing the form again, with status 200. The walk takes a 200 as accepted. So such a step is checked by its address and its message, not by the server's answer.
- **Not completeness.** A replay that passes on the old application shows that the journey is right. It does not show that the journeys cover the whole application.

A replay also has these conditions:

- **Every journey starts from the first page**, and reaches each page by clicking. A journey that needs a sign-in made in another journey stops at its first step that needs it.
- **A replay writes.** A journey that creates a product creates one again on each replay. Use a copy whose database you can reset.
- **An honest server.** The walk refuses a server that signs a visitor in by itself with a development identity. See [The walk and parity](../modernization/walk-and-parity.md).

## What rules and facts cannot say

- **They hold only as far as the steps that they cite.** A rule seen once is a rule seen once. The Analyst reads no code. For what the code says, see [The dossier](../modernization/dossier.md).
- **They read only some requests**: the requests of the pages' scripts, every write, and the answer to a screen that was opened. That answer counts only when it is not the screen asked for. A 401, a 403 or a redirect to a sign-in page is a refusal. A 404, a server error or no answer is a screen that could not be opened, and gives no rule.
- **Some refused screens are not read as refusals.** One is a page answered 200 that only says "not allowed". Another is a redirect to a page without a sign-in form. A redirect counts as a refusal only when its page shows a password field.
- **An old application's refused save** with status 200 and the form shown again is not read as a refusal. It is read as an accepted write, after which the page showed a message: an `effect` rule. Only a sign-in refused this way is read as a refusal.
- **Users, not roles.** The recordings name each user by its key. A 403 says that the user lacks a role. The facts never name the role itself.
- **Old recordings without times.** When a recording does not say when each action happened, the rules guess which click sent each write. They guess the same way each time.

## Windows sign-in

- **One role.** The browser's session is one account for every window.
- **The computer's own session** answers only on a Windows computer that is signed in to the application's domain. A Mac, or a computer outside the domain, needs the account's password.
- **What was tested.** The Analyst's own tests use HTTP Basic authentication as a stand-in. NTLM, Negotiate and the computer's own session were not tested by the Analyst's own tests. Try Windows sign-in on your own network before you rely on it.

## Other known limits

- **The pro plan.** Every Analyst command needs it in the current folder. See [Licence and plans](../concepts/licence-and-plans.md).
- **A context** holds at most 4,000 characters.

## Related content

- [Analyst and Explorer overview](index.md)
- [Roles, journeys and steps](concepts.md)
- [Explore an application](explore-an-application.md)
- [Record a journey](record-a-journey.md)
- [From a recording to a specification](from-recording-to-spec.md)
- [The walk and parity](../modernization/walk-and-parity.md)
- [Journeys format](../reference/journeys-format.md)
