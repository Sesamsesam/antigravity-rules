# 40 TOOLS & SETUP

**Status**: ACTIVE | **Scope**: GLOBAL

This document describes the tools required for the Antigravity ecosystem and how they work together with the Bridge Protocol.

---

## Overview: The Agent Stack

```
User
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  OpenCode (IDE/Agent)                                   │
│  + opencode-antigravity-auth plugin (OAuth to models)   │
│  + Beads CLI (bd) for task tracking                     │
└─────────────────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  Bridge Protocol (Watcher)                              │
│  • Orchestrates tasks via .ai-handoff/                  │
│  • Runs verification (bun test)                         │
│  • Manages Beads lifecycle (bd new → bd done)           │
└─────────────────────────────────────────────────────────┘
  │
  ▼
┌─────────────────────────────────────────────────────────┐
│  Antigravity API (Google's Model Gateway)              │
│  • Claude (sonnet-4-5, opus-4-5-thinking)              │
│  • Gemini (gemini-3-pro, gemini-3-flash)               │
└─────────────────────────────────────────────────────────┘
```

---

## 1. OpenCode

**What It Is:**  
A local AI coding agent/IDE that connects to various LLM providers.

**Download:**  
- https://opencode.ai

**Key Features:**
- Runs locally on your machine
- Reads `AGENTS.md` for project-specific instructions
- Supports plugins for authentication and extensions
- Provides headless server mode (`opencode serve`) for automation

**Our Use:**  
OpenCode is the **"Builder"** in the Bridge Protocol. It edits files based on prompts but does NOT run tests (the Watcher does that).

### OpenCode Configuration Files

OpenCode reads instructions from these locations (in order):

| Location | Scope | Purpose |
|----------|-------|---------|
| `~/.config/opencode/AGENTS.md` | Global | Core rules for ALL projects |
| `project/AGENTS.md` | Project | Project-specific instructions |
| `.opencode/agent/*.md` | Project | Specialized agents (optional) |
| `~/.config/opencode/opencode.json` | Global | OpenCode settings + plugins |

### Global AGENTS.md Setup

Create `~/.config/opencode/AGENTS.md`:

```markdown
# AGENTS.md (Global OpenCode)

## Your Role
You are the **Builder** in the Antigravity ecosystem.

## Core Rules
- User First: If >10% risk of being wrong → ASK
- No-Ghost Policy: If a step fails, STOP and report
- Professional Git: Never push to main, use conventional commits

## File Creation Rules
1. Search first: Does file exist? → UPDATE it
2. Related file? → ADD section to it
3. Truly new? → Create in correct location
```

### Project AGENTS.md

Each project should have an `AGENTS.md` in the root with:

```markdown
# AGENTS.md - [Project Name]

## Rules
@docs/rules/AGENTS_CORE.md
@docs/rules/AGENTS_BRIDGE.md

## Build & Test
[Project-specific commands]
```

### Specialized Agents (Optional)

For task-specific agents, create files in `.opencode/agent/`:

```
.opencode/
└── agent/
    ├── refactor.md      # Specialized for refactoring
    ├── test.md          # Specialized for writing tests
    └── document.md      # Specialized for documentation
```

**Example: `.opencode/agent/refactor.md`**

```markdown
---
name: Refactor
description: Specialized agent for code refactoring tasks
---

# Refactor Agent

## Specialization
You are specialized in refactoring code while maintaining behavior.

## Rules
1. Never change functionality, only improve structure
2. Run tests before and after confirming behavior unchanged
3. Make atomic commits for each refactor step
```

### OpenCode vs Antigravity: Key Differences

| Aspect | Antigravity | OpenCode |
|--------|-------------|----------|
| Config file | `GEMINI.md` | `AGENTS.md` |
| Auto-load folder | `.agent/rules/` ✅ | ❌ Not supported |
| Global config | `~/.gemini/GEMINI.md` | `~/.config/opencode/AGENTS.md` |
| Specialized agents | Not applicable | `.opencode/agent/*.md` |
| Rule imports | `@path/to/file.md` | `@path/to/file.md` (same syntax) |

### Tips for OpenCode

1. **Use `/init`** to generate an initial AGENTS.md
2. **Commit AGENTS.md** to version control
3. **Check project AGENTS.md** first, then global
4. **Specialized agents** are optional but help for recurring tasks

---


## 2. opencode-antigravity-auth

