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

## References

- [OpenCode Documentation](https://opencode.ai)
- [opencode-antigravity-auth](https://github.com/NoeFabris/opencode-antigravity-auth)
- [Beads Agent Workflow](https://github.com/steveyegge/beads/blob/main/AGENT_INSTRUCTIONS.md)
- [Bridge Protocol Spec](./30_The_Bridge_Protocol.md)
