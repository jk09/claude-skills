# CLAUDE 

A persistent context for Claude. The agentic code generation should follow the standards and instructions below.

## Architecture - Modularity

- A module is an unit of functionality/business value. Depending on the programming environment it may be a package, a class, a function, a library, or similar.
- Modularize by concern, not technical layer.
- Each module should implement one or few concerns.
- Every change should affect only a small number of modules. Refine and break down a feature request to achieve this. Refactor module decomposition when feature changes repeatedly cannot be neatly mapped to modules.
- Use deep modules: significant functionality behind a simple API.
- Aim at high cohesion of modules, and low coupling between between modules.
- Modules interact by sending messages to other modules. The exact implementation of "sending messages" depends on the programming environment. It may be a method or function call, or a posting of message into a message queue. The name and payload of the message should be independent of the module's internals. The message payload has a separate public model (a collection of types), to be translated into a module's internal model.


## Design - Design by Contract

- Each module's public API should define preconditions, postconditions, and invariants (in the sense of Design by Contract, or DbC). If the programming environment does not have a support for DbC, record the contracts in the comments. 
- The contracts may be specified before the agentic run, or inferred during code generation. The defined contracts should not be overridden without asking the user for approval.

## Design - Coding Standards

- Use `lint`-like  tools appropriate for each development environment to deterministically find code smells, compiler warnings and the like, and iteratively fix them.

## Design - Functional Code

- Prefer functional code with immutable data structures, when it is practical and does not negatively impact performance.
- Functions should be idempotent when practical. 
- State modification (e.g. in a class instance) should be explicit.

## Features

- Each features other than a small or an ad-hoc one should be recorded in its **feature spec**. 
- One folder per feature under `src/Api/Features/<Feature>/` with its code, a short `README.md` and, if needed, a `CLAUDE.md` for feature-specific rules. Index: [docs/features.md](./docs/features.md).
- Docs are written by the [`document-feature`](.claude/skills/document-feature/) skill; the Stop hook blocks finishing while feature code or the active spec changed without its docs.
- Removing a feature means deleting its folder, flags, tests and `docs/features.md` row in the same PR.
- Decisions go in `docs/adr/`, append-only: a changed decision gets a new ADR that supersedes the old one.
- Every feature flag has an owner and a removal date in its feature README. The [`audit-features`](.claude/skills/audit-features/) skill lists overdue flags and sprawl.



## Workflow

- Non-trivial work starts from a spec in the [spec folder](./docs/specs/). No spec → use the [`spec`](.claude/skills/spec/) skill first.
- Finish every task with the [`ship`](.claude/skills/ship/) skill. Don't commit ad hoc.
- Commits: [Conventional Commits](https://www.conventionalcommits.org/), scope = feature folder name, `Spec: <spec-id>` line when a spec drove the change. Format in the `ship` skill.


## Where instructions go

- Short rules that always apply → this file, a few lines. Feature-specific rules → that feature's `CLAUDE.md`.
- Step-by-step procedures → a skill in `.claude/skills/`.
- Steps that must run at a set moment → hooks in `.claude/settings.json` (scripts in `.claude/scripts/`, PowerShell 7).
