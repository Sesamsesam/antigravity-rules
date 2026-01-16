# Repository Separation Guide

> **Why Bridge is separate from your app, and how to keep it that way.**

---

## The Principle

**Bridge is a development tool. It never deploys with your app.**

| Repo | Purpose | Deploys To |
|------|---------|------------|
| `bridge-watcher/` | Orchestration tool | Never (dev only) |
| `my-app/` | Your application | Cloudflare |

---

## Why Separate?

1. **Clean deploys**: Your app repo contains only app code
2. **No accidents**: Bridge code can't accidentally ship
3. **Reusability**: Use Bridge across multiple projects
4. **OSS-ready**: Bridge can be open-sourced independently

---

## How It Works

### Bridge runs *against* your app repo

```bash
# You're in your app repo
cd ~/dev/my-app

# Bridge is installed elsewhere, runs here
bridge run --repo .
```

### State lives in your app repo (gitignored)

```
my-app/
├── src/              # Your app code (deployed)
├── .ai-handoff/      # Bridge state (NOT deployed)
│   ├── tasks/
│   ├── results/
│   └── ...
└── .gitignore        # Includes .ai-handoff/
```

---

## .gitignore Requirements

In every app repo, add:

```gitignore
# Bridge Watcher state
.ai-handoff/
```

The `bridge init` command does this automatically.

---

## Directory Structure

```
~/dev/
├── bridge-watcher/      # The tool
│   ├── src/
│   ├── Dockerfile
│   └── docs/
│
├── my-app/              # Your app (Cloudflare)
│   ├── src/
│   ├── .ai-handoff/     # (gitignored)
│   └── .gitignore
│
└── another-project/     # Another app
    ├── src/
    ├── .ai-handoff/     # (gitignored)
    └── .gitignore
```

---

## Cloudflare Deployment

Your Cloudflare deployment sees only:

```
my-app/
├── src/
├── package.json
└── ... (app files)
```

It does **not** see:
- `.ai-handoff/` (gitignored)
- `bridge-watcher/` (different repo)

---

## Common Mistakes to Avoid

| Mistake | Problem | Fix |
|---------|---------|-----|
| Bridge code in app repo | Ships with app | Keep in separate repo |
| `.ai-handoff/` not gitignored | State tracked | Run `bridge init` |
| Running `bridge run` in Bridge repo | Wrong target | Run in app repo |

---

## Quick Setup

```bash
# 1. Clone Bridge (once)
cd ~/dev
git clone <bridge-repo> bridge-watcher
cd bridge-watcher && bun install

# 2. Initialize your app (once per app)
cd ~/dev/my-app
bridge init

# 3. Use daily
cd ~/dev/my-app
bridge run
```
