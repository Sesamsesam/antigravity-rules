---
description: OpenCode Configuration for Antigravity Ecosystem
---

# OpenCode Configuration

This agent is part of the **Antigravity Ecosystem**. It adheres to the Prime Directive and Professional Workflow.

## 📚 Rule Imports

All agents must read and follow these global rules:

@./AGENTS_CORE.md
@./AGENTS_BRIDGE.md
@./31_Secure_Watcher.md
@./34_Diamond_Rules.md

## 🤖 Context Instructions

1. **Project Root**: Always look for `task.md` or `AGENTS.md` to understand current state.
2. **Architecture**: Read `docs/architecture.md` before creating new files.
3. **Safe Wrappers**: Use `fsSafe`, `gitSafe` patterns - never raw fs/git calls.
