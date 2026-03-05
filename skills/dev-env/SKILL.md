---
name: dev-env
description: "Use when managing isolated dev environments - creating worktrees, starting/stopping infrastructure, checking status. Triggers on /dev command or when user mentions dev environments, worktrees, or feature branch isolation."
---

# Dev Environment Management

## Overview

Manage isolated dev environments using git worktrees. Each environment gets its own branch checkout and (optionally) its own running infrastructure - containers, dev servers, reverse proxy, DNS.

**Announce at start:** "I'm using the dev-env skill to manage your dev environment."

## Configuration

This skill reads `.dev-env.yml` from the project root. If it doesn't exist, run `/dev-init` first.

### Config format

```yaml
name: my-project

worktrees:
  dir: .worktrees                    # where worktrees are created
  repos:                             # omit for single-repo projects
    - path: backend
      branch_from: main
    - path: frontend
      branch_from: main

services:
  app:
    type: dev-server                 # dev-server | docker-compose | custom
    command: npm run dev             # start command
    env:
      template: .env.local           # env file to copy as base
      overrides:                     # per-worktree overrides
        API_URL: http://localhost:<port>
    ports:
      dev: auto                      # auto-assign (base: 3100, step: 100)
    node_modules: symlink            # symlink | install | none

  backend:
    type: docker-compose
    file: docker-compose.yml
    project_prefix: myapp

  gateway:
    type: nginx                      # nginx | traefik | none
    template: nginx/worktree.conf.template
    conf_dir: nginx/conf.d
    reload_command: docker exec nginx nginx -s reload

dns:
  type: none                         # dnsmasq | hosts | none
  domain: dev.myapp.local
  subdomains:
    api: <name>.dev.api.myapp.local
    app: <name>.dev.myapp.local

scripts:                             # optional custom scripts
  up: ./scripts/worktree-up.sh
  down: ./scripts/worktree-down.sh
  status: ./scripts/worktree-status.sh
```

## Operations

### Reading config

1. Look for `.dev-env.yml` in the project root (git toplevel)
2. If not found, tell user to run `/dev-init` and STOP
3. Parse the YAML config

### List (default action)

1. For each repo (or single repo), run `git worktree list`
2. If `scripts.status` is defined, run it
3. Otherwise, check each service:
   - Docker: `docker ps --filter name=<prefix>-<name>`
   - Dev server: check if port is in use (`lsof -ti :<port>`)
4. Present combined view with status (running/stopped/partial)

### Create

1. For each repo, check if branch exists (local or remote)
2. Create worktree: `git worktree add <dir>/<short-name> <branch>`
   - Short name: strip `feature/`, `fix/`, `update/` prefixes
3. Run setup per service type:
   - `node_modules: symlink` -> symlink from main repo
   - `node_modules: install` -> run package manager install
   - Copy env files from template
   - Apply overrides (replace `<name>` with worktree name, `<port>` with assigned port)
4. Start infrastructure (same as `up`)

### Up

1. If `scripts.up` is defined, run it with worktree name as argument
2. Otherwise, for each service in order:
   a. **docker-compose**: `docker compose -f <file> -p <prefix>-<name> up -d`
   b. **dev-server**: start command in background, save PID, redirect to log file
   c. **nginx**: generate config from template (replace `{{NAME}}`, `{{PORT}}`), reload
   d. **traefik**: generate labels/config
3. Verify by checking ports/containers

### Down

1. If `scripts.down` is defined, run it with worktree name as argument
2. Otherwise, reverse order of services:
   a. Kill dev-server by PID or port
   b. Remove nginx/traefik config, reload
   c. `docker compose -p <prefix>-<name> down`
3. Clean up port allocation and log files
4. Git worktrees are preserved

### Remove

1. Safety check: uncommitted changes, unpushed commits
2. Warn user and ask for confirmation if dirty
3. Run `down` to stop infrastructure
4. Remove git worktrees: `git worktree remove <path> --force`
5. Run `git worktree prune`

### Switch

1. Check for uncommitted changes - WARN and STOP if dirty
2. For each repo:
   - Switching TO feature: remove worktree, checkout branch in main dir
   - Switching BACK to main/master: checkout default, recreate worktree
3. Git doesn't allow same branch in two places - switch handles this automatically

### Status

1. If `scripts.status` is defined, run it with worktree name
2. Otherwise check each service and report:
   - Container status and uptime
   - Port availability
   - HTTP response (if URL pattern known)
   - Last log lines

## Port allocation

- Store in `.worktree-ports` file in project root
- Format: `<name> <port1> [<port2> ...]`
- Auto-assign: start at base (default 3100), increment by step (default 100)
- Add `.worktree-ports` to `.gitignore`

## State files

All state is stored in the project root:
- `.worktree-ports` - port assignments
- `.dev-env.yml` - configuration (may be committed or gitignored)
- `logs/worktree/<name>-*.log` - service logs
- `logs/worktree/<name>-*.pid` - PID files

## Safety

- ALWAYS check for uncommitted changes before switch/remove
- ALWAYS ask user confirmation before removing worktrees with unpushed work
- NEVER force-remove without showing what will be lost
- Add `.worktrees/` to `.gitignore` if not already there

## Common patterns

**Simple Node.js project:**
- One repo, one dev server, auto port assignment

**Docker Compose project:**
- One repo, compose with project prefix per worktree

**Multi-repo with gateway:**
- Multiple repos, Docker backend, Node frontend, nginx proxy, DNS subdomains
- Each worktree gets its own container + dev server + nginx config + subdomain
