# Secure Watcher Specification

> **Canonical security specification. Supersedes watcher security in `30_The_Bridge_Protocol.md`.**

---

## Executive Summary

**Purpose**: Build a production-grade, OSS-ready Bridge Watcher that is safe by default.

### Core Promise

If you only ever run `bridge run`, the system will not:
- open inbound access to your machine
- execute untrusted code directly on your host
- leak secrets to disk or console
- allow tasks to escape the workspace

---

## 1. Non-Negotiable Invariants

### Security Model

1. **No Inbound Exposure**: Bridge never runs a network server, never binds ports
2. **Sandbox Required**: All untrusted execution happens in Docker with `--network=none`
3. **Secrets Never Persist Unredacted**: All outputs scanned before writing
4. **Worktree Confinement**: All writes constrained to `.ai-handoff/tmp/`
5. **No Raw Calls**: All fs/git operations through wrappers only

### Operational Model

1. **Always Write Result**: Every task produces `results/{id}.json`
2. **Idempotency**: Skip if result exists
3. **Single Worker**: Worker lock prevents concurrent execution
4. **Atomic Operations**: Temp file + rename pattern

---

## 2. Safety Wrappers

### fsSafe

All filesystem operations MUST go through `fsSafe`:

```typescript
fsSafe.read(path)      // O_NOFOLLOW, path validation
fsSafe.writeAtomic()   // Parent chain check, temp+rename
fsSafe.isContained()   // Verify path is under allowed root
```

### gitSafe

All git operations MUST go through `gitSafe`:

```typescript
gitSafe.status()       // With hooks disabled, timeout
gitSafe.diff()         // With size limits
gitSafe.worktreeAdd()  // With path validation
```

### StreamScanner

All output scanning MUST use `StreamScanner`:

```typescript
scanner.scan(chunk)    // With 8KB overlap buffer
scanner.finalize()     // Catch patterns at end
```

---

## 3. Docker Sandbox Policy

### Required Flags

```bash
--network=none                      # No network access
--read-only                         # Container root read-only
--cap-drop=ALL                      # Drop all capabilities
--security-opt=no-new-privileges    # Prevent privilege escalation
--user <uid>:<gid>                  # Run as non-root
```

### Required Mounts

```bash
-v <wsPath>:/workspace:rw           # Only worktree is writable
--tmpfs /tmp:noexec,nosuid,nodev    # Temp space, no execution
```

### Forbidden

- Port publishing (`-p`)
- Host networking (`--network=host`)
- Privileged mode (`--privileged`)
- Mounting host directories outside worktree

---

## 4. Secret Handling

### Detection Patterns

```regex
Bearer\s+[A-Za-z0-9\-_\.]+
sk-[A-Za-z0-9]{10,}
AIza[0-9A-Za-z\-_]{20,}
ghp_[A-Za-z0-9]{36}
github_pat_[A-Za-z0-9_]{22,}
AKIA[A-Z0-9]{16}
-----BEGIN.*PRIVATE KEY-----
https?://[^:]+:[^@]+@
```

### On Detection

1. Delete worktree immediately
2. Write safe incident record (pattern name + location, NO raw secret)
3. Write NO logs, patches, or artifacts

### Console Policy

- NEVER print raw child output to console
- Stream to memory → Scan → Print redacted OR block

---

## 5. Adversarial Harness

MUST pass before system is considered usable:

| Test | Proves |
|------|--------|
| Symlink Race | O_NOFOLLOW works |
| Untracked Secret | New files are scanned |
| Overlap Leak | Chunk boundary secrets caught |
| Delete Escape | Cleanup confined |
| Hook Trap | Git hooks disabled |
| Exfil Attempt | Network denied |

---

## 6. CI Policy

When `CI=true`:

- DockerRunner: **Required**
- gitleaks: **Required**
- Exceptions: **Forbidden**
- Harness: **Must pass**

---

## 7. Exception Handling

### Local Only

Exceptions are allowed locally but:
- Must be explicit in task JSON
- Must be recorded in result JSON
- Forbidden in CI

### Valid Exceptions

```json
{
  "allow_large_file_head_tail_scan": true,
  "allow_fallback_fs": true,
  "require_gitleaks": false
}
```

---

## References

- Full protocol: `30_The_Bridge_Protocol.md`
- Docker policy: `32_Docker_Sandbox_Policy.md`
- Diamond rules: `34_Diamond_Rules.md`
- Master blueprint: `architecture/BLUEPRINT.md`
