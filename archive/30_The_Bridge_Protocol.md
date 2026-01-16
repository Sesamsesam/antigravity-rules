# 30 THE BRIDGE PROTOCOL (PRODUCTION EDITION)

**Status**: ACTIVE
**Scope**: GLOBAL
**Components**: Antigravity (Planner), Watcher (Infrastructure), OpenCode (Executor)

## 1. Core Philosophy
*   **Watcher is the Judge**: Runs verification commands (`bun test`).
*   **OpenCode is the Builder**: Edits files only.
*   **Super Reliability**: Automation + Safety + Auditability + No Silent Failures.

## 2. Directory Structure (`.ai-handoff/`)
*   `tasks/`: Inbox for new tasks.
*   `running/`: Locked tasks currently executing.
*   `results/`: Single Source of Truth (deterministic outcomes).
*   `patches/`: Git diffs (Pre/Post verify).
*   `locks/`: Atomic locks (Worker + Task).
*   `logs/`: Full command logs (redacted, local-only).
*   `meta/`: Internal metadata (Bead IDs).
*   `pending/`: Tasks awaiting manual confirmation.
*   `failed/`: Terminal failures (optional archive).

## 3. Global Invariants (ALWAYS ENFORCED)

**Critical Operational Guarantees:**

1.  **Always Write Result**: ANY exit path (STOP/BLOCK/FAIL/SUCCESS) MUST write `results/{id}.json` with `status`, `reason`, and `exit_path`.
    *   This ensures **no "silent" failures**. Every task has a deterministic, auditable outcome.

2.  **Worker Lock TTL is Authoritative**: PID liveness checks are best-effort only.
    *   In Docker/CI, PIDs may be reused or unavailable. **TTL (2h default) is the ultimate arbiter.**
    *   If PID check is unavailable/unreliable, watcher uses **TTL-only behavior**.

3.  **Task Independence**: Pending/blocked tasks do **not** block the queue.
    *   Watcher continues to next task after moving one to `pending/` or writing `blocked` result.

4.  **Stop on Failure (Default)**: If a task fails (`exit_path: "failed"`), watcher **stops processing queue** by default.
    *   Prevents cascading failures. Override with `continue_on_failure: true` config (only when system is trusted).

## 4. The State Machine & Execution Order

**Watcher Implementation (Strict Sequence):**

0.  **Worker Lock**: Acquire `.ai-handoff/locks/__worker__.lock`. (FIFO Guarantee).
    *   **Lock Content**: `{ "pid": 123, "created_at": "ISO-8601", "host": "hostname" }`
    *   **On Startup**:
        *   If lock exists AND PID is alive on same host → EXIT (another watcher running).
        *   If lock exists AND (age > 2h OR PID not alive) → Replace stale lock.
        *   **Note**: PID check is best-effort; **TTL is authoritative** (critical for Docker/CI).
        *   If PID check unavailable, use TTL-only behavior.
    *   **On Shutdown**: Delete worker lock.
    *   **Scope**: Worker lock is held for **process lifetime**; releasing task lock ≠ releasing worker lock.

1.  **Validate**: Check task JSON schema.
    *   If invalid: Write result (`status: "failed"`, `reason: "schema_invalid"`), SKIP task.

2.  **Idempotency**: If `results/{id}.json` exists → SKIP (already done).

3.  **Safety Check**:
    *   **Dirty Check**: If `git status --porcelain` is not empty (and `allow_dirty` is false):
        *   Write result (`status: "blocked"`, `reason: "dirty_repo"`).
        *   SKIP task.
    *   **Branch Check**: If on `main` → **Auto-branch** to `feat/ai/{id}`.
        *   If checkout/create fails:
            *   Write result (`status: "blocked"`, `reason: "branch_checkout_failed"`).
            *   Include git state.
            *   SKIP task (leave in `tasks/` or move to `failed/`).

4.  **Atomic Lock**: Acquire `locks/{id}.lock`.
    *   **Lock Content**: `{ "created_at": "...", "pid": 123, "task_id": "{id}", "timeout_sec": 1800 }`
    *   **Stale Recovery**:
        *   If `results/{id}.json` exists → Delete lock, SKIP.
        *   Else if lock age > `max(timeout_sec*2, 2h)`:
            *   Write `results/{id}.json`: `status: "failed"`, `reason: "stale_lock_recovered"`.
            *   If bead exists: `bd block "stale lock recovered"`.
            *   If `attempt < max_attempts`:
                *   Increment `attempt` in task JSON (before re-queue).
                *   Move `running/{id}` → `tasks/{id}`.
            *   Else: Move to `failed/{id}` (terminal).
            *   Delete lock, continue to next task.

