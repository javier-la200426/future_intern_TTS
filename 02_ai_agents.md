# 02 — AI agents (Codex, Claude Code, MCPs)

Get an AI agent set up before you write a single line of code. It is the difference between a one-week change and an afternoon.

---

## 1. Pay $20/month for ChatGPT, install Codex CLI

The Codex CLI is the cheapest start: $20/month ChatGPT Plus includes Codex usage on a 5-hour rolling session quota. The same workload via OpenAI's API would cost much more — **sign in with OAuth, not an API key**.

```bash
npm install -g @openai/codex
```

If npm isn't installed, ask any LLM to walk you through installing it on your OS — the steps drift, so I won't pin them. You'll install Codex on both your laptop and the cluster (use `nvm` on the cluster since you can't `sudo`).

Run `codex` once, choose "Sign in with ChatGPT" (the OAuth option), log in, done.

### Commands you'll use daily

- `/status` — shows (a) **session quota** + reset time, and (b) **context window** usage in the current conversation. Check often.
- `/permissions` — set to "full access" so it stops asking permission for every shell command. Safe in your sandbox/cluster home; never on machines you wouldn't trust the agent to overwrite.
- `/new` — fresh session.

### Watch your context window

When `/status` shows ~50% context used, **start a new session**. Past 50% the model drifts and forgets earlier instructions ("context rot"). The handoff trick:

1. Ask the current session: *"Write `summary.md` describing what we worked on, what's done, what's left, files touched, testing tricks discovered."*
2. New session, first message: *"Read summary.md and continue."*

Works astonishingly well. Use it 2–3 times a day on long tasks.

---

## 2. Optional: Claude Code alongside

Same install pattern (`npm install -g @anthropic-ai/claude-code`), same OAuth sign-in (Claude Pro/Max subscription).

- Claude Code currently hits its session limit **faster** than Codex on equivalent work.
- Claude is currently better at **frontend/UI/CSS**; Codex is better at long, methodical refactors.
- `claude --dangerously-skip-permissions` is the equivalent of Codex's full-access mode.

You don't need Claude on day one. Add it when you want a second opinion or hit Codex's quota.

---

## 3. Install MCP servers

MCP servers are agent plugins. The two that matter:

### Playwright MCP — install on your laptop

Lets the AI drive a real browser: click in the OnDemand portal, take screenshots, run JS in the page console, read console logs. Critical for testing.

```bash
codex mcp add playwright npx "@playwright/mcp@latest"
# or
claude mcp add playwright npx "@playwright/mcp@latest"
```

Don't install on the cluster — it has no display. Full Playwright workflow is in **[03_testing.md](03_testing.md)**.

### Worth evaluating

A newer "agent browser" MCP claims to be more token-efficient than Playwright (which returns chunky DOM snapshots). If true, you get more turns per session. I haven't switched.

---

## 4. The "save what you learned" habit

When you finish a chunk of work, ask the agent:

> *"Summarize what you learned: tools available, testing techniques that worked, gotchas, files that matter. Save it to a `LESSONS.md` in the project directory. Add to `.gitignore` if it's too rough to commit."*

The next session reads that file and skips rediscovering everything. Examples already in the repos: `OpenComposer/TESTING_REPORT.md`, `OpenComposer/CLAUDE.md`, `ansys/PERMISSIONS_TRAP.md`, `ansys/WHY_GPU_ONLY.md`. By month three you'll have a personal library of "things AI doesn't have to figure out twice."

---

## Cheat sheet

| Codex | Claude Code | What |
|---|---|---|
| `/status` | `/status` | Session + context usage |
| `/permissions` | `--dangerously-skip-permissions` | Stop permission prompts |
| `/new` | `/clear` | Fresh session |
| `/mcp` | `/mcp` | List installed MCPs |
| `codex resume` | `claude --resume` | Restart last session |

Move on to **[03_testing.md](03_testing.md)**.
