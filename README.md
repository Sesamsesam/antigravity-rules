# Antigravity Rules

**Status**: ACTIVE | **Version**: Production v2.1  
**Repository**: `github.com/sesamsesam/antigravity-rules`

A modular, reusable AI agent instruction framework. These rules govern AI behavior across all Antigravity projects.

---

## 🚨 START HERE

**New session on an existing project?** Rules auto-load. Just start working.

**New project setup?** Tell the agent:
```
Read and execute /Users/yeti/Documents/Antigravity/Rules/00_BOOTSTRAP.md
```

**Need to understand the full system?** Read this entire README first.

---

## ⚠️ Common Mistakes — AVOID THESE

**The #1 Rule: All edits happen at the canonical location FIRST**

| ❌ Wrong | ✅ Right |
|----------|----------|
| Editing any file in `project/docs/rules/` | Edit in `/Users/yeti/Documents/Antigravity/Rules/` |
| Editing any file in `project/.agent/rules/` | Same — it's a symlink to docs/rules |
| Editing `~/.gemini/GEMINI.md` directly | Edit the source template in canonical, re-run bootstrap |
| Creating a new rule file in a project | Create in canonical, update this README's index, push, then submodule update |

**Why?** 
- `project/docs/rules/` = readonly submodule (reference, not source)
- `project/.agent/rules/` = symlink (not even a real folder)
- `~/.gemini/GEMINI.md` = applied output (template lives in canonical)

**The only writable location is:** `/Users/yeti/Documents/Antigravity/Rules/`

---

## 📖 Index by Category

### Category 1: Core Identity & Behavior
> **When to read:** Any request about agent behavior, principles, git workflow, file creation, or modifying global rules.

| File | Purpose |
|------|---------|
| `AGENTS_CORE.md` | **Prime directive**, git workflow, file creation conventions, **global rules modification workflow** |
| `34_Diamond_Rules.md` | The 10 non-negotiable rules to never forget |

⚠️ **`AGENTS_CORE.md` contains the canonical workflow for adding/modifying rules (Section 5: "Global Rules Workflow").** If you're asked to create or update a rule, you MUST read this first.

---

### Category 2: Bridge Protocol (Automation Infrastructure)
> **When to read:** Any request about the Watcher, OpenCode integration, automated task execution, or security.

| File | Purpose |
|------|---------|
| `AGENTS_BRIDGE.md` | Execution contract between Watcher and OpenCode |
| `31_Secure_Watcher.md` | Security specification, safe wrappers, secrets handling |
| `32_Docker_Sandbox_Policy.md` | Docker container security flags and mount rules |
| `33_Repo_Separation.md` | Why Bridge lives in its own repo, deployment boundaries |

Read ALL files in this category together — they form a complete security model.

---

### Category 3: Setup & Installation
> **When to read:** Setting up a new project, installing tools, or configuring the rules system.

| File | Purpose |
|------|---------|
| `00_BOOTSTRAP.md` | **Imperative** - Automatic setup script (EXECUTE, don't just read) |
| `40_Tools_Setup.md` | Install OpenCode, Beads, auth plugins, tool relationships |
| `50_Rules_Installation.md` | Complete guide to installing rules via submodule + symlink |
| `99_OpenCode_Config.md` | OpenCode-specific entry point and imports |

**Reading order for setup:** `00_BOOTSTRAP.md` → `50_Rules_Installation.md` → `40_Tools_Setup.md`

---

## 🎯 Action-Based Navigation

Use this table to find what to read for specific actions:

| If you want to... | Read these files |
|-------------------|------------------|
| **Add/modify a global rule** | `AGENTS_CORE.md` (Section 5: Global Rules Workflow) |
| **Set up a new project** | `00_BOOTSTRAP.md` (execute it) |
| **Understand the rule system** | `50_Rules_Installation.md` → `README.md` |
| **Know the core principles** | `AGENTS_CORE.md` → `34_Diamond_Rules.md` |
| **Work with Bridge/Watcher** | All Category 2 files |
| **Install OpenCode/Beads/tools** | `40_Tools_Setup.md` |
| **Configure OpenCode specifically** | `99_OpenCode_Config.md` |
| **Understand Docker security** | `31_Secure_Watcher.md` → `32_Docker_Sandbox_Policy.md` |

---

## 📂 File Structure

```
antigravity-rules/
│
├── # CORE IDENTITY (Category 1)
├── AGENTS_CORE.md               # Prime directive, git, file rules, RULE MODIFICATION WORKFLOW
├── 34_Diamond_Rules.md          # 10 cardinal rules
│
├── # BRIDGE PROTOCOL (Category 2)
├── AGENTS_BRIDGE.md             # Watcher-OpenCode execution contract
├── 31_Secure_Watcher.md         # Security specification
├── 32_Docker_Sandbox_Policy.md  # Docker security config
├── 33_Repo_Separation.md        # Multi-repo strategy
│
├── # SETUP & INSTALLATION (Category 3)
├── 00_BOOTSTRAP.md              # Automatic setup (IMPERATIVE)
├── 40_Tools_Setup.md            # Install OpenCode, Beads, etc.
├── 50_Rules_Installation.md     # Complete installation guide
├── 99_OpenCode_Config.md        # OpenCode entry point
│
├── README.md                    # THIS FILE - the index
│
└── archive/                     # Archived detailed specs
    └── 30_The_Bridge_Protocol.md  # Full 15-step state machine
```

---

## ⚡ Quick Reference: The Canonical Workflow

### Where Rules Live

| Location | Type | Editable? |
|----------|------|-----------|
| `/Users/yeti/Documents/Antigravity/Rules/` | **Canonical source** | ✅ YES - edit here |
| `project/docs/rules/` | Git submodule | ❌ NO - it's a reference |
| `project/.agent/rules/` | Symlink to docs/rules | ❌ NO - it's a symlink |
| `~/.gemini/GEMINI.md` | Global identity | ⚠️ Template lives in `00_BOOTSTRAP.md` |

### Rule Modification Flow

```
1. EDIT in canonical: /Users/yeti/Documents/Antigravity/Rules/
2. COMMIT & PUSH to GitHub
3. Projects pull via: git submodule update --remote
```

**Never edit `docs/rules/` or `.agent/rules/` directly in a project.**

---

## 🔄 Updating Rules

In any project that uses these rules:

```bash
git submodule update --remote --merge
git add docs/rules
git commit -m "chore: Bump antigravity-rules"
```

---

## 📝 Adding New Rules

When creating a new rule file:

1. **Create in canonical location:** `/Users/yeti/Documents/Antigravity/Rules/`
2. **Use numbered prefix for ordering:** `XX_Name.md` (e.g., `35_New_Rule.md`)
3. **Update this README:** Add to appropriate category and action table
4. **Commit & push** to the rules repo
5. **Projects pull** via submodule update

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v2.1 | 2026-01-23 | Added comprehensive index, action-based navigation, categories |
| v2.0 | 2026-01-16 | Symlink strategy, archived redundant files |
| v1.0 | 2026-01-15 | Production Edition - Full Bridge Protocol |
