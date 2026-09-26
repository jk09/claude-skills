# claude-skills

A reusable Claude Code setup that keeps agent work spec-driven and documented, and prevents feature sprawl. It is made of `CLAUDE.md` rules, skills, hooks and doc templates. Copy it into a repository so every Claude Code session there follows the same workflow:

**spec → implement → ship**, with feature docs kept in sync with the code.

Source: [`jk09/browse-ledger@2e1c30c`](https://github.com/jk09/browse-ledger/tree/2e1c30cd86f62c060e77607d54b83d15fc425661). The files in `template/` are byte-identical to that commit.

## What's inside

```
template/
├── CLAUDE.md                                   # always-loaded rules: workflow, features, where instructions go
├── .claude/
│   ├── settings.json                           # SessionStart + Stop hooks
│   ├── scripts/
│   │   ├── active-spec.ps1                     # SessionStart: loads the Active spec into context
│   │   └── check-done.ps1                      # Stop: blocks "done" while docs/spec lag the code
│   └── skills/
│       ├── spec/SKILL.md                       # write a spec, get approval, set it Active
│       ├── ship/SKILL.md                       # verify, tick acceptance criteria, update docs, commit, PR
│       ├── document-feature/
│       │   ├── SKILL.md                        # feature README / CLAUDE.md / ADR / index upkeep
│       │   └── templates/{feature-readme,adr}.md
│       └── audit-features/SKILL.md             # monthly sprawl audit: undocumented, overlapping, overdue flags
└── docs/
    ├── features.md                             # generated feature index (one row per feature folder)
    └── specs/feature-spec-template.md          # spec template (Status: Draft → Active → Done)
```

## How it keeps the workflow on track

| Mechanism | Guards against |
|---|---|
| `spec` skill + spec template, one `Active` spec at a time | Starting non-trivial work with no agreed scope or testable acceptance criteria |
| SessionStart hook (`active-spec.ps1`) | New sessions losing track of the current spec |
| `ship` skill | Ad hoc commits, unticked acceptance criteria, inconsistent commit messages |
| Stop hook (`check-done.ps1`) | Finishing while feature code changed without its README, a removed feature is still listed in `docs/features.md`, or the active spec wasn't updated |
| `document-feature` skill | Doc drift. READMEs are capped at ~30 lines, so an oversized feature shows up as sprawl |
| One folder per feature, `docs/features.md` index, flag owner and removal date | Features and flags nobody can find, own or remove |
| `audit-features` skill | Sprawl that builds up over time: overlaps, new cross-feature dependencies, overdue flags, stale `CLAUDE.md` lines |
| ADRs in `docs/adr/`, append-only | Decisions changing without a record |

## Install into a project

Requirements: [PowerShell 7](https://learn.microsoft.com/powershell/scripting/install/installing-powershell) (`pwsh`) on `PATH` for the hooks, and `git`.

```sh
git clone https://github.com/jk09/claude-skills
cp -rn claude-skills/template/. path/to/your-repo/   # -n: don't overwrite existing files
```

If the target already has a `CLAUDE.md` or `.claude/settings.json`, merge them by hand. `cp -n` leaves existing files in place.

## Adapt to your project

The template was written for a project whose features live in `src/Api/Features/<Feature>/`. If yours use a different layout, update these places:

| What | Where |
|---|---|
| Feature root `src/Api/Features` | `CLAUDE.md`, `.claude/scripts/check-done.ps1` (`$featuresRoot`), `.claude/skills/document-feature/SKILL.md`, `.claude/skills/audit-features/SKILL.md`, `docs/features.md` |
| Default branch `main` | `.claude/skills/document-feature/SKILL.md` (Step 1 `git diff main...HEAD`) |
| Test file pattern | `.claude/scripts/check-done.ps1` (`$testPattern`) |
| .NET-specific wording (DI registrations, `appsettings`) | `CLAUDE.md`, `ship` and `document-feature` skills, `templates/feature-readme.md` |
| Build and test commands | `.claude/skills/ship/SKILL.md` step 1, if you want them spelled out |

## Run the doc check in CI

`check-done.ps1` can also check a PR's diff instead of the working tree:

```sh
pwsh -NoProfile -File .claude/scripts/check-done.ps1 -Base origin/main
```

It exits with code 2 and lists the problems when docs or the active spec lag the code.
