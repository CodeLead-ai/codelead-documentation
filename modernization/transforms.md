---
title: Transforms
description: The changes that CodeLead makes in the old code without changing its behaviour, each under the gate of the pins, and when each is refused.
kind: concept
area: modernization
date: 2026-11-20
---

# Transforms

A transform is a change to the old application's code that keeps its behaviour. Examples are a move to a library, a new project format, an adapter, or a security fix. CodeLead composes the request for a transform from the dossier, and runs it under the gate of the [pins](pins.md). The Upgrade in place stage runs the transforms that fit, one after another.

**In short**
- You know which transforms CodeLead composes by itself, and which ones a person writes.
- You know what each composed transform changes, what it keeps, and when it is refused.
- You know the rule "pin first", and that each landing waits for your yes in approve mode.

## Why transforms exist

Some parts of an old application can stay. A class library can move to current .NET while the old web project keeps running on it. A transform makes such a change in a small step, and the pins show that the behaviour did not change. It is the opposite of a rewrite: the code stays, and only what the transform names changes.

## Pin first

A transform runs under the gate of the pins. Without pins in the gate, there is no way to show that a change kept the behaviour, so CodeLead refuses before it composes anything:

```console
/modernize --transform T1 --scope webapp
```

```output
Pin first: the declared gate (.codelead/validation.json) carries no characterization suite, so no transform can be shown safe. Run /modernize --pin <project> for the projects in scope, or pass --without-pins to proceed on your own responsibility.
```

`--without-pins` runs the transform anyway, and every result says that behaviour changes will not be caught. Use it only on code that you can check another way.

## How a transform runs

Each transform is an [`/implement`](../chat-commands/implement.md) run that CodeLead composes:

1. **The composer** reads the dossier and the project files, and writes the request. The request names the files that change and the files that stay. It ends with the sentence `Change exactly what this request names and nothing else.`
2. **The preflight** checks the request. CodeLead answers its questions itself, and shows its answers once, in a card titled `CodeLead assumed`. When the preflight objects that the request cannot be done, the transform stops with those words.
3. **The patch**: the model writes it, and [the patch check](../concepts/the-patch-check.md) reads it.
4. **Your approval**: in approve mode, CodeLead asks `Apply this patch? [Y/n]` for each landing. Unattended, the answer is yes.
5. **The gate** builds the projects and runs the pins. When it refuses, CodeLead runs up to two [repair](../concepts/repair.md) rounds.
6. **The landing**: the change passed the gate, and an [evidence record](../concepts/evidence-records.md) records it.

A transform that does not land changes nothing past its last green step.

## The transforms that CodeLead composes

| Type | What it changes | What stays | Command |
|---|---|---|---|
| **T5D** | Moves the data layer of a web project into a new class library: the data context, its entities, their enums, the initializer. | Every namespace and type name. The controllers compile unchanged. The Identity context stays. | `/modernize --transform T5D --scope <WEB_PROJECT>` |
| **T1** | Makes projects SDK-style, and adds a current .NET target beside the old one. | The compiled files, the assembly names, the namespaces, every source file. | `/modernize --transform T1 --scope <PROJECTS> [--target <FRAMEWORK>]` |
| **T5** | Puts a seam (an interface) between the code and a static data gateway. | Every public method and constructor signature. | `/modernize --transform T5 [--scope <PROJECTS>]` |
| **T10** | Puts one unmaintained package behind an adapter, as the first step of replacing it. Never the framework that the application is built on. | The package itself, in this step. Every signature. | `/modernize --transform T10 --dependency <PACKAGE>` |
| **S** | Fixes security findings of the dossier. | Everything but the findings. | `/modernize --transform S --security all\|<CLASS>\|<IDS>` |

The rebuild has its own composed transform, `T9`, which builds a new application part by part. See [The rebuild](rebuild.md).

> [!NOTE]
> **New to this?** The number `T9` has two meanings. In the options report, T9 is a data migration, which a person writes. The command `/modernize --transform T9` is the rebuild. They share the number only.

### T5D: the data layer into a class library

An MVC application often keeps its data context and entities inside the web project. Then T1 has no library to upgrade, because a web project on System.Web has no current .NET target. T5D moves the data layer out, so that T1 can take it.

On the Basic Store sample, CodeLead composes this request (the start of it):

