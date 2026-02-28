---
name: codex-agents
description: "Orchestrate OpenAI Codex coding agents from OpenClaw. Use when: (1) building features or apps, (2) fixing bugs across repos, (3) reviewing PRs, (4) refactoring codebases, (5) running parallel coding tasks. Requires Codex CLI installed and authenticated (`codex auth`). NOT for: simple one-line edits (use edit tool), reading code (use read tool), or non-coding tasks."
metadata:
  openclaw:
    emoji: "🧬"
    requires:
      anyBins: ["codex"]
---

# Codex Agents

Orchestrate OpenAI Codex coding agents via OpenClaw's exec tool with PTY support.

## Prerequisites

1. Install Codex CLI: `npm install -g @openai/codex`
2. Authenticate: `codex auth` (opens browser OAuth — uses your OpenAI account)
3. Verify: `codex --version`

## Core Pattern

Always use `pty:true` — Codex is an interactive terminal app.

### One-Shot Task

```bash
exec pty:true workdir:~/projects/myapp command:"codex exec --full-auto 'Add dark mode toggle to the settings page'"
```

### Background Task (Long-Running)

```bash
exec pty:true workdir:~/projects/myapp background:true command:"codex exec --full-auto 'Refactor auth module to use JWT tokens. When finished, run: openclaw system event --text \"Done: JWT auth refactor\" --mode now'"
```

Then monitor:
```
process action:log sessionId:<id>
process action:poll sessionId:<id>
```

### Notify on Completion

Append this to any prompt for automatic notification when the agent finishes:

```
When completely finished, run: openclaw system event --text "Done: <summary>" --mode now
```

This triggers an immediate wake — delivers to your configured channel (Telegram, Discord, etc).

## Execution Modes

| Flag | Behavior |
|------|----------|
| `exec "prompt"` | One-shot, exits when done |
| `--full-auto` | Sandboxed, auto-approves file changes in workspace |
| `--yolo` | No sandbox, no approvals (fastest, most dangerous) |

**Default recommendation:** `--full-auto` for most tasks. Use `--yolo` only when speed matters and you trust the task scope.

## Common Workflows

### Build a Feature

```bash
exec pty:true workdir:~/projects/myapp background:true command:"codex exec --full-auto 'Build a REST API for user management with CRUD endpoints. Use Express and Prisma. Include tests.

When completely finished, run: openclaw system event --text \"Done: User management API\" --mode now'"
```

### Fix a Bug

```bash
exec pty:true workdir:~/projects/myapp command:"codex exec --full-auto 'Fix issue #42: Login fails when email has uppercase letters. Write a test that reproduces it first, then fix.'"
```

### Review a PR

Clone or worktree to avoid touching your working branch:

```bash
# Create a worktree for safe review
exec command:"cd ~/projects/myapp && git worktree add /tmp/pr-review-130 origin/feature-branch"

# Review it
exec pty:true workdir:/tmp/pr-review-130 command:"codex exec 'Review this branch vs main. Focus on: correctness, edge cases, security. Summarize findings.'"

# Cleanup
exec command:"cd ~/projects/myapp && git worktree remove /tmp/pr-review-130"
```

### Parallel Tasks (Multiple Agents)

Use git worktrees to run multiple Codex agents simultaneously:

```bash
# Create worktrees
exec command:"cd ~/projects/myapp && git worktree add -b fix/issue-10 /tmp/issue-10 main"
exec command:"cd ~/projects/myapp && git worktree add -b fix/issue-11 /tmp/issue-11 main"

# Launch parallel agents
exec pty:true workdir:/tmp/issue-10 background:true command:"codex exec --full-auto 'Fix issue #10: <description>. Commit when done.

When finished, run: openclaw system event --text \"Done: issue-10\" --mode now'"

exec pty:true workdir:/tmp/issue-11 background:true command:"codex exec --full-auto 'Fix issue #11: <description>. Commit when done.

When finished, run: openclaw system event --text \"Done: issue-11\" --mode now'"

# Monitor all
process action:list
```

After completion, push branches and create PRs:
```bash
exec workdir:/tmp/issue-10 command:"git push -u origin fix/issue-10"
exec command:"gh pr create --repo user/repo --head fix/issue-10 --title 'fix: issue 10' --body 'Automated fix'"
```

### Scratch Work (No Existing Repo)

Codex requires a git repo. For scratch/prototype work:

```bash
exec command:"mkdir -p /tmp/scratch && cd /tmp/scratch && git init"
exec pty:true workdir:/tmp/scratch command:"codex exec --full-auto 'Build a snake game in Python with pygame'"
```

## Monitoring Background Agents

```bash
# List all running sessions
process action:list

# View output from a session
process action:log sessionId:<id>

# Check if still running
process action:poll sessionId:<id>

# Send input if agent asks a question
process action:submit sessionId:<id> data:"yes"

# Kill a stuck agent
process action:kill sessionId:<id>
```

## Rules

1. **Always use `pty:true`** — Codex needs a terminal
2. **Use `workdir`** — scope the agent to the project directory
3. **Git repo required** — Codex won't run outside one; `git init` for scratch
4. **Don't hand-code** — if Codex fails, respawn or ask the user; don't silently take over
5. **Verify completion** — check output/files before telling the user it's done
6. **One update on start, one on finish** — don't spam progress unless something changes
7. **Never run Codex in the OpenClaw workspace** — keep agent work in project directories
