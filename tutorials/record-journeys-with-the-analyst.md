---
title: Record journeys with the Analyst
description: Record one journey on the running Basic Store sample, draft it into a journeys file, check the file, and replay it on the application.
kind: tutorial
area: tutorials
date: 2026-11-20
---

# Record journeys with the Analyst

In this tutorial, you record what a person does on the Basic Store sample, while it runs. The [Analyst](../analyst/index.md) watches the browser, cuts your journey into steps, and keeps what each step showed. You then turn the recording into a **journeys file**: a specification that CodeLead's walk can replay. No model is called at any step.

**In short**
- You record one journey, "create a product", in the control window and a role's browser window.
- You mark what the page must show with **Expect this**.
- You draft the journeys file, check it, and replay it on the running application.
- You know where the files are, and what to do when a step does not pass.

In this tutorial, you:

> [!div class="checklist"]
> - Check the plan and the browser
> - Start the Basic Store sample
> - Start a recording session
> - Record the journey "create a product"
> - Finish the session
> - Draft the journeys file
> - Check the journeys file
> - Replay the journey

## Prerequisites

What you must know:

- You know how to open a terminal and run a command.
- You can use the Basic Store sample in a browser: its products and its **New product** page. You do not need to know ASP.NET.

What you must have:

- CodeLead installed, with Playwright's Chromium. See [Install CodeLead](../get-started/install.md). [`codelead setup`](../cli/codelead-setup.md) installs Chromium.
- A licence of the Pro plan in the project folder. The Analyst is in the Pro plan. See [Licence and plans](../concepts/licence-and-plans.md).
- The Basic Store sample, and a way to run it. On Windows, Visual Studio or IIS Express runs it. See [Start the old application for CodeLead](../modernization/source-run-profile.md).
- A copy of the sample's database that you may change. The replay creates a product each time that it runs.
- About 30 minutes.

> [!NOTE]
> **New to this?** A *journey* is a path that a user takes through the application, such as "create a product". A *step* is one part of it that ends where something visible happens. A *role* is a kind of user, such as a clerk. See [Roles, journeys and steps](../analyst/concepts.md).

## Check the plan and the browser

1. Open a terminal in the folder of the sample, the folder that holds `sampleapp.slnx`.
2. Check the plan of the folder with [`codelead licence`](../cli/codelead-licence.md):

   ```console
   codelead licence
   ```

   The line `Plan:` must say `pro`. Without a Pro licence, it says:

   ```output
   Licence (docs/codelead/licence.json)
     No licence file. A project without one is not warned.
     Plan: free (no licence file)
   ```

   An Analyst command in that folder then stops with: `The Analyst is in CodeLead's Pro plan; this folder's licence is missing. Ask for a licence that includes it.`

3. Check the browser with [`codelead doctor`](../cli/codelead-doctor.md). Its line for Chromium must be `ok`:

   ```output
   ok       Playwright's Chromium (the walk, the Analyst)         playwright 1.61.1; Chromium starts (…)
   ```

## Start the Basic Store sample

1. Start the sample. On Windows, in PowerShell, run IIS Express on the web project's folder:

   ```console
   & "C:\Program Files\IIS Express\iisexpress.exe" /path:<your project>\WebApplication1 /port:5080
   ```

2. Open `http://localhost:5080/` in a browser. The store's home page opens, with the button **New product**.
3. Close that browser tab. The Analyst opens its own browser.

This tutorial uses the address `http://localhost:5080/`. When your copy runs at another address, use that one in every step.

## Start a recording session

1. In the folder of the sample, start the recorder with [`codelead analyst record`](../analyst/reference/analyst-record.md):

   ```console
   codelead analyst record
   ```

   A small **control window** opens. It drives the session. Nothing that you do in it is recorded.

2. Fill the setup:
   - **Application address**: `http://localhost:5080/`
   - **Application name**: `Basic Store`
   - Choose **Its own sign-in form**. The store needs no sign-in to create a product, and this choice works for it.
