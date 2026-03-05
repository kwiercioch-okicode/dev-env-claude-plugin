---
description: "Initialize dev environment config for a project. Usage: /dev:init [path]"
---

Invoke the dev:dev-env skill to initialize a `.dev-env.yml` config for this project. This config tells `/dev:env` how to manage worktrees and infrastructure.

<UserRequest>
  $ARGUMENTS
</UserRequest>

**Steps:**

1. Determine project root:
   - If $ARGUMENTS contains a path, use it
   - Otherwise use the git toplevel of current directory
   - Verify it's a git repository

2. Check if `.dev-env.yml` already exists. If yes, show it and ask if user wants to reconfigure.

3. Auto-detect project type by scanning files:

   | Files found | Detected type |
   |---|---|
   | `docker-compose.yml` + multiple repo subdirs with `.git` | Multi-repo (Docker + services) |
   | `sst.config.ts` or `sst.config.js` | SST (Serverless Stack) |
   | `next.config.*` | Next.js |
   | `docker-compose.yml` + `Dockerfile` | Docker Compose app |
   | `bun.lock` or `bunfig.toml` | Bun project |
   | `package.json` with `scripts.dev` | Node.js project |
   | `composer.json` | PHP project |
   | `Cargo.toml` | Rust project |
   | `go.mod` | Go project |
   | `pyproject.toml` or `requirements.txt` | Python project |

4. Show detected type, ask user to confirm or override.

5. Ask project-specific questions:

   **All types:**
   - Project name (default: directory name)
   - Worktree directory (default: `.worktrees`)

   **Multi-repo:**
   - Which subdirectories are separate repos?
   - Which need worktrees?
   - Base branch for each (default: main)

   **Docker Compose:**
   - Which compose file? (auto-detect)
   - Needs gateway/proxy? (nginx, traefik, none)

   **Dev server (Next.js, Node, Bun):**
   - Dev command (auto-detect from package.json scripts)
   - Port (auto-detect or default 3000)
   - Env file template (scan for .env.local, .env.dev, .env.example)

   **SST:**
   - Stage naming strategy: worktree name as stage (default) or custom prefix
   - `sst dev` runs with `--stage <name>` per worktree
   - No port management needed (Lambda runs in AWS)
   - Ask about `.env` or `.env.<stage>` patterns
   - Check for `sst.config.ts` to confirm app name

   **With subdomains:**
   - Domain pattern
   - DNS method (dnsmasq, /etc/hosts, none)

6. Generate `.dev-env.yml` with minimal config (only include non-default values).

7. Add to `.gitignore` if user wants it local-only, or leave for committing.

8. Verify `.worktrees/` is in `.gitignore`. If not, add it.

9. Show summary:
   ```
   Created .dev-env.yml for <project-name>
   Type: <detected-type>
   Services: <list>

   Use /dev:env create <branch> to create a dev environment
   Use /dev:env list to see all environments
   ```