```output
Request: Extract the data layer of webapp into a new class library webapp.Data, with no behaviour change. Today webapp holds the EF data context StoreContext (DAL/StoreContext.cs) with the entities Client, Product, Order and OrderItem, the enum OrderStatus they use and the initializer StoreInitializer; ClientsController, HomeController, OrdersController and ProductsController hold the context directly (59 uses) and MvcApplication (Global.asax.cs) creates the initializer. Move these 7 files from WebApplication1/ to webapp.Data/, each to the same folder below it, with its content unchanged: …
```

The rest of the request describes the new project file, the reference from the web project, and the solution entry. The new project keeps the old format and framework. It also says what stays: the Identity context, the web project's packages, its `Web.config` and its connection strings.

T5D refuses, with the way out, when:

- no web project holds a data context: `No web project holds an EF data context, so T5D has nothing to extract.`
- the scope is a class library that holds the context already: it says to run T1 on it instead;
- a class library and the web project both hold a copy, which means that an earlier extraction did not finish;
- the library's folder or name is taken, or the name is not a valid project name;
- a moved file uses a type that stays in the web project.

> [!TIP]
> **For experienced users:** T5D wants a gate entry that builds the solution. Without one, it adds the note `No declared gate entry builds: declare one that builds sampleapp.slnx (msbuild or dotnet build) in .codelead/validation.json, or the extraction is not shown to compile.` See [Declare validation commands](../how-to/declare-validation-commands.md).

### T1: the platform upgrade in place

T1 converts each project in its scope to an SDK-style project file that targets the old framework and a current one, for example `net45` and `net8.0`. The request says to compile exactly the same files, to keep the names, and to change no source file. The old web project keeps building against the library, and a new application can use the current target. The default target is `net8.0`; `--target` names another.

Use T1 on class libraries. A web project on System.Web, such as an MVC 5 project, has no current .NET target, so the program never applies T1 to it. T1 refuses a scope with no project: `T1 has no scope: name the projects with --scope.`

### T5: a seam around a static gateway

T5 finds a class named like `*Data`, `*DB`, `*Gateway`, `*Repository` or `*Store` that the code calls statically. It adds an interface and a class that delegates to the gateway, and routes the callers through the interface. On the Basic Store sample, the controllers create their data context directly, so T5 has nothing to wrap:

```output
No static gateway call found in webapp (a class named *Data, *DB, *Gateway, *Repository or *Store called statically); the seam has nothing to wrap. Name the modules with --scope or write the request by hand.
```

That is why the options report proposes T5D for this sample.

### T10: a dependency behind an adapter

T10 takes one package that the options report names as unmaintained, for example `EntityFramework (EF6: maintenance only)` on the Basic Store sample. It adds an interface with the members that the code calls, and an adapter that delegates to the package. It does not remove the package in this step. With no unmaintained package in scope, it refuses and asks for `--dependency <package>`.

T10 never takes the platform: a package of the framework that a project is built on. An MVC project takes its controllers' base class, its routes and its views' helpers from `Microsoft.AspNet.Mvc`, so an adapter cannot stand in for it. That package is the work of T1 or the rebuild. On the Basic Store sample, the T10 section of `options.md` says so:

```output
### T10 Dependency replacement

Why it fits: webapp: EntityFramework (EF6: maintenance only); not for an adapter, the platform itself: T1 or the rebuild: webapp: Microsoft.AspNet.Mvc, the ASP.NET MVC it is built on.
```

CodeLead treats the packages of ASP.NET MVC, ASP.NET Web API, ASP.NET Web Forms and ASP.NET Core this way, in a project built on that framework.

### S: the security findings

S fixes the findings of the dossier's security exposure report (D15): `all`, a class of findings, or ids such as `S-004`. The request names each finding, its place and its fix, with no other change. CodeLead adds an entry to the gate that runs the security scan again and expects the findings gone. So the increment cannot land with a finding left. A finding that changes behaviour on purpose, such as a new refusal, gets its own test.

