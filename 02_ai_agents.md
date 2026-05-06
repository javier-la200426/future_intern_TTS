# 02 — AI agents (Codex, Claude Code, MCPs)

This is the chapter I most wish I had on day one. **Get an AI agent set up before you write a single line of code.** It is the difference between making a one-week change in a week and making it in an afternoon.

---

## Step 1 — Pay $20/month for ChatGPT, install Codex CLI

The Codex CLI is OpenAI's terminal-based coding agent. It is the cheapest way to start because:

- A $20/month ChatGPT Plus subscription includes Codex usage on a generous **5-hour rolling window**, billed by *session quota*, not per token.
- The same $20 quota would cost vastly more if you used the OpenAI API directly. **Do not pay via API**, sign in with OAuth.

### Install

```bash
npm install -g @openai/codex
```

If npm is not installed, ask any LLM (ChatGPT web, Claude web) "how do I install npm on macOS" / "...on RHEL 9" — the steps drift, so I won't pin them. You'll install it twice, once on your laptop and once on the cluster (via `nvm` is easiest on the cluster since you can't `sudo`).

If the install errors, paste the error into ChatGPT and iterate. Don't get stuck — there is nothing magic about it.

### Sign in with OAuth (do this, not API keys)

After install, run `codex` and choose "Sign in with ChatGPT" (the OAuth option). It opens a browser, you log in to your Plus account, and the CLI now consumes your subscription quota instead of API credits.

### Codex commands you will use daily

- `/status` — shows two crucial numbers: (a) **session quota usage** and time until reset (the 5-hour window), and (b) **context window usage** for the current conversation. Check this often.
- `/permissions` — set this to "full access" so it stops asking permission for every shell command. Only safe to do this in a sandbox/cluster home — never on a machine where you don't trust the agent to overwrite things.
- `/new` — start a fresh session (resets context, NOT quota).

### Watch your context window

When `/status` shows you've used **~50% of the context window**, start a new session. After 50% the model starts to drift and forget earlier instructions ("context rot"). The fix:

1. Ask the current session: *"Write a `summary.md` file describing what we worked on, what's done, what's left, what files we touched, and any testing tricks you discovered."*
2. Start a new session. First message: *"Read summary.md and continue."*

This handoff pattern works astonishingly well. I do it 2-3 times a day on big tasks.

---

## Step 2 — Optional: Claude Code in addition

I run Claude Code (Anthropic's competitor to Codex) alongside Codex and switch between them. Same install pattern (`npm install -g @anthropic-ai/claude-code`), same OAuth sign-in (need a Claude Pro / Max subscription), same general feel. Differences in practice:

- Claude Code hits its session limit **faster** than Codex on equivalent work — burn through Claude first, then fall back to Codex.
- Claude is currently better at **frontend / UI / CSS** work in my experience. Codex is currently better at long, methodical, careful refactors.
- Useful flag: `claude --dangerously-skip-permissions` — same idea as Codex's `/permissions` full-access. Same caution.

You don't *have* to install Claude Code at first. Start with Codex, get comfortable, and add Claude later when you want a second opinion or hit Codex's quota.

---

## Step 3 — Run agents on the cluster too

Install Codex (and optionally Claude Code) on the cluster as well. You'll usually want one agent running on the cluster (where it can read files, run Slurm commands, edit ERB) and one on your laptop (where it can drive the browser via Playwright). They communicate by you copying prompts between terminals.

Cluster install: same npm command. You may need to set up `nvm` first since the system Node is too old. Ask the AI on the cluster terminal to bootstrap it for you:

```bash
codex
> Bootstrap nvm + Node 20 + Codex CLI for me on this RHEL 9 login node, no sudo available.
```

---

## Step 4 — Install MCP servers

MCP (Model Context Protocol) servers give your agent extra capabilities. The two I rely on:

### Playwright MCP (the big one)

This lets the AI control a real browser — click buttons in your OnDemand portal, take screenshots, run JS in the page console, read console logs. Critical for testing OnDemand apps because the only way to know "did my form change actually work" is to render the form in a browser.

Install on your **laptop's** Codex (not the cluster — the cluster has no display):

```bash
codex mcp add playwright npx "@playwright/mcp@latest"
```

For Claude Code:

```bash
claude mcp add playwright npx "@playwright/mcp@latest"
```

After adding, restart the agent. It will now have tools like `browser_navigate`, `browser_click`, `browser_screenshot`, `browser_evaluate` (run JS), `browser_console_messages` (read logs).

The full Playwright workflow lives in **[03_testing.md](03_testing.md)**.

### "Agent Browser" alternative (worth exploring)

There's a newer browser-control MCP marketed as more token-efficient than Playwright. I haven't switched to it but you should evaluate it — Playwright eats tokens because it returns DOM snapshots. If a task is feasible in fewer tokens, that means more turns before you hit the session limit.

---

## Step 5 — The "save what you learned" habit

When you finish a chunk of work, ask Codex / Claude:

> *"Summarize what you learned in this session: what tools were available, what testing techniques worked, gotchas you ran into, and which files matter. Save it to a `LESSONS.md` (or `TESTING_REPORT.md`) in the project directory. I'll add it to `.gitignore` if it's too rough to commit."*

The next session will read this file and skip rediscovering everything. Examples of this in the repo:

- `OpenComposer/TESTING_REPORT.md` — how to run Ruby on the cluster when Ruby isn't installed.
- `OpenComposer/CLAUDE.md` — architectural gotchas for AI agents working in that repo.
- `ansys/PERMISSIONS_TRAP.md` and `ansys/WHY_GPU_ONLY.md` — debugging stories captured as docs.

These files compound. By month three you will have a personal library of "things AI doesn't have to figure out twice."

---

## Cheat sheet — agent commands I use most

| Codex | Claude Code | What |
|---|---|---|
| `/status` | `/status` | Session + context usage |
| `/permissions` | `--dangerously-skip-permissions` | Stop asking permission |
| `/new` | `/clear` | Fresh session |
| `/mcp` | `/mcp` | List installed MCP servers |
| `codex resume` | `claude --resume` | Restart your last session |

---

Move on to **[03_testing.md](03_testing.md)**.
