---
title: codelead seed
description: Write a T-SQL script of plausible rows for a database that arrived as a definition without data.
kind: reference
area: cli
date: 2026-11-20
---

# codelead seed

**Applies to:** CodeLead 0.1 and later

`codelead seed` writes a T-SQL script of plausible rows for a database that has its tables but no data. You run the script yourself. CodeLead never connects to the database.

## Name

`codelead seed` - Writes a T-SQL script of sample rows from the System Model's schema.

## Synopsis

```console
codelead seed --schema <SCHEMA_JSON> --out <SEED_SQL>
              [--model <SYSTEM_MODEL>] [--users <JOURNEYS>] [--days <N>] [--rows <TABLE=N>]...
              [--tag <TEXT>] [--person-id <USER=VALUE>]... [--person-value <USER=COLUMN=VALUE>]...
codelead seed -h|--help
```

## Description

Sometimes an old application arrives with its database's definition, and no data. A walk of the application then finds empty screens, and many journeys cannot be checked. `codelead seed` writes rows that fit the schema, so that the screens have something to show.

It reads:

- the schema, `schema.json`, that [`codelead model`](codelead-model.md) writes: the tables, the columns and the keys;
- the [System Model](../modernization/system-model.md), for its identity section (the people and the roles) and its rules that read a clock (the rows of each day);
- a walk's users, for one person per user with the account that it gives.

It writes three files: the script, a markdown file that says what each table holds and why, and a keys file.

> [!NOTE]
> **New to this?** *Seed data* is sample data that you put in an empty database. Then people can use and test the application.

### Safe to run again

The script runs in one transaction. It first deletes the rows that it inserted before, by the tag or by the keys in the keys file, in reverse dependency order. Then it inserts, then it counts each table. It never truncates or drops anything. When another program inserted a row that points at a seeded row, the database's key stops the delete, and the transaction changes nothing.

### Which tables get rows

A table gets rows only when something asks for them. That can be `--rows`, a rule that reads a clock, a seeded row that points at the table, or the people. The markdown file lists every table that was not seeded, and the `--rows` option that would seed it.

> [!TIP]
> **For experienced users:** a run covers `--days` days that end the day before the script runs, by the database server's clock. The day of the run is left to the walk. For `--rows`, `N` counts in three ways. A catalog table gets `N` rows in all. A day table gets `N` per person per day. A table that points at the people gets `N` per person.

The command is part of the modernization program, which is in CodeLead's Pro plan. In a folder without the Pro plan, it is refused before anything runs, with exit code 2. See [Licence and plans](../concepts/licence-and-plans.md).

## Options

- **`--schema <SCHEMA_JSON>`**

  Required. The schema read from the code. [`codelead model`](codelead-model.md) writes it, reconciled with the database's definition when one was given.