5.  **Atomic Move**: `tasks/{id}` → `running/{id}`.

6.  **Confirmation Gate**: If `requires_confirmation: true`:
    *   Move to `pending/{id}`.
    *   Capture git state: `branch`, `base_commit`, `dirty_before`, `status_porcelain`.
    *   Write `results/{id}.json`:
        *   `status: "needs_confirmation"`
        *   `reason: "requires_confirmation"`
        *   `exit_path: "needs_confirmation"`
        *   `git: { branch, base_commit, dirty_before, status_porcelain }`
        *   `task_snapshot`
    *   Release **task lock** (worker lock remains held).
    *   **Continue to next task** (pending tasks do NOT block queue).
    *   STOP (for this task only).

7.  **Create Bead**: Run `bd new "<bead_title>"` → write raw output to `meta/{id}.bead`.
    *   Store bead reference (not fragile ID parsing).

8.  **Edit (OpenCode)**: Invoke OpenCode to apply changes (respecting `timeout_sec`).
    *   **If OpenCode crashes/times out**:
        *   Watcher still saves patches + git status (partial evidence).
        *   Watcher does NOT revert.
        *   Write result: `status: "failed"`, `reason: "opencode_failed"`, `exit_path: "failed"`.
        *   Bead becomes `blocked`.
        *   Results include partial state.
        *   If `stop_on_failure: true` → STOP queue.

9.  **Audit 1 (Pre-Verify)**:
    *   Capture `git rev-parse HEAD` → `git.base_commit`.
    *   Capture `git diff --shortstat` → `git.diff_stat_pre`.
    *   Save `git diff > patches/{id}_pre.patch`.

10. **Verify (Watcher)**:
    *   Run each command in `commands_to_run` (in shell, with `timeout_sec`).
    *   Capture raw stdout/stderr per command.
    *   **Redact Secrets** (before storing):
        *   **Patterns** (minimum):
            *   `Bearer\s+[A-Za-z0-9\-_\.]+`
            *   `sk-[A-Za-z0-9]{10,}`
            *   `AIza[0-9A-Za-z\-_]{20,}`
            *   `anthropic[_-]?(api)?[_-]?key[:=]\s*\S+`
            *   `x-api-key[:=]\s*\S+`
            *   `Authorization[:=]\s*\S+`
            *   Env vars matching: `*KEY*`, `*TOKEN*`, `*SECRET*`
        *   **Size Limits**:
            *   Cap `stdout`/`stderr` in `results/{id}.json` to **10 KB** per command.
            *   If logs exceed cap, write **full redacted logs** to `.ai-handoff/logs/{id}.log` and note:
                ```json
                "commands": [
                  { "cmd": "bun test", "stdout": "[TRUNCATED - see logs/{id}.log]", "exit_code": 1 }
                ]
                ```
            *   **CRITICAL**: Logs in `logs/{id}.log` must **also be redacted** (or kept local-only, gitignored).

11. **Audit 2 (Post-Verify)**:
    *   Save `git diff > patches/{id}_post.patch`.
    *   Capture `git status --porcelain` → `git.status_after_verify`.
    *   Capture `git diff --shortstat` → `git.diff_stat_post`.
    *   Capture `git rev-parse HEAD` → `git.head_commit`.

12. **Write Result**: Save `results/{id}.json` (see schema below).

13. **Finalize**:
    *   **Fail Condition**: ANY of:
        *   `commands_to_run` command exits non-zero.
        *   Required bridge operation fails (validate, lock, move, bd, OpenCode, git).
    *   **On Fail**:
        *   Write result (`status: "failed"`, `reason: "verify_failed"` or specific, `exit_path: "failed"`).
        *   `bd block "<reason>"`.
        *   Release task lock.
        *   If `stop_on_failure: true` → STOP queue.
    *   **On Success**:
        *   Write result (`status: "success"`, `exit_path: "success"`).
        *   `bd done`.
        *   Release task lock.
        *   Continue to next task.

14. **Unlock**: Release task lock (worker lock remains held until watcher shutdown).

## 5. Task Schema (Antigravity Output)

```json
{
  "id": "YYYY-MM-DD_NN_slug",
  "bead_title": "Human readable title",
  "goal": "Objective",
  "done_condition": "Acceptance criteria",
  "prompt": "Instructions for OpenCode (EDIT files only)",
  "scope": ["src/target.ts"],
  "constraints": ["no arbitrary shell commands"],
  "commands_to_run": ["bun test"],
  "requires_confirmation": false,
  "allow_dirty": false,
  "timeout_sec": 1800,
  "risk_level": "low",
  "retry_policy": { "max_attempts": 1 },
  "attempt": 1
}
```