**What It Is:**  
An OpenCode plugin that authenticates against **Antigravity** (Google's IDE) via OAuth, giving you access to premium models like `gemini-3-pro` and `claude-opus-4-5-thinking`.

**Repository:**  
- https://github.com/NoeFabris/opencode-antigravity-auth

**Installation:**
```bash
# Add plugin to OpenCode config
# ~/.config/opencode/opencode.json:
{
  "plugin": ["opencode-antigravity-auth@beta"]
}

# Authenticate with Google
opencode auth login
```

**How It Works:**
1. Plugin intercepts requests to Google's generativelanguage API
2. Uses your Google OAuth credentials for authentication
3. Routes requests through Antigravity's quota system
4. Supports multi-account rotation for higher combined quotas

**Do You Need to Clone This Repo?**  
**NO.** The plugin is installed via OpenCode's plugin system. You don't need the source code in your project. The folder at `~/opencode-antigravity-auth/` is only useful if you're developing/debugging the plugin itself.

**What It Provides:**
- Access to Antigravity quota (Claude, Gemini 3 models)
- Access to Gemini CLI quota (Gemini 2.5 models)
- Dual quota pools per account (effectively doubles Gemini quota)
- Automatic token refresh
- Rate limit handling with account rotation

---

## 3. Beads (bd)

**What It Is:**  
A distributed, git-backed graph issue tracker designed for AI agents. Think of it as "memory" for coding agents that persists across sessions.

**Repository:**  
- https://github.com/steveyegge/beads

**Installation:**
```bash
# Option 1: Install script (macOS/Linux)
curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash

# Option 2: npm
npm install -g @beads/bd

# Option 3: Homebrew (macOS)
brew install steveyegge/beads/bd

# Option 4: Go
go install github.com/steveyegge/beads/cmd/bd@latest
```

**Initialize in a Project:**
```bash
cd /path/to/project
bd init
```

**How It Works:**
- Issues stored as JSONL in `.beads/` directory
- Git-backed: versioned, branched, merged like code
- Hash-based IDs (e.g., `bd-a1b2`) prevent merge collisions
- Supports hierarchy: Epic → Task → Sub-task

**Essential Commands:**
| Command | Description |
|---------|-------------|
| `bd init` | Initialize Beads in a project |
| `bd ready` | List ready-to-work tasks |
| `bd create "Title"` | Create a new task |
| `bd show <id>` | Show task details |
| `bd done <id>` | Mark task as done |
| `bd block <id> "reason"` | Block a task |

---

## 4. How These Tools Work Together

### The Bridge Protocol Workflow

```
1. Antigravity (Planner)
   └─ Creates task in .ai-handoff/tasks/{id}.json

2. Watcher (Bridge)
   ├─ Reads task
   ├─ Creates Bead: `bd new "{bead_title}"`
   ├─ Invokes OpenCode to edit files
   ├─ Runs verification: `bun test`
   ├─ On Success: `bd done`
   ├─ On Fail: `bd block`
   └─ Writes result to .ai-handoff/results/

3. OpenCode (Builder)
   └─ Edits files only (authenticated via antigravity-auth plugin)
```

### Who Does What?

| Component | Role | Runs Tests? | Manages Beads? |
|-----------|------|-------------|----------------|
| **Antigravity** | Planner/Judge | No | No (Watcher does) |
| **Watcher** | Orchestrator | **Yes** | **Yes** |
| **OpenCode** | Builder | No | No |
| **Beads** | Memory/Tracking | N/A | N/A |

### Important Relationships

1. **OpenCode + antigravity-auth:**
   - Plugin handles authentication
   - OpenCode sends prompts, receives responses
   - You don't need the plugin source code in your project

2. **Watcher + Beads:**
   - Watcher creates beads (`bd new`)
   - Watcher finalizes beads (`bd done` / `bd block`)
   - OpenCode does NOT manage beads directly

3. **Task → Bead Mapping:**
   - Every `.ai-handoff/tasks/{id}.json` maps to exactly one Bead
   - Bead ID is stored in `.ai-handoff/meta/{id}.bead`

---

## 5. Quick Setup Checklist

### First-Time Setup

- [ ] **Install OpenCode:** https://opencode.ai
- [ ] **Install antigravity-auth plugin:**
  ```bash
  # Add to ~/.config/opencode/opencode.json
  { "plugin": ["opencode-antigravity-auth@beta"] }
  
  # Authenticate
  opencode auth login
  ```
- [ ] **Install Beads:**
  ```bash
  curl -fsSL https://raw.githubusercontent.com/steveyegge/beads/main/scripts/install.sh | bash
  ```
- [ ] **Verify installation:**
  ```bash
  opencode --version
  bd --version
  ```

### Per-Project Setup

- [ ] **Initialize Beads:**
  ```bash
  bd init
  ```
- [ ] **Add rules submodule:**
  ```bash
  git submodule add https://github.com/Sesamsesam/antigravity-rules.git docs/rules
  ```
- [ ] **Create AGENTS.md:**
  ```markdown
  @docs/rules/AGENTS_CORE.md
  @docs/rules/AGENTS_BRIDGE.md
  ```

---

## 6. Future Extensions (Roadmap)

These are recommended OpenCode plugins to enhance agent capabilities when needed. They are **OpenCode plugins** (installed via `opencode.json`), NOT Antigravity repo code.

### 1. opencode-browser (Headless Browsing)
- **Goal**: Allow agents to verify web apps visually or look up documentation.
- **Capabilities**: `browser_open`, `browser_click`, `browser_screenshot`.
- **Use When**: You need deployed verification (e.g., "Check if the H1 renders correctly on localhost").

### 2. opencode-dcp (Development Context Protocol)
- **Repo**: `@tarquinen/opencode-dcp`
- **Goal**: Manage long context windows in massive tasks.
- **Capabilities**: Compacts older conversation turns to keep context fresh.
- **Use When**: Agents get "lost" or slow in long sessions (50+ turns).
- **Note**: Must be listed **AFTER** `opencode-antigravity-auth` in config.

### 3. opencode-rag (Local Knowledge)
- **Goal**: Semantic search for large codebases.
- **Capabilities**: "Find logic related to X" without reading every file.
- **Use When**: Project grows >100 files and agents struggle to find relevant code.

---

## References

- [OpenCode Documentation](https://opencode.ai)
- [opencode-antigravity-auth](https://github.com/NoeFabris/opencode-antigravity-auth)
- [Beads Agent Workflow](https://github.com/steveyegge/beads/blob/main/AGENT_INSTRUCTIONS.md)
- [Bridge Protocol Spec](./30_The_Bridge_Protocol.md)
