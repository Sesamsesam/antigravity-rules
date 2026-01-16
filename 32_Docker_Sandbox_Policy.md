# Docker Sandbox Policy

> **Operational truth for container execution.**

---

## Overview

All untrusted execution (OpenCode, verification commands) runs inside Docker containers with strict security settings.

---

## Required Docker Flags

```bash
docker run --rm \
  --network=none \                     # No network access
  --read-only \                        # Container root read-only
  --cap-drop=ALL \                     # Drop all capabilities
  --security-opt=no-new-privileges \   # Prevent privilege escalation
  --pids-limit=256 \                   # Limit processes
  --memory=2g \                        # Memory limit
  --cpus=2 \                           # CPU limit
  --user "$(id -u):$(id -g)" \         # Run as host user
  -v "$WSPATH:/workspace:rw" \         # Only worktree mounted
  --tmpfs /tmp:noexec,nosuid,nodev \   # Temp space
  -w /workspace \                      # Set working directory
  bridge-runner:dev \                  # Our custom image
  <command>
```

---

## Forbidden Configurations

| Configuration | Why Forbidden |
|---------------|---------------|
| `-p` (port publish) | Opens inbound access |
| `--network=host` | Shares host network |
| `--privileged` | Full host access |
| `--cap-add` | Adds capabilities |
| `-v /:/host` | Mounts host root |

---

## Mount Rules

### Allowed

- Worktree: `-v $wsPath:/workspace:rw`
- Temp: `--tmpfs /tmp`

### Forbidden

- Home directory
- Docker socket
- Any path outside worktree

---

## Environment Variables

### Allowlist Only

Only pass explicitly allowed env vars:

```bash
-e CI="$CI"
-e NODE_ENV="$NODE_ENV"
-e OPENAI_API_KEY="$OPENAI_API_KEY"
```

### Never Pass

- Full host environment
- SSH keys
- Cloud credentials (unless explicitly needed)

---

## Image Management

### Build Image

```bash
cd bridge-watcher
docker build -t bridge-runner:dev .
```

### Rebuild After Changes

```bash
docker build --no-cache -t bridge-runner:dev .
```

---

## Network Policy

### Default: None

```bash
--network=none
```

No DNS, no internet, no local network.

### Exception (Local Only)

If task requires network:

```json
"allow_network": true
```

This uses default bridge network. Forbidden in CI.

---

## CI Requirements

In CI:
- `--network=none` is **mandatory**
- `allow_network` exception is **forbidden**
- Image must be built fresh or pulled from trusted registry
