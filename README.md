# openclaw-codex-agents

A guide for setting up [OpenClaw](https://openclaw.ai) to orchestrate [OpenAI Codex](https://github.com/openai/codex) coding agents. Covers two setups: single machine (Mac) and split architecture (VPS + Mac).

## Prerequisites

1. **[OpenClaw](https://docs.openclaw.ai)** — installed and configured with at least one chat channel (Telegram, Discord, etc.)
2. **[OpenAI Codex CLI](https://github.com/openai/codex)** — install on the machine that will run agents
   ```bash
   npm install -g @openai/codex
   ```
3. **Codex authentication** — sign in to your OpenAI account (requires a plan that includes Codex)
   ```bash
   codex auth
   ```
4. **Node.js 18+**
5. **Git** — Codex requires a git repository to run in

## Setup A: Single Machine (Mac)

OpenClaw and Codex run on the same machine. Simplest setup.

### How it works

OpenClaw's built-in `coding-agent` skill handles everything. Your assistant uses `exec` with `pty:true` to launch Codex directly.

```
You (Telegram/Discord) → OpenClaw (Mac) → Codex (Mac) → notifies you when done
```

### Steps

1. Install OpenClaw and Codex on your Mac
2. Add this to your `AGENTS.md` (in your OpenClaw workspace):

```markdown
## Coding Tasks
- For any code building, fixing, refactoring, or PR review, use the coding-agent skill.
- Don't write code yourself — delegate to Codex.
- Default to `codex exec --full-auto` for all tasks.
- Only use `--yolo` if I explicitly ask for it.
- Always verify the output before telling me it's done.
```

3. That's it. Ask your assistant to build something and it'll spawn Codex.

### Example

> "Build a REST API for todos in ~/projects/my-app"

Your assistant will:
1. Spawn Codex in the project directory with `--full-auto`
2. Monitor progress in the background
3. Notify you on completion via `openclaw system event`

---

## Setup B: Split Architecture (VPS + Mac)

OpenClaw runs on a VPS (always-on, handles chat). Codex runs on your Mac (where your code lives). Connected via [Tailscale](https://tailscale.com) + OpenClaw node pairing.

```
You (Telegram/Discord) → OpenClaw (VPS) → nodes.run → Codex (Mac) → notifies VPS → notifies you
```

### Why this setup?

- VPS is always on — no missed messages, crons run 24/7
- Mac has your code, editor, and Codex with full filesystem access
- Tailscale connects them securely without port forwarding

### Steps

1. Install OpenClaw on your VPS (gateway, channels, crons)
2. Install OpenClaw + Codex on your Mac
3. Pair your Mac as a node: `openclaw devices pair` on VPS, approve on Mac
4. Add this to your `AGENTS.md` on the VPS:

```markdown
## Coding Tasks
- All code lives on Mac. Never clone repos to VPS for coding.
- For ANY code changes, launch Codex on Mac via nodes.run.
- Use `nodes action:run node:"<your-mac-name>" command:[...]` for git reads (status, log, diff).
- For Codex tasks, write the prompt to a temp file on Mac, then launch:
  `nohup codex exec --full-auto "$(cat /tmp/task-prompt.txt)" > /tmp/task.log 2>&1 &`
- Codex `--full-auto` sandbox blocks network calls — don't expect `openclaw system event` from inside Codex.
- Check if Codex is alive: `pgrep -f "codex exec"`
- Read logs: `tail -50 /tmp/task.log`

## Node Access
- Read files: `nodes action:run node:"<mac-name>" command:["cat", "/path/to/file"]`
- Git operations: `nodes action:run node:"<mac-name>" cwd:/path/to/project command:["git", "status"]`
- NEVER use nodes.run to write/edit code — use Codex instead.
```

### Limitations of split setup

- **No auto-notification from Codex**: `--full-auto` sandboxes network, so `openclaw system event` won't work from inside a Codex session. Use a cron or task watcher to poll for completion.
- **Gateway timeouts**: Long `nodes.run` commands may timeout, but the process keeps running on Mac. Use `nohup` + background.
- **Node allowlist**: Only approved binaries can run via `nodes.run`. Check your allowlist if commands get denied.

---

## Codex modes

| Flag | Behavior |
|------|----------|
| `exec "prompt"` | One-shot, exits when done |
| `--full-auto` | Sandboxed, auto-approves file changes (recommended) |
| `--yolo` | No sandbox, no approvals (fast but risky) |

## Notify on completion

For single-machine setups, append this to any Codex prompt:

```
When completely finished, run: openclaw system event --text "Done: <summary>" --mode now
```

This triggers an immediate notification to your configured channel.

## License

MIT