3. Select **Start session**. CodeLead checks that the address answers.

   The terminal prints the control window's address and the sign-in mode:

   <!-- described, not captured -->
   ```output
   The control window's page: http://127.0.0.1:53237/2MlrZ2…/ (served on this machine alone; its address holds its key).
   Form sign-in: each role signs in through the application's own pages, in a window of its own with its own cookies.
   ```

4. Type `clerk` in **Role**, and select **Start role**. A Chromium window opens at the store's home page. Its title begins with the role's name.

> [!TIP]
> When **Start session** says that the address does not answer, the store does not run at that address. It must answer within 5 seconds. Start it again, and check the address in a browser.

## Record the journey "create a product"

1. Type `create a product` in **Journey**, and select **Start journey**.
2. In the clerk's window, select **New product**.
3. Fill the form: **SKU** `LAMP-001`, **Name** `Desk lamp`, **Unit price** `24.50`.
4. Select **Create product**. The product's page opens, with the message `Product "Desk lamp" was created.`
5. In the control window, select **Expect this**. Then, in the clerk's window, click the message `Product "Desk lamp" was created.`
6. Select **End journey**.

The terminal says each step as the recorder cuts it:

<!-- described, not captured -->
```output
Journey "create a product" started (clerk): use the application in clerk's window, and the recorder cuts the journey into steps as it goes.
s01 cut (the page went to another address): / → /Products/Create, 2 actions, 0 requests: New product on /.
s02 cut (a form was sent): /Products/Create → /Products/Details/7, 5 actions, 1 request: Create product on /Products/Create.
Journey "create a product" ended: 2 steps (s01 to s02).
```

You did not mark the steps. The recorder cut one step where the page went to another address, and one where a form was sent. The control window shows each step with its screenshot.

> [!TIP]
> When the SKU exists already, the store says `Another product already uses this SKU.` and the journey records a refusal. Use another SKU, or delete the step with **Delete** in the control window and record it again.

## Finish the session

1. Select **Finish session**, then **Yes, finish the session**. The files are written, and the clerk's window closes.
2. Read the terminal. It names the recording's folder:

   <!-- described, not captured -->
   ```output
   Recorded 2 steps into analyst-recordings/2026-11-20T09-30-00Z: recording.json, recording.md and the screenshots under steps/.
   The control window stays open for Draft journeys, Check and Replay on this recording: close it, or press Enter here (or Ctrl+C), to end.
   Next: codelead analyst draft analyst-recordings/2026-11-20T09-30-00Z/recording.json
   ```

3. Close the control window, or press **Enter** in the terminal.

Your folder's name is the time of your session. Use it in the next steps, in place of `2026-11-20T09-30-00Z`. The last line is the next command to run.

The recording keeps the actions, the requests, the page's headings and alerts, and a screenshot of each step. It never keeps a password. See [Record a journey](../analyst/record-a-journey.md#what-the-analyst-captures-and-what-it-never-captures).

## Draft the journeys file

1. Draft the journeys from the recording with [`codelead analyst draft`](../analyst/reference/analyst-draft.md):

   ```console
   codelead analyst draft analyst-recordings/2026-11-20T09-30-00Z/recording.json
   ```

   <!-- described, not captured -->
   ```output
   Drafted 2 steps into analyst-recordings/2026-11-20T09-30-00Z/journeys.json and analyst-recordings/2026-11-20T09-30-00Z/journeys.md.
   To confirm (journeys.md): 0 refused, 0 secret-looking, 0 with nothing to assert, 0 other.
   analyst-recordings/2026-11-20T09-30-00Z/journeys.json: no problems; CodeLead can walk it.
   ```

2. Open `journeys.md`. It starts with the list **To confirm**. Here every group says `None.`
3. Read the steps under the list. Each step says what it does and what it expects. The second step expects the address of the product's page, the form's request, and the message that you marked.

The file `journeys.json` holds the same journey for the walk:

