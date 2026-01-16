# 50 RULES INSTALLATION

**Status**: ACTIVE | **Scope**: GLOBAL

This document explains how to install and configure Antigravity rules in any project.

---

## Overview: Where Rules Live

| Location | Scope | Auto-Loaded? | Purpose |
|----------|-------|--------------|---------|
| `~/.gemini/GEMINI.md` | Global (all projects) | ✅ Yes | Your personal coding identity |
| `.agent/rules/*.md` | Workspace | ✅ Yes | Project-specific rules |
| `.agent/workflows/*.md` | Workspace | On `/command` | Triggered actions |
| `.agent/skills/*/SKILL.md` | Workspace | When relevant | On-demand capabilities |
| `GEMINI.md` (project root) | Workspace | ✅ Yes | System-level project config |
| `docs/rules/` (submodule) | Workspace | Via `@` reference | Shared rules across projects |

---

## 1. Set Up Global Identity

Create your personal identity file that applies to ALL projects:

```bash
# Create/edit your global rules
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

---

## 2. Add Shared Rules to a Project

Add the antigravity-rules repo as a git submodule:

```bash
cd /path/to/your-project

# Add submodule
git submodule add https://github.com/Sesamsesam/antigravity-rules.git docs/rules

# Commit
git add .gitmodules docs/rules
git commit -m "feat: Add Antigravity rules as submodule"
```

### Updating Rules Later

```bash
# Pull latest rules
git submodule update --remote --merge

# Commit the update
git add docs/rules
git commit -m "chore: Bump antigravity-rules to latest"
```

### Cloning a Project with Rules

```bash
# Clone with submodules included
git clone --recurse-submodules https://github.com/user/project.git

# Or if already cloned:
git submodule update --init --recursive
```

---

## 3. Create Project Root GEMINI.md

Create a `GEMINI.md` in your project root that imports the shared rules:

```markdown
# Project Name

> Brief project description.

## Global Rules

@docs/rules/AGENTS_CORE.md
@docs/rules/AGENTS_BRIDGE.md

## Project Overview

[What this project does]

## Build & Test

\`\`\`bash
[Build commands]
\`\`\`

## Key Files

- `src/` - Source code
- `docs/` - Documentation
```

---

## 4. Create Project-Specific Rules

Create the `.agent/` folder structure:

```bash
mkdir -p .agent/rules .agent/workflows .agent/skills
```

### Rules (Auto-Loaded)

Create `.agent/rules/project_rules.md`:

```markdown
# Project-Specific Rules

## Coding Standards
- [Your project-specific rules]

## Testing Requirements
- [What tests must pass]
```

### Workflows (Triggered by /command)

Create `.agent/workflows/test.md`:

```yaml
---
description: Run tests
---

# /test Workflow

\`\`\`bash
npm test
\`\`\`
```

---

## 5. Create Skills (Optional)

Skills are agent-triggered capabilities. Create in `.agent/skills/<skill_name>/SKILL.md`:

```yaml
---
name: Skill Name
description: Brief description of when this skill should be used
---

# Skill Name

## When to Use
[Describe scenarios]

## How to Execute
[Step-by-step instructions]

## Example
[Code or command example]
```

---

## Complete Example Structure

```
your-project/
├── .agent/
│   ├── rules/
│   │   └── project_rules.md     # Project-specific rules
│   ├── workflows/
│   │   ├── test.md              # /test command
│   │   └── build.md             # /build command
│   └── skills/
│       └── deploy/
│           └── SKILL.md         # Deployment skill
├── docs/
│   └── rules/                   # ← Git submodule (antigravity-rules)
│       ├── AGENTS_CORE.md
│       ├── AGENTS_BRIDGE.md
│       └── ...
├── GEMINI.md                    # Project root config
└── ...
```

---

## Troubleshooting

### Rules Not Being Applied

1. Check that `~/.gemini/GEMINI.md` exists and has content
2. Verify `.agent/rules/` files have `.md` extension
3. Ensure `GEMINI.md` in project root uses correct `@` paths

### Submodule Not Cloning

```bash
git submodule update --init --recursive
```

### Editing Global Rules

```bash
# Edit in your canonical copy
cd ~/Documents/Antigravity/Rules
vim AGENTS_CORE.md

# Commit and push
git add .
git commit -m "docs: Update core rules"
git push

# Then update in each project
cd /path/to/project
git submodule update --remote
git commit -am "chore: Bump rules"
```

---

## References

- [AGENTS_CORE.md](./AGENTS_CORE.md) - Core protocol
- [AGENTS_BRIDGE.md](./AGENTS_BRIDGE.md) - Bridge execution contract
- [30_The_Bridge_Protocol.md](./30_The_Bridge_Protocol.md) - Full specification
