# never-rm-rf

> **Because your AI shouldn't `rm -rf` your career.**

A minimal set of Markdown rules that turns your AI coding assistant into a safe, human-approved terminal co-pilot. No blind execution. No production disasters. No 3 AM post-mortems.

---

## The problem

You gave your AI access to the shell.

It is very helpful.  
It is also one hallucination away from:

```bash
rm -rf /var/lib/postgresql/data
docker system prune -af
git reset --hard HEAD~50
DROP TABLE users;
```

This repo exists so that never happens.

---

## What it does

**Every command gets shown to you before it runs.**  
You click yes or no.  
That's it.

And a small list of commands — `rm`, `drop`, `truncate`, `git reset --hard`, and friends — **can never be run by the AI at all**, even if you say please.

---

## Two modes. Pick one at the start of every session.

| Mode | Behaviour |
|------|-----------|
| **main** | Read-only commands (`ls`, `cat`, `git status` …) run freely. Everything else needs your approval. |
| **production** | Nothing runs without your explicit approval. Not even `ls`. Every command. One by one. |

If you forget to pick, it defaults to **production**. Because that's the safer assumption.

---

## The permanently banned list

The AI will never execute these. Not if you ask nicely. Not if you insist. Not ever.

```
rm  rmdir  shred  dd  mkfs
chmod -R  chown -R
git reset --hard  git clean  git push --force
docker system prune  docker rm  docker volume rm
DROP  TRUNCATE  DELETE FROM
kill -9  pkill  shutdown  reboot
```

If you genuinely need one of these, the AI will:
1. Tell you to take a backup first (and give you the exact command).
2. Explain what will happen and why it's risky.
3. Print the command in a code block for you to run yourself — manually, in your own terminal.

---

## Session isolation

Every conversation gets a fresh tmux session with a unique name:

```bash
# Created automatically on greeting
work-1011-143000

# You attach yourself
tmux attach -t work-1011-143000
```

The AI never reuses sessions. It never attaches itself. Sessions from yesterday's incident don't bleed into today's work.

---

## Secret safety

In production mode, any command that might print tokens, passwords, or credentials to the chat window requires explicit approval first — so your secrets don't end up in a chat log.

---

## How to use it

1. Copy `.agents/` and `.gemini/` into the root of your project.
2. Open it with an AI assistant that supports agent rules (Antigravity, Gemini CLI, etc.).
3. Say hi. The AI creates a tmux session and asks: **main or production?**
4. Start talking. Every command will be shown to you before it runs.

No code. No dependencies. Just Markdown files.

---

## File layout

```
.
├── .agents/
│   └── rules/
│       └── terminal-tmux.md        # Sessions, env modes, whitelist, banned commands
└── .gemini/
    └── rules/
        ├── terminal_workflow.md    # Co-pilot behaviour & destructive-op policy
        └── command_approval.md     # Command display format & 1-click confirm UX
```

---

## Safe command whitelist (main mode)

These run without asking. They're read-only by definition:

```
ls  pwd  cd  cat  head  tail  less  echo  whoami  date  which  tree
git status  git log  git diff  git branch
docker ps  docker logs  docker compose ps
```

Chain any of these with a non-safe command and the whole thing needs approval.

---

## License

MIT
