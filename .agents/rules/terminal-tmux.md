---
trigger: always_on
description: Terminal workflow with tmux, environment selection (main/production), confirmations, dangerous command ban
---

# Terminal Workflow

## Session
- When the user greets me (e.g. "hi", "hello", "salam", "سلام"), FIRST create a NEW detached tmux session with a unique name, every time, by running:
  SESSION="work-$(date +%m%d-%H%M%S)"; tmux new-session -d -s "$SESSION"; echo "$SESSION"
- Never reuse an old session and never use a fixed name like "work". Each new greeting = a new session.
- Tell the user the exact session name and give them the attach command in a code block, for example: tmux attach -t work-1005-141530
- NEVER attach to tmux myself (no "tmux attach", no "new-session -A"). Attaching is interactive and blocks the agent. The user attaches themselves in the terminal panel.
- For the rest of the conversation, use only the session name created in this conversation. Run every command the user asks for with: tmux send-keys -t <SESSION> '<command>' Enter
- Read the results with: tmux capture-pane -p -t <SESSION>
- Commands sent via tmux send-keys follow the exact same rules as any other command: environment mode, confirmation requirement, dangerous-command ban, and whitelist. Using tmux must never be a way around them.
- tmux kill-session, tmux kill-server and tmux kill-pane count as dangerous commands. Never run them, give them to the user instead.

## Environment (ask on every greeting)
- Every time the user greets me, right after creating the tmux session, ask which environment we are working in, as selectable options (use the ask-user / multiple-choice question UI if available), with exactly these two options:
  1. main
  2. production
- Do NOT run any command other than the tmux session creation until the user has chosen.
- The chosen environment applies to the whole conversation. A new greeting starts a new session and asks again.
- If the user never answers, or the answer is unclear, treat it as production (the stricter mode).
- **main mode**: behave exactly as described in the Confirmation, Dangerous Commands and Safe Command Whitelist sections below.
- **production mode**:
  - The Safe Command Whitelist is DISABLED. Before running ABSOLUTELY EVERY command, including ls, pwd, cat, echo, git status, docker ps and every other read-only command, ask the user for explicit confirmation and wait for approval.
  - Ask for each command separately. Approval of one command never covers later commands.
  - The Dangerous Commands ban stays fully in force. Those commands are never run, even with approval.
  - Confirmation format (mandatory, every time):
    1. FIRST, in the chat, print the exact command alone inside a fenced code block (```bash ... ```), with nothing else inside the block. Never put the command inline in a sentence or inside the question box.
    2. THEN ask a short confirmation question in Persian in the question box. The question must NOT contain the command itself. Use a short sentence such as: «حالت Production فعال است. آیا اجرای دستور بالا را تایید می‌کنی؟» with the options «بله، اجرا شود» and «خیر، اجرا نشود».
    3. If the command is long or has several parts, add one short Persian line in the chat (outside the code block) explaining what it does.
  - Write all Persian text in the chat as RTL: start the paragraph with a Persian word (not with English, a number, or a symbol), and keep English terms, file names and commands out of the middle of Persian sentences where possible (put them in the code block instead). If an English term is unavoidable, put it in backticks.
  - Remind the user in one short line that production mode is active (for example at the start of the question).

## Confirmation
- In main mode: before running any command that is NOT in the Safe Command Whitelist below, ask the user for explicit confirmation and wait for approval.
- In production mode: before running ANY command, ask for explicit confirmation and wait for approval (see Environment).
- Show the confirmation request at the top of the user's input box.

## Dangerous Commands (NEVER run, even with user approval, in both modes)
- Never execute destructive or irreversible commands under any circumstances, even if the user explicitly approves or insists. Examples:
  rm, rmdir, shred, dd, mkfs, chmod -R, chown -R, git reset --hard, git clean, git push --force, docker system prune, docker rm, docker volume rm, DROP, TRUNCATE, DELETE FROM (SQL), kill -9, pkill, shutdown, reboot.
- This also applies to dangerous commands chained inside other commands (pipes, &&, ;, xargs, sudo, bash -c, scripts).
- Instead, print the exact command in a code block, briefly explain what it does and its risk, and ask the user to run it manually themselves.

## Safe Command Whitelist (main mode ONLY; run WITHOUT asking for confirmation)
- This whitelist applies only in main mode. In production mode it is ignored and every command needs confirmation.
- ls, pwd, cd, cat, head, tail, less, echo, whoami, date, which, tree
- git status, git log, git diff, git branch
- docker ps, docker logs, docker compose ps
- (add more commands here as needed)
- A whitelisted command is safe only if it is used read-only. If it is combined with a non-whitelisted or dangerous command, treat the whole thing as non-whitelisted.