# prompt-terminal

> **AI Terminal Co-Pilot Rules for SREs and DevOps Engineers**  
> A set of prompt-engineering rules that turn your AI coding assistant into a safe, human-in-the-loop terminal co-pilot — no blind command execution, no accidental production disasters.

---

## Why this exists

AI agents connected to a shell are powerful — and dangerous. Without guardrails, an AI can silently run a `DROP TABLE`, delete a directory, or leak secrets to a chat window. This repository defines a minimal, opinionated ruleset that changes that behaviour:

- Every command is shown to the engineer **before** it runs.
- Destructive commands are **never** executed automatically — not even if you say "yes".
- Production environments get an extra layer: **every** command, including read-only ones, requires explicit one-by-one approval.
- Sensitive output (env vars, tokens, logs) is surfaced intentionally, not accidentally.

The rules are plain Markdown files that you drop into your project. No code, no dependencies.

---

## Who it is for

| Role | How it helps |
|------|-------------|
| **SRE** | Safely query Kubernetes, read logs, tail metrics — in production — without fear of a fat-finger incident. |
| **DevOps engineer** | Run deployment pipelines, Docker operations, and git workflows through AI assistance with a full human approval gate. |
| **Platform engineer** | Enforce a consistent "confirm before run" culture across the team when pairing with AI agents on infra work. |

---

## How it works

### 1 · Session isolation via tmux

On every new conversation the agent creates a **fresh, uniquely-named tmux session** (`work-MMDD-HHMMSS`). All commands run inside that session. Sessions are never reused across conversations, which prevents context bleed between unrelated tasks.

```
SESSION="work-$(date +%m%d-%H%M%S)"
tmux new-session -d -s "$SESSION"
# → e.g. work-1011-143000
```

The agent never attaches to tmux itself. You attach in your own terminal:

```bash
tmux attach -t work-1011-143000
```

### 2 · Environment selection

At the start of every conversation you pick the operating context:

| Mode | Behaviour |
|------|-----------|
| **main** | Read-only commands on the whitelist run without confirmation. Everything else needs one approval. |
| **production** | The whitelist is disabled. **Every** command — `ls`, `cat`, `git status`, everything — requires its own explicit approval before the agent touches the shell. |

If you never answer, the agent defaults to **production** (the stricter mode).

### 3 · Command display + 1-click approval

Before any command runs, the agent:

1. Prints the **exact command** in a syntax-highlighted code block in chat.
2. Shows an interactive confirmation prompt with **بله، اجرا شود** / **خیر، اجرا نشود** options (single click, no typing required).
3. In production mode, adds a one-line Persian explanation of what the command does and why.

Approval of one command **never** covers the next one.

### 4 · Absolute ban on destructive commands

The following commands are **hardcoded off** — the agent will refuse to run them under any circumstances, even with explicit user approval:

```
rm  rmdir  shred  dd  mkfs
chmod -R  chown -R
git reset --hard  git clean  git push --force
docker system prune  docker rm  docker volume rm
DROP  TRUNCATE  DELETE FROM   (SQL)
kill -9  pkill  shutdown  reboot
```

This also covers these commands when they appear **inside** pipes, `&&`, `;`, `xargs`, `sudo`, `bash -c`, or shell scripts.

If a destructive operation is genuinely needed, the agent:
1. Reminds you to take a backup first and provides the exact backup command.
2. Explains the risk in plain language.
3. Hands you the command to run **yourself** — it never executes it.

### 5 · Production secret safety

The agent is instructed to treat any command that might print secrets, tokens, or credentials (env dumps, log tails with auth headers, etc.) as requiring explicit confirmation in both modes — so sensitive values are never inadvertently echoed into the chat window.

---

## Repository layout

```
.
├── .agents/
│   └── rules/
│       └── terminal-tmux.md        # Core rule: tmux sessions, env modes,
│                                   # whitelist, dangerous-command ban
└── .gemini/
    └── rules/
        ├── terminal_workflow.md    # Co-pilot workflow & destructive-op policy
        └── command_approval.md     # Command display format & 1-click confirm UX
```

---

## Getting started

1. Copy the `.agents/` and `.gemini/` directories into the root of your project (or your `$HOME` for global effect).
2. Open the project in an AI coding assistant that supports agent rules (e.g. Antigravity / Gemini CLI).
3. Say **hi** — the agent will create a tmux session and ask which environment you're working in.
4. Start issuing natural-language terminal commands. Every one will be shown to you for approval before execution.

> **No code changes needed.** These are pure prompt-engineering rules in Markdown.

---

## Safe command whitelist (main mode only)

The following commands run **without** a confirmation prompt in `main` mode because they are read-only by nature:

```
ls  pwd  cd  cat  head  tail  less  echo  whoami  date  which  tree
git status  git log  git diff  git branch
docker ps  docker logs  docker compose ps
```

A whitelisted command loses its safe status the moment it is chained with a non-whitelisted or destructive command.

---

## Security guarantees

| Guarantee | How it is enforced |
|-----------|-------------------|
| No silent command execution | Every command requires at minimum a confirmation prompt before the agent touches the shell |
| No destructive auto-execution | Hardcoded ban — cannot be overridden by user approval |
| Production isolation | All whitelists disabled; every command individually gated |
| Secret leakage prevention | Commands that may print credentials require explicit approval and are flagged |
| Session isolation | Unique tmux session per conversation; no cross-session contamination |
| Backup before destruction | Mandatory backup step must be surfaced before any destructive instruction is handed to the user |

---

## License

MIT
