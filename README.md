# openclaw-codex-agents

An [OpenClaw](https://openclaw.ai) skill that lets your AI assistant orchestrate [OpenAI Codex](https://openai.com/index/codex/) coding agents — spawn background tasks, run parallel fixes, review PRs, and get notified when work is done.

## What it does

- Launch Codex agents from chat (Telegram, Discord, etc.)
- Run tasks in the background with automatic completion notifications
- Parallel agents via git worktrees for batch work
- PR review workflows with safe branch isolation
- Monitor, interact with, or kill running agents

## Prerequisites

- [OpenClaw](https://docs.openclaw.ai) installed and configured
- [Codex CLI](https://github.com/openai/codex) installed: `npm install -g @openai/codex`
- Codex authenticated: `codex auth` (opens browser OAuth to your OpenAI account)

## Install

Copy the `codex-agents/` folder into your OpenClaw skills directory:

```bash
# Clone this repo
git clone https://github.com/alymatcha/openclaw-codex-agents.git

# Copy skill to your OpenClaw skills directory
cp -r openclaw-codex-agents/codex-agents ~/.openclaw/skills/
```

Or if using [ClawHub](https://clawhub.com):
```bash
clawhub install codex-agents
```

## Quick example

Once installed, just ask your assistant:

> "Build a REST API for todos in ~/projects/my-app"

Your assistant will:
1. Spawn a Codex agent in the project directory
2. Run it in the background with `--full-auto`
3. Notify you on completion via your configured channel

## How it works

The skill teaches your OpenClaw assistant how to:

| Pattern | Description |
|---------|-------------|
| One-shot | `codex exec --full-auto "prompt"` for quick tasks |
| Background | Long-running tasks with `openclaw system event` notification on completion |
| Parallel | Git worktrees + multiple Codex agents for batch fixes |
| PR Review | Safe worktree checkout → review → cleanup |
| Monitoring | `process log/poll/kill` for running agents |

## Codex modes

| Flag | Behavior |
|------|----------|
| `--full-auto` | Sandboxed, auto-approves file changes (recommended) |
| `--yolo` | No sandbox, no approvals (fast but risky) |

## Configure your assistant to use Codex

By default, your OpenClaw assistant (Claude, etc.) may write code directly instead of delegating to Codex. To make it reach for Codex automatically, add this to your `AGENTS.md` file in your OpenClaw workspace:

```markdown
## Coding Tasks
- For any code building, fixing, refactoring, or PR review, use the codex-agents skill.
- Don't write code yourself — delegate to Codex.
- Default to `codex exec --full-auto` for all tasks.
- Only use `--yolo` if I explicitly ask for it.
- Always verify the output before telling me it's done.
```

Your `AGENTS.md` lives in the workspace directory configured for your OpenClaw agent (usually `~/.openclaw/workspace/` or wherever you pointed it during setup). This file shapes how your assistant behaves — think of it as standing instructions.

## License

MIT
