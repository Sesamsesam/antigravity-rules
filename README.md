# Antigravity Rules

**Status**: ACTIVE | **Version**: Production v1.0  
**Repository**: `github.com/sesamsesam/antigravity-rules`

A modular, reusable AI agent instruction framework. These rules govern AI behavior across all Antigravity projects.

---

## Quick Start

### Adding Rules to a New Project

```bash
cd /path/to/your-project

# Add this repo as a submodule
git submodule add https://github.com/sesamsesam/antigravity-rules.git docs/rules

# Commit the submodule
git add .gitmodules docs/rules
git commit -m "feat: Add Antigravity rules as submodule"
```

### Create Your Project's AGENTS.md

Create `AGENTS.md` in your project root:

```markdown
# AGENTS.md

## Global Rules
@docs/rules/AGENTS_CORE.md
@docs/rules/AGENTS_BRIDGE.md

## Project Overview
[Your project description here]

## Build & Test
[Your build commands here]
```

That's it! OpenCode will import the global rules via `@` references.

---

## Updating Rules in a Project

When global rules are updated, pull the latest into your project:

```bash
cd /path/to/your-project

# Update submodule to latest commit
git submodule update --remote --merge

# Commit the updated pointer
git add docs/rules
git commit -m "chore: Bump antigravity-rules to latest"
git push
```

---

## Editing Global Rules

When you need to update the global rules themselves:

### Option A: Edit from Local Canonical Folder

```bash
# Go to local rules folder
cd ~/Documents/Antigravity/Rules

# Make your changes
vim AGENTS_CORE.md

# Commit and push
git add .
git commit -m "feat: Update worker lock policy"
git push
```

### Option B: Edit from Within a Project

```bash
cd /path/to/your-project/docs/rules

# Make your changes
vim AGENTS_BRIDGE.md

# Commit and push (this pushes to the rules repo!)
git add .
git commit -m "fix: Clarify retry policy"
git push

# Back in project root, update the submodule pointer
cd ../..
git add docs/rules
git commit -m "chore: Bump rules (retry policy fix)"
git push
```

---

## Cloning a Project with Rules

When cloning a project that uses this submodule:

```bash
# Clone with submodules
git clone --recurse-submodules https://github.com/user/project.git

# Or if already cloned without submodules:
git submodule update --init --recursive
```

---

## File Structure

```
antigravity-rules/
├── README.md                    # This file
├── AGENTS_CORE.md               # Prime Directive, Workflow, Agent Identity
├── AGENTS_BRIDGE.md             # Bridge Protocol execution contract
├── 30_The_Bridge_Protocol.md    # Full technical specification
├── 00_PRIME_DIRECTIVE.md        # Foundational principles
├── 10_The_Workflow.md           # Professional Git workflow
├── 20_The_Agent_Protocol.md     # Agent roles and handoff
└── 99_OpenCode_Config.md        # OpenCode-specific configuration
```

---

## How It Works

```
Global (this repo)              Project (your repo)
┌─────────────────┐            ┌─────────────────┐
│ AGENTS_CORE.md  │◄───────────│ AGENTS.md       │
│ AGENTS_BRIDGE.md│   @import  │ (uses @ refs)   │
│ 30_Bridge_...   │            │                 │
└─────────────────┘            └─────────────────┘
         ▲
    Git Submodule (docs/rules/)
```

1. **Global Rules** live in this repo (single source of truth)
2. **Projects** add this repo as a Git submodule to `docs/rules/`
3. **AGENTS.md** in each project uses `@` syntax to import global rules
4. **Updates** are pulled via `git submodule update --remote`

---

## Core Files

### `AGENTS_CORE.md`
Essential operational rules:
- **Prime Directive**: User first, no ghost policy, documentation
- **Workflow**: Professional Git (never push to main, conventional commits)
- **Agent Identity**: Planner, Coder, OpenCode roles

### `AGENTS_BRIDGE.md`
Bridge Protocol execution contract:
- **Watcher-OpenCode Contract**: Watcher verifies, OpenCode edits
- **Deterministic Order**: Safety check → Lock → Edit → Audit → Verify → Result
- **Critical Guarantees**: Always write result, stop on failure, auto-branch

### `30_The_Bridge_Protocol.md`
Full technical specification:
- Complete 15-step state machine
- Task and Result JSON schemas
- Canonical reason enum
- Redaction patterns and size caps
- Operations runbook

---

## Command Reference

| Action | Command |
|--------|---------|
| Add submodule | `git submodule add https://github.com/sesamsesam/antigravity-rules.git docs/rules` |
| Update submodule | `git submodule update --remote --merge` |
| Clone with submodules | `git clone --recurse-submodules <url>` |
| Init submodules after clone | `git submodule update --init --recursive` |
| Push rule changes | (from docs/rules) `git push` |

---

## No-Regression Policy

**Do not remove or weaken reliability guarantees.**

1. **Idempotency**: Skip if result exists + Stale Lock Recovery
2. **Single Worker**: Worker Lock (FIFO, TTL-based)
3. **Safety**: Dirty Block + Auto-Branch
4. **Audit**: Pre/Post Patches + Redacted Logs + Git Status
5. **Always Write Result**: No silent failures

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v1.0 | 2026-01-15 | Production Edition - Full Bridge Protocol |

---

## License

Internal use. Adapt as needed for your projects.
