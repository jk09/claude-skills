# Project rules

Persistent context for Claude. Agentic code generation follows the rules below. Rules are imperative: "Do" and "Never" are mandatory.

## Commands

Fill in for the project; run them before finishing any task.

- Build: `<command>`
- Test: `<command>`
- Lint / static analysis: `<command>`

## Architecture

- A module is a unit of functionality or business value (package, class, function, library or similar, depending on the environment).
- Modularize by concern, not by technical layer. Give each module one concern, or a few closely related ones.
- Keep changes local: a change should touch few modules. Break a feature request down to achieve this. If changes repeatedly cannot be mapped to modules, refactor the decomposition.
- Prefer deep modules: significant functionality behind a simple API. Keep cohesion high and coupling low.
- At module boundaries that cross a package or process, modules interact only by messages (a function call, a method call or a queued message). Message names and payloads are independent of module internals. The payload uses a separate public model, translated into the module's internal model. Do not add this indirection between closely related code inside one module.
- A feature (see Features) maps to one module or a small set of modules. Record which in the feature's README.

## Design by Contract

- Define preconditions, postconditions and invariants in each module's public API. Where the environment has no DbC support, record them in comments.
- Contracts may be given before the run or inferred during generation. Never override a defined contract without the user's approval.

## Coding standards

- Run the lint / static analysis command and fix all warnings, iterating until clean.
- Prefer functional code with immutable data when practical and not harmful to performance. Prefer pure functions; make operations with side effects (handlers, migrations, retries) idempotent.
- Make state modification explicit, e.g. in a class instance.

## Testing

- Every behavior change comes with tests; bug fixes start with a failing test.
- Test the public API and contracts of a module, not its internals.
- Never finish a task with failing tests or lint errors. Report them if they cannot be fixed.

## Boundaries

Ask the user before:

- changing a public API or contract, a database schema or a dependency;
- deleting files or data outside the current task;
- touching secrets, credentials or deployment configuration. Never commit secrets.

## Features

- Features root and test-file pattern: `featuresRoot` and `testPattern` in [.claude/project.json](./.claude/project.json). Set them once for the project; the hook script and skills read them from there.
- A feature is non-trivial if it adds user-visible behavior or touches more than one module. Record each non-trivial feature in a feature spec; small or ad-hoc changes need none.
- One folder per feature under `<features root>/<Feature>/` with its code, a short `README.md` and, if needed, a `CLAUDE.md` for feature-specific rules. Index: [docs/features.md](./docs/features.md).
- Docs are written by the [`document-feature`](.claude/skills/document-feature/) skill; the Stop hook blocks finishing while feature code or the active spec changed without its docs.
- Removing a feature means deleting its folder, flags, tests and `docs/features.md` row in the same PR.
- Decisions go in `docs/adr/`, append-only: a changed decision gets a new ADR that supersedes the old one.
- Every feature flag has an owner and a removal date in its feature README. The [`audit-features`](.claude/skills/audit-features/) skill lists overdue flags and sprawl.

## Workflow

- Non-trivial work (see Features) starts from a spec in [docs/specs/](./docs/specs/). No spec → use the [`spec`](.claude/skills/spec/) skill first.
- Finish every task with the [`ship`](.claude/skills/ship/) skill. Don't commit ad hoc.
- Commits: [Conventional Commits](https://www.conventionalcommits.org/), scope = feature folder name, `Spec: <spec-id>` line when a spec drove the change. Format in the `ship` skill.

## Where instructions go

- Short rules that always apply → this file, a few lines. Feature-specific rules → that feature's `CLAUDE.md`.
- Step-by-step procedures → a skill in `.claude/skills/`.
- Steps that must run at a set moment → hooks in `.claude/settings.json` (scripts in `.claude/scripts/`; the template ships PowerShell 7 scripts and needs `pwsh`, replace them for other shells).
- Decisions and their rationale → `docs/adr/`. Specs → `docs/specs/`.