- **`--model <SYSTEM_MODEL>`**

  The System Model: its identity section (the people, the roles) and its rules that read a clock (a day's rows).

- **`--users <JOURNEYS>`**

  A walk's journeys spec, a journeys draft, or a file of users alone. One person per user, with the account that it gives. Only its identity and its users are read. The default is one person per role of the model.

- **`--days <N>`**

  The days of the run, ending the day before the script runs. The default is 30.

- **`--rows <TABLE=N>`**

  The rows of a table. Give it again for more tables. See the tip above for how `N` counts.

- **`--tag <TEXT>`**

  The tag that marks the seeded rows, in a column that the code never reads. The default is `codelead-seed`.

- **`--person-id <USER=VALUE>`**

  The key of a user's row in the people table: an identifier that the directory gives the account. Give it again for more users.

- **`--person-value <USER=COLUMN=VALUE>`**

  A column of a user's row in the people table: the value that a setting of the configuration looks a person up by. Give it again for more values.

- **`--out <SEED_SQL>`**

  Required. The script. `<out>.md` and `<out>.keys.json` are written beside it. The folder must exist.

## Examples

- Seed 5 products and 3 clients for the Basic Store sample, over 7 days:

  ```console
  codelead seed --schema docs/codelead/rebuild/schema.json --model docs/codelead/rebuild/system-model.json --rows Products=5 --rows Clients=3 --days 7 --out seed/seed.sql
  ```

  ```output
  Seed of 8 rows in 2 tables: Clients 3, Products 5.
  Not seeded (nothing asks rows of them): OrderItems, Orders.
  Written: seed/seed.sql
  Written: seed/seed.sql.md
  Written: seed/seed.sql.keys.json
  ```

  The start of `seed/seed.sql`:

  ```output
  -- Seed rows written by codelead seed from docs/codelead/rebuild/schema.json and docs/codelead/rebuild/system-model.json.
  -- 8 rows in 2 tables; a run of 7 days ending the day before the script runs (the database server's clock: today is left to the walk). Tag: codelead-seed.
  -- It deletes only the rows it inserted before, then inserts, in one transaction, then counts; it never truncates or drops anything.
  …
  SET NOCOUNT ON;
  SET XACT_ABORT ON;
  BEGIN TRANSACTION;

  -- The rows inserted before, in reverse dependency order.
  DELETE FROM [dbo].[Products] WHERE [Id] IN (1, 2, 3, 4, 5);
  DELETE FROM [dbo].[Clients] WHERE [Id] IN (1, 2, 3);

  -- Clients: 3 rows (asked for).
  INSERT INTO [dbo].[Clients] ([Id], [Address], [City], [ContactName], [Country], [CreatedOn], [Email], [Name], [Phone]) VALUES (1, N'1 Example Street', N'Lyon', N'Contact name 1', N'Country 1', DATEADD(minute, 540, CAST(DATEADD(day, -7, CAST(GETDATE() AS date)) AS datetime)), N'client1@example.com', N'Client 1', N'555-0101');
  …
  COMMIT TRANSACTION;
  ```

- Run it with no `--rows`. Nothing in the model asks for rows, and the markdown says how to ask:

  ```console
  codelead seed --schema docs/codelead/rebuild/schema.json --model docs/codelead/rebuild/system-model.json --out seed/seed.sql
  ```

  ```output
  Seed of 0 rows in 0 tables: none.
  Not seeded (nothing asks rows of them): Clients, OrderItems, Orders, Products.
  Written: seed/seed.sql
  Written: seed/seed.sql.md
  Written: seed/seed.sql.keys.json
  ```

  In `seed/seed.sql.md`:

  ```output
  ## Not seeded

  - `Clients`: the code writes it, and no rule that reads a clock does, nothing seeded points at it, and it holds no person; --rows Clients=<n> seeds it.
  …
  ```

- Name an `--out` folder that does not exist. Nothing is written, and the exit code is 2:

  ```console
  codelead seed --schema docs/codelead/rebuild/schema.json --model docs/codelead/rebuild/system-model.json --out seed/seed.sql
  ```

  ```output
  Refused: the folder of --out does not exist: <your project>/seed. Nothing was written.
  ```

## Exit codes

| Code | Meaning |
|---|---|
| `0` | The script was written, and everything asked for was seeded. |
| `1` | The script was written, and something could not be seeded. The markdown lists it. |
| `2` | Nothing was written: the folder does not have the Pro plan, an input is missing or cannot be read, or an option was refused. |

## Files

| File | What `seed` does with it |
|---|---|
| The `--schema`, `--model` and `--users` files | Reads them. |
| `<out>` | Writes the T-SQL script. |
| `<out>.md` | Writes what each table holds and why, and what was not seeded. |
| `<out>.keys.json` | Writes the keys that the script inserts. The next script reads it, to delete those rows too. |

## See also

- [`codelead model`](codelead-model.md): writes the schema.
- [The System Model](../modernization/system-model.md)
- [The rebuild](../modernization/rebuild.md): a `sample-seed` database.
- [`codelead walk`](codelead-walk.md): the walk that needs the rows.
- [Journeys format](../reference/journeys-format.md): the users that `--users` reads.
