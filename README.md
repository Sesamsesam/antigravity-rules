# Antigravity Rules

**Status**: ACTIVE | **Version**: Production v2.0  
**Repository**: `github.com/sesamsesam/antigravity-rules`

A modular, reusable AI agent instruction framework. These rules govern AI behavior across all Antigravity projects.

---

## Quick Start

See **[50_Rules_Installation.md](./50_Rules_Installation.md)** for complete setup instructions.

### 5-Minute Setup

```bash
# 1. Add submodule
git submodule add https://github.com/Sesamsesam/antigravity-rules.git docs/rules

# 2. Create symlink for auto-loading
mkdir -p .agent/workflows .agent/skills
ln -s ../docs/rules .agent/rules

# 3. Verify
ls .agent/rules/*.md  # Should list all rules
```

---

## File Structure

```
antigravity-rules/
│
├── # RUNTIME RULES (auto-injected via .agent/rules/)
├── AGENTS_CORE.md               # Core identity, git workflow
├── AGENTS_BRIDGE.md             # Bridge execution contract
├── 31_Secure_Watcher.md         # Security specification
├── 32_Docker_Sandbox_Policy.md  # Docker security config
├── 34_Diamond_Rules.md          # 10 cardinal rules
│
├── # SETUP GUIDES (reference only)
├── 33_Repo_Separation.md        # Multi-repo strategy
├── 40_Tools_Setup.md            # Install OpenCode, Beads, etc.
├── 50_Rules_Installation.md     # Complete installation guide
├── 99_OpenCode_Config.md        # OpenCode entry point
├── README.md                    # This file
│
└── archive/                     # Archived detailed specs
    ├── 00_PRIME_DIRECTIVE.md
    ├── 10_The_Workflow.md
    ├── 20_The_Agent_Protocol.md
    └── 30_The_Bridge_Protocol.md
```

---

## Core Files

### Runtime Rules (Auto-Loaded)

| File | Purpose |
|------|---------|
| `AGENTS_CORE.md` | Prime directive, git workflow, agent identity |
| `AGENTS_BRIDGE.md` | Watcher-OpenCode execution contract |
| `31_Secure_Watcher.md` | Security invariants, safe wrappers |
| `32_Docker_Sandbox_Policy.md` | Docker flags and mount rules |
| `34_Diamond_Rules.md` | 10 rules to never forget |

### Setup Guides

| File | Purpose |
|------|---------|
| `50_Rules_Installation.md` | **Start here** - Complete setup guide |
| `40_Tools_Setup.md` | Install OpenCode, Beads, auth plugins |
| `33_Repo_Separation.md` | Why Bridge is separate from app |
| `99_OpenCode_Config.md` | OpenCode-specific imports |

---

## Updating Rules

```bash
# In your project
git submodule update --remote --merge
git add docs/rules
git commit -m "chore: Bump rules"
```

---

## Version History

| Version | Date | Changes |
|---------|------|---------|
| v2.0 | 2026-01-16 | Symlink strategy, archived redundant files |
| v1.0 | 2026-01-15 | Production Edition - Full Bridge Protocol |