**Retry Mechanism**:
*   `attempt` field lives in **task JSON** (default: 1) and is **copied** to results.
*   **Increment timing**: Watcher increments `attempt` **before re-queueing** to `tasks/`.
*   **Results always reflect the attempt number used for that execution.**

## 6. Result Schema (Watcher Output)

```json
{
  "id": "YYYY-MM-DD_NN_slug",
  "status": "success|failed|blocked|needs_confirmation",
  "exit_path": "success|failed|blocked|needs_confirmation|skipped",
  "reason": "<canonical_reason>",
  "attempt": 1,
  "bead": { "title": "...", "raw_output": "..." },
  "git": {
    "base_commit": "abc123...",
    "head_commit": "def456...",
    "branch": "feat/ai/2024-01-15_01_example",
    "dirty_before": false,
    "dirty_after_verify": true,
    "status_before": "",
    "status_after_verify": "M  src/test.ts",
    "diff_stat_pre": "1 file changed, 2 insertions(+)",
    "diff_stat_post": "2 files changed, 5 insertions(+), 1 deletion(-)"
  },
  "commands": [
    { "cmd": "bun test", "exit_code": 0, "stdout": "[REDACTED]", "stderr": "" }
  ],
  "artifacts": {
    "patch_pre": "patches/2024-01-15_01_example_pre.patch",
    "patch_post": "patches/2024-01-15_01_example_post.patch",
    "logs": "logs/2024-01-15_01_example.log"
  },
  "task_snapshot": { },
  "timestamp": "2024-01-15T00:33:19+08:00"
}
```

**Canonical Reasons** (enum for `reason` field):
*   `schema_invalid`: Task JSON failed validation.
*   `dirty_repo`: Repo had uncommitted changes (and `allow_dirty: false`).
*   `branch_checkout_failed`: Could not create/checkout task branch.
*   `requires_confirmation`: Task flagged for manual approval.
*   `opencode_failed`: OpenCode crashed or timed out.
*   `verify_failed`: One or more `commands_to_run` exited non-zero.
*   `stale_lock_recovered`: Task lock was stale; task re-queued or failed.
*   `internal_error`: Unexpected watcher error.

## 7. No-Regression Policy (GLOBAL)

**Do not remove or weaken reliability guarantees unless replaced by stronger ones.**

1.  **Idempotency**: Skip if result exists + Stale Lock Recovery.
2.  **Single Worker**: Worker Lock (FIFO, TTL-based).
3.  **Safety**: Dirty Block + Auto-Branch off `main`.
4.  **Audit**: Pre/Post Patches + Redacted Logs + Git Status.
5.  **Role**: Watcher verifies; OpenCode edits.
6.  **Always Write Result**: No silent failures.

## 8. Operations / Runbook

**Start System**:
```bash
# Terminal 1: OpenCode Server
opencode serve

# Terminal 2: Bridge Watcher
bun scripts/bridge.ts
```

**Approve Pending Task**:
```bash
mv .ai-handoff/pending/{id}.json .ai-handoff/tasks/{id}.json
# Or use: bridge approve {id} (if implemented)
```

**Clear Stale Worker Lock** (after crash):
```bash
rm .ai-handoff/locks/__worker__.lock
```

**Debug Blocked Bead**:
1.  Check: `.ai-handoff/results/{id}.json` → `reason` and `exit_path` fields.
2.  Check: `.ai-handoff/meta/{id}.bead` → Bead raw output.
3.  Run: `bd ls` to see bead status.
4.  Review patches: `cat .ai-handoff/patches/{id}_pre.patch`

**Resume After Crash**:
1.  Delete stale worker lock: `rm .ai-handoff/locks/__worker__.lock`
2.  Restart watcher: `bun scripts/bridge.ts`
3.  Watcher auto-recovers stale task locks (Step 4).

**Monitor Queue**:
```bash
# Count tasks by state
ls .ai-handoff/tasks/ | wc -l       # Queued
ls .ai-handoff/running/ | wc -l     # Running
ls .ai-handoff/results/ | wc -l     # Done
ls .ai-handoff/pending/ | wc -l     # Awaiting approval

# Check recent failures
grep -l '"exit_path": "failed"' .ai-handoff/results/*.json | tail -5
```

**Configuration** (optional `bridge.config.json`):
```json
{
  "stop_on_failure": true,
  "worker_lock_ttl_sec": 7200,
  "log_size_cap_kb": 10,
  "redaction_patterns": ["custom_pattern"]
}
```
