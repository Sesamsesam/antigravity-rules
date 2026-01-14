# Antigravity Bridge Protocol - Execution Contract

**Status**: ACTIVE | **Scope**: GLOBAL  
**Import**: `@docs/rules/AGENTS_BRIDGE.md`  
**Full Spec**: See `30_The_Bridge_Protocol.md` for complete technical details.

This is the execution contract between the **Watcher** (bridge infrastructure) and **OpenCode** (AI executor).

---

## Core Philosophy

| Role | Responsibility |
|------|---------------|
| **Watcher** | The Judge - runs verification, manages state, finalizes beads |
| **OpenCode** | The Builder - edits files only, follows prompts, never runs tests |

**Super Reliability**: Automation + Safety + Auditability + No Silent Failures

---

## Deterministic Execution Order

```
1. Safety Check    → Block if dirty repo or on main (auto-branch if needed)
2. Worker Lock     → FIFO guarantee (process lifetime)
3. Task Lock       → Per-task lock with stale recovery
4. OpenCode Edit   → Invoke OpenCode to modify files
5. Pre-Verify      → git diff, base_commit, shortstat
6. Verification    → Watcher runs commands_to_run
7. Post-Verify     → git diff, status, redaction
8. Write Result    → Always (success or fail)
9. Finalize Bead   → bd done or bd block
```

---

## Critical Guarantees

### 1. Always Write Result
Every task produces `results/{id}.json`, even on failure. **No silent failures.**

### 2. Stop on Failure (Default)
Watcher stops queue on first failure. Override with config if needed.

### 3. Auto-Branch from Main
If on `main` branch, Watcher creates `feat/ai/{id}` automatically.

### 4. Worker Lock TTL
PID checks are best-effort; **TTL (2h) is authoritative**. Safe for Docker/CI.

### 5. Task Independence
Pending/blocked tasks do not block the queue. Watcher continues to next.

---

## Canonical Reasons

When a task fails or blocks, `reason` is one of:

| Reason | Meaning |
|--------|---------|
| `schema_invalid` | Task JSON failed validation |
| `dirty_repo` | Uncommitted changes (and `allow_dirty: false`) |
| `branch_checkout_failed` | Couldn't create/checkout task branch |
| `requires_confirmation` | Task flagged for manual approval |
| `opencode_failed` | OpenCode crashed or timed out |
| `verify_failed` | One or more commands exited non-zero |
| `stale_lock_recovered` | Task lock was stale, recovered |
| `internal_error` | Unexpected watcher error |

---

## Confirmation Gating

If task has `requires_confirmation: true`:
1. Watcher moves to `pending/{id}.json`
2. Writes result with `status: "needs_confirmation"`
3. **Continues to next task** (does not block queue)
4. Resume by moving file back to `tasks/`

---

## Secrets Redaction

All logs are redacted before saving. Patterns include:
- `Bearer` tokens
- `sk-*` API keys
- `AIza*` Google keys
- Any env var matching `*KEY*`, `*TOKEN*`, `*SECRET*`

**Size Cap**: 10KB per command in results; full logs to `logs/{id}.log`

---

## No-Regression Policy

**Do not remove or weaken reliability guarantees.**

1. **Idempotency**: Skip if result exists + Stale Lock Recovery
2. **Single Worker**: Worker Lock (FIFO, TTL-based)
3. **Safety**: Dirty Block + Auto-Branch
4. **Audit**: Pre/Post Patches + Redacted Logs + Git Status
5. **Always Write Result**: No silent failures

---

## Quick Reference

**Directories** (`.ai-handoff/`):
- `tasks/` - Inbox
- `running/` - Locked tasks
- `results/` - Outcomes (single source of truth)
- `patches/` - Git diffs (pre/post)
- `locks/` - Worker + task locks
- `pending/` - Awaiting approval
- `logs/` - Full redacted logs

**Commands**:
```bash
# Approve pending task
mv .ai-handoff/pending/{id}.json .ai-handoff/tasks/

# Clear stale worker lock (after crash)
rm .ai-handoff/locks/__worker__.lock

# Check blocked beads
grep -l '"exit_path": "failed"' .ai-handoff/results/*.json
```

---

## Full Specification

For complete technical details including:
- Full 15-step state machine
- Task and Result JSON schemas
- Lock metadata format
- All redaction patterns

See: [30_The_Bridge_Protocol.md](./30_The_Bridge_Protocol.md)
