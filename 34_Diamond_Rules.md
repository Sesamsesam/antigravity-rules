# The 10 Diamond Rules

> **If you forget everything else, remember these.**

---

## 1. One Command Only

You run tasks only via: `bridge run`

You do not manually run OpenCode or verification commands outside Bridge.

---

## 2. No Inbound Exposure

Bridge must **never**:
- Run an HTTP/WebSocket server
- Bind to `0.0.0.0`
- Publish ports from Docker (`-p` is forbidden)

---

## 3. Sandbox is Mandatory

All task execution (OpenCode + verify) runs in Docker with:
- `--network=none`
- `--read-only` container root
- `--cap-drop=ALL`
- Only worktree mounted

---

## 4. Secrets Never Hit Disk or Console

Bridge must buffer and scan stdout/stderr/diffs/files **before**:
- Printing anything to console
- Writing logs/patches/artifacts

If secrets detected: delete worktree, write safe incident, write nothing else.

---

## 5. Secretless Contract

- `.env` is gitignored and blocked from creation
- `.env.example` is allowed (placeholders only)
- Runtime secrets via environment variables only

---

## 6. Worktree Confinement

All writes happen only under: `<repo>/.ai-handoff/tmp/ws-<id>/`

Any path that resolves outside is blocked.

Cleanup can only delete inside `.ai-handoff/tmp/`.

---

## 7. No Raw fs/git Calls

All operations go through wrappers:
- `fsSafe` for filesystem
- `gitSafe` for git
- `StreamScanner` for output

Enforce with tests: fail if raw calls appear outside wrappers.

---

## 8. Proof Before Progress

`bridge harness` must pass before Bridge is considered usable.

Harness includes:
- Symlink race
- Untracked secret
- Overlap leak
- Delete escape
- Hook trap
- Exfil attempt

---

## 9. CI is the Strict Authority

In CI:
- DockerRunner required
- gitleaks required
- Exceptions forbidden
- Harness must pass on every PR

---

## 10. Separation of Concerns

- Bridge lives in its own repo (tooling)
- App lives in its own repo (Cloudflare deploy)
- `.ai-handoff/` is always gitignored in app repos

---

## The One Thing to Remember

**If you only ever run `bridge run`, you're safe—because Bridge refuses unsafe operation by default.**