```json
{
  "schema": "codelead.journeys/1",
  "app": "Basic Store",
  "users": { "clerk": {} },
  "journeys": [
    {
      "id": "create-a-product",
      "steps": [
        {
          "id": "s01",
          "title": "New product on /",
          "user": "clerk",
          "do": [{ "open": "/" }, { "click": ["New product"], "role": "link" }],
          "expect": [{ "path": "^/Products/Create$" }, { "text": "New product" }]
        },
        {
          "id": "s02",
          "title": "Create product on /Products/Create",
          "user": "clerk",
          "do": [
            { "fill": { "label": "SKU" }, "value": "LAMP-001" },
            { "fill": { "label": "Name" }, "value": "Desk lamp" },
            { "fill": { "label": "Unit price" }, "value": "24.50" },
            { "click": ["Create product"], "role": "button" }
          ],
          "expect": [
            { "path": "^/Products/Details/[^/]+$" },
            { "responses": { "method": "POST", "path": "^/Products/Create$" } },
            { "text": "Product \"Desk lamp\" was created." }
          ]
        }
      ]
    }
  ]
}
```

Your draft may hold more of what the page showed, such as its headings. The shape is the same.

## Check the journeys file

1. Check the file with [`codelead analyst check`](../analyst/reference/analyst-check.md). It needs no browser and no application:

   ```console
   codelead analyst check analyst-recordings/2026-11-20T09-30-00Z/journeys.json
   ```

   ```output
   analyst-recordings/2026-11-20T09-30-00Z/journeys.json: no problems (1 journey, 2 steps, 1 user).
   ```

   The exit code is `0`.

2. Run the check again after each change that you make by hand. A file that the walk cannot use names each problem with its place in the file. For example, a journey that opens a page by its address:

   ```output
   broken.json: 1 problem
     journeys[0].steps[0].do[0].open: opens "/Products/Create": a walk starts at "/" and reaches every other page by clicking (no address typed).
   ```

## Replay the journey

1. Check that the store still runs at `http://localhost:5080/`.
2. Delete the product `Desk lamp` that you created while you recorded, or restore your copy of the database. The replay creates the product again, and the store refuses a second product with the SKU `LAMP-001`.
3. Replay the journeys file with [`codelead analyst replay`](../analyst/reference/analyst-replay.md):

   ```console
   codelead analyst replay analyst-recordings/2026-11-20T09-30-00Z/journeys.json --base http://localhost:5080/
   ```

   A browser opens at the first page, and clicks its way through the journey. Each step passes or fails:

   <!-- described, not captured -->
   ```output
   PASS    create-a-product · New product on / — clicked "New product"
   PASS    create-a-product · Create product on /Products/Create — clicked "Create product"

   2 steps: 2 pass, 0 fail, 0 blocked. 0 findings.
   ```

4. Read the last line. The exit code is `0` only when every step passed, and nothing that no step expected happened.

When the product exists already, step `s02` fails: the store refuses the SKU, as its rule says. That is not a fault of the file. Delete the product, or restore the database, before each replay.

> [!TIP]
> When the first step fails and the others say `BLOCKED`, the store does not answer at the address. Start it, and replay again. For other failures, see [From a recording to a specification](../analyst/from-recording-to-spec.md#replay-on-the-old-application).

## Clean up

1. Stop IIS Express: press **Q** in its terminal.
2. Delete the product `Desk lamp` in the store, or restore your copy of the database.
3. Keep `analyst-recordings/` when you will modernize the store. The program's Observe stage counts each recording that holds a `journeys.json`. Otherwise, delete the folder.

## Next steps

> [!div class="nextstepaction"]
> [From a recording to a specification](../analyst/from-recording-to-spec.md)

- [Record a journey](../analyst/record-a-journey.md): the other controls, the keys and the sign-in modes
- [Explore an application](../analyst/explore-an-application.md): the screens that nobody records
- [Modernize a .NET Framework application](modernize-a-dotnet-framework-app.md)
- [The Analyst view](../extension/analyst-view.md) in VS Code
