# 50 RULES INSTALLATION

**Status**: ACTIVE | **Scope**: GLOBAL

Complete guide to installing Antigravity rules in any project. Follow this once and future setups take minutes.

---

## Overview: Folder Conventions

| Location | Auto-Loaded? | Purpose |
|----------|--------------|---------|
| `~/.gemini/GEMINI.md` | ✅ Yes | Global identity (all projects) |
| `.agent/rules/*.md` | ✅ Yes | Runtime rules (auto-injected) |
| `.agent/workflows/*.md` | On `/command` | Triggered actions |
| `.agent/skills/*/SKILL.md` | When relevant | Agent capabilities |
| `GEMINI.md` (project root) | ✅ Yes | Project config |
| `AGENTS.md` (project root) | ✅ Yes | OpenCode config |
| `docs/rules/` | No (source) | Submodule (shared rules) |

---

## Quick Setup (5 Minutes)

### Step 1: Clone with Submodules

```bash
git clone --recurse-submodules <repo-url>
cd <project>
```

### Step 2: Create Symlink (Option C)

This makes all rules in `docs/rules/` auto-load via `.agent/rules/`:

```bash
# Remove existing .agent/rules/ folder
rm -rf .agent/rules

# Create symlink to submodule
ln -s ../docs/rules .agent/rules

# Verify symlink works
ls -la .agent/rules/  # Should show -> ../docs/rules
ls .agent/rules/*.md  # Should list all rule files
```

### Step 3: Verify Setup

```bash
# Check all components exist
ls ~/.gemini/GEMINI.md        # Global identity
ls GEMINI.md                  # Project config
ls AGENTS.md                  # OpenCode config (optional)
ls .agent/rules/              # Symlink to docs/rules/
ls .agent/workflows/          # Workflow definitions
ls .agent/skills/             # Skill definitions
```

---

## Full Setup (New Project)

### 1. Set Up Global Identity

```bash
cat > ~/.gemini/GEMINI.md << 'EOF'
# Antigravity Global Identity

## User First
- Amplify the User's capability, not replace their judgment
- If a decision has >10% risk of being wrong → **ASK**

## No-Ghost Policy
- If a step fails, **stop and report immediately**
- Never hallucinate success or hide errors

## Communication Style
- **In Chat**: Be brief (max 3 lines unless explaining)
- **In Artifacts**: Be exhaustive and thorough

## Professional Git
- Never push directly to `main`
- Use Conventional Commits: `feat:`, `fix:`, `docs:`, `chore:`
EOF
```

### 2. Add Rules Submodule

```bash
cd /path/to/your-project

# Add submodule
git submodule add https://github.com/Sesamsesam/antigravity-rules.git docs/rules

# Commit
git add .gitmodules docs/rules
git commit -m "feat: Add Antigravity rules as submodule"
```

### 3. Create Symlink (Option C)

```bash
# Create .agent structure
mkdir -p .agent/workflows .agent/skills

# Symlink rules to submodule (auto-load all rules)
ln -s ../docs/rules .agent/rules

# Verify
ls .agent/rules/*.md
```

### 4. Create Project GEMINI.md

```bash
cat > GEMINI.md << 'EOF'
# Project Name

> Brief project description.

## Security Specifications

@docs/rules/31_Secure_Watcher.md
@docs/rules/34_Diamond_Rules.md

## Build & Test

```bash
# Your build commands here
```
EOF
```

### 5. Create AGENTS.md (for OpenCode)

```bash
cat > AGENTS.md << 'EOF'
# AGENTS.md - Project Name

## Your Role: The Builder

You are the **Builder** in the Bridge Protocol.

### Your Job ✅
- Edit files as instructed
- Use safe wrapper patterns

### NOT Your Job ❌
- Running tests (Watcher does this)
- Managing git state

## Rules
@docs/rules/AGENTS_CORE.md
@docs/rules/AGENTS_BRIDGE.md
EOF
```

### 6. Add Workflows

```bash
# Create test workflow
cat > .agent/workflows/test.md << 'EOF'
---
description: Run tests
---
# /test Workflow
```bash
npm test
```
EOF
```

---

## Updating Rules

```bash
# Pull latest from rules repo
git submodule update --remote --merge

# Commit the update
git add docs/rules
git commit -m "chore: Bump antigravity-rules to latest"
```

---

## File Structure After Setup

```
your-project/
├── GEMINI.md                    # Project config (Antigravity)
├── AGENTS.md                    # Project config (OpenCode)
├── .agent/
│   ├── rules/ → ../docs/rules  # SYMLINK to submodule
│   ├── workflows/
│   │   └── test.md
│   └── skills/
│       └── (your skills)
├── docs/
│   └── rules/                   # Git submodule (source files)
│       ├── AGENTS_CORE.md       # Core identity
│       ├── AGENTS_BRIDGE.md     # Bridge contract
│       ├── 31_Secure_Watcher.md # Security spec
│       ├── 32_Docker_Sandbox.md # Docker policy
│       ├── 34_Diamond_Rules.md  # Diamond rules
│       └── ...
└── src/
```

---

## Troubleshooting

### Symlink Not Working

```bash
# Check if symlink exists
ls -la .agent/rules/

# If broken, recreate
rm -rf .agent/rules
ln -s ../docs/rules .agent/rules
```

### Submodule Empty

```bash
git submodule update --init --recursive
```

### Rules Not Auto-Loading

1. Verify `.agent/rules/` is a symlink: `ls -la .agent/`
2. Check symlink target exists: `ls docs/rules/`
3. Verify files are `.md` extension

---

## References

- [AGENTS_CORE.md](./AGENTS_CORE.md) - Core protocol
- [AGENTS_BRIDGE.md](./AGENTS_BRIDGE.md) - Bridge contract
- [40_Tools_Setup.md](./40_Tools_Setup.md) - Tool installation
- [archive/30_The_Bridge_Protocol.md](./archive/30_The_Bridge_Protocol.md) - Full spec