To decide not to fix a finding, record why: [`/modernize --waive <FINDING> --reason <WHY>`](../chat-commands/modernize.md#--waive). The audit pack lists every waiver.

`all` means all the open findings, and a class name means the open findings of that class. A waived finding is never taken by them. An id names one finding on purpose, and S fixes it even when it was waived. A selector that takes only waived findings composes nothing, and says how to fix one after all. In CodeLead's tests, with four findings that are all waived:

<!-- described, not captured -->
```output
--security all selects only waived findings (S-001, S-002, S-003, S-004; docs/codelead/security-waivers.json): nothing is open to fix. To fix a waived finding after all, name it: --security S-001.
```

## The kinds that a person writes

The options report judges twelve kinds, T1 to T12. CodeLead composes T1, T5, T5D, T10 and S. For any other kind, it says so:

```output
/modernize --transform composes T1 (platform upgrade in place), T5 (module extraction / strangler seam), T5D (the data-layer extraction into a class library) and T10 (dependency replacement) today; T2 is written by hand with /implement, using the options report's row as the brief.
```

Write such a change with [`/implement`](../chat-commands/implement.md). Use the row of the options report as the brief, and keep the pins in the gate.

## The Upgrade in place stage

In the [program](the-program.md), the Upgrade in place stage reads the options report and runs, in this order:

1. T5D for each web project that the T5 row names.
2. T1 on the class libraries on disk, including the one that T5D made. The framework answer, when you gave one, is the target.
3. T10 for each unmaintained package that the T10 row names. It leaves out the platform, and says so once.
4. S on the open security findings that a patch can close, and whose fix changes nothing for a legitimate caller. S always names its classes. A landed S is known by the findings that it covered, so it is not run again.

A finding that no patch can close is named with what closes it, and is not composed. These are the old runtime, a package with a known vulnerability, and a binary without source.

On the Basic Store sample, the stage says this before its first transform:

```output
T10: not on Microsoft.AspNet.Mvc (ASP.NET MVC, webapp): the platform the application is built on is not put behind an adapter (the platform itself: T1 or the rebuild).
S-002 end-of-life-runtime: closed by the platform upgrade (T1) or the rebuild; not a security patch.
```

A transform that landed in an earlier run of the stage is not run again. When a transform is refused or does not land, the stage fails with its words, and the transforms after it wait. When the report indicates no transform, the stage is `skipped`.

### The security findings that wait for a person

The fix of two classes of findings changes who may do what:

| Class | What the fix does |
|---|---|
| `unauthorized-write-action` | It adds an authorization requirement: a caller who may write today without signing in is refused. |
| `permissive-cors` | It names the origins that may call with the user's credentials: a site that calls today from another origin is refused. |

The journeys recorded the application as it is. So the stage does not apply these fixes by itself. After the other transforms, it ends `waiting`. Its problem lists each such finding with two ways on: waive it with a reason, or fix it on purpose. The fixes of the other classes change nothing that a legitimate caller does or sees, such as output encoding or a parameterized query. The stage applies them under the pins.

On the Basic Store sample, nine write actions accept a write without `[Authorize]`, by design. The stage waits with `9 security findings are decisions for a person`. When you have decided each, record it with `codelead modernize answer security=decided`, and run the stage again. Two other answers decide all of them at once: `security=fix-all` fixes them, and `security=fix-none` leaves them open. See [Waive a security finding](../how-to/waive-a-security-finding.md).

> [!IMPORTANT]
> The transforms of a .NET Framework project build with Visual Studio's MSBuild, so they need Windows. See [Build .NET Framework projects on Windows](../how-to/build-on-windows.md).

## Details

- **Options**: `--scope`, `--target`, `--dependency`, `--library` (T5D only), `--security`, `--without-pins` and `--again`. See [`/modernize --transform`](../chat-commands/modernize.md#--transform).
- **Validation**: a transform validates what it changed plus the declared gate, and asks no scope question.
- **The named surface**: the request names its folders and files. A change outside them gets a note: `outside the folder the request names: <file>`.
- **A second run**: over an increment that is applied and not validated, CodeLead asks to validate it first. After a declined patch, it offers to apply it, show it or compose again. `--again` composes anew on purpose.
- **Repair rounds**: `modernization.repair.rounds` in `.codelead/config.json`, 2 by default. 0 runs none.
- **Results**: `Transform T5D landed with the parity gate green` when it lands. `Transform T1 did not land: MSBuild exited 1` when the gate refused it. See [Results and stops](../concepts/results-and-stops.md).

## What a transform is not

- **A transform is not a rewrite**. It changes what its request names, and nothing else.
- **A transform is not run without proof**. The pins run on every landing. Without them, CodeLead refuses, unless you say `--without-pins`.
- **A transform does not commit for you**, unless `CODELEAD_CHECKPOINT_COMMITS=1` is set. Commit after each landing, so that each change shows on its own.

## Related content

- [The eight stages](the-eight-stages.md#4-upgrade-in-place)
- [`/modernize --transform`](../chat-commands/modernize.md#--transform)
- [Pins](pins.md)
- [The dossier](dossier.md): the options report
- [The rebuild](rebuild.md): when to rebuild in place of an upgrade
- [Validation and the gate](../concepts/validation-and-the-gate.md)
- [Build .NET Framework projects on Windows](../how-to/build-on-windows.md)
