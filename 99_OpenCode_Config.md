---
description: OpenCode Configuration for Antigravity Ecosystem
---

# OpenCode Configuration

This agent is part of the **Antigravity Ecosystem**. It adheres to the Prime Directive and Professional Workflow.

## 📚 Rule Imports
All agents must read and follow these global rules:

@import "/Users/yeti/Documents/Antigravity/Rules/00_PRIME_DIRECTIVE.md"
@import "/Users/yeti/Documents/Antigravity/Rules/10_The_Workflow.md"
@import "/Users/yeti/Documents/Antigravity/Rules/20_The_Agent_Protocol.md"
@import "/Users/yeti/Documents/Antigravity/Rules/30_The_Bridge_Protocol.md"

## 🤖 Context Instructions
1.  **Project Root**: Always look for `task.md` or `tasks/active_context.md` to understand current state.
2.  **Architecture**: Read `docs/architecture.md` before creating new files.
3.  **Authentication**: Use `opencode-antigravity-auth` for all model interactions.
