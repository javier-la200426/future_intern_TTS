# 05 — Picking models, watching limits, what's coming next

The model landscape moves fast. Anything model-specific in this guide will be stale in three months. Treat this chapter as **the principles**, not the recipe.

---

## Models I currently default to

As of this writing, my defaults across the two CLIs are:

- **OpenAI Codex CLI** — GPT-5.4. Strong all-around for refactors, careful multi-file edits, anything where I want methodical reasoning.
- **Claude Code** — Opus 4.7 (or 4.6 in fast mode if I'm impatient). Strong for UI/CSS, frontend changes, "make this look better," and writing prose.

When one hits its 5-hour rate limit, switch to the other. They are good substitutes.

### Earlier in the year

For most of the spring I used **Claude Sonnet** heavily on UI work — it was the best frontend coder available. I've largely stopped reaching for Sonnet now that Opus 4.7 is out, but Sonnet remains a fast cheap option if your task is small.

The lesson: re-evaluate every couple of months. The "best model for X" changes.

---

## Reading your `/status` output

Both Codex and Claude Code have a `/status` slash command. Two things to track:

### Session quota (the 5-hour window)

Tells you: percent of quota used in the current 5-hour rolling window, and timestamp when it resets to 100%.

- Below 50% used: keep going.
- 50–80%: budget the rest carefully. Avoid kicking off anything huge.
- 80–100%: finish what you're on, then take a break or switch CLIs.

When the window resets, you're back to 100%.

### Context window usage

Tells you: percent of the *current conversation*'s context window in use.

- **Below 50%**: fine, keep going.
- **At 50%**: start a new session (the [`summary.md` handoff trick](02_ai_agents.md#step-1--pay-20month-for-chatgpt-install-codex-cli) from chapter 02).
- **Above 50%**: the model starts to forget early instructions and quality drops noticeably. This is **context rot**. It's not a hard cliff — quality just degrades. Don't push past it for important work.

Context window and session quota are separate. You can blow through context (start a new session) without using much quota, and you can hammer quota across many short sessions.

---

## Tools I install on every new agent setup

In rough priority order:

1. **Playwright MCP** — non-negotiable for OnDemand work. See [03_testing.md](03_testing.md).
2. A second browser/automation MCP if you want to compare token cost — there's an "agent browser" MCP that's reportedly more efficient than Playwright. Worth evaluating; I haven't switched.
3. Codex/Claude on **both** your laptop and the cluster. Cluster agent does Slurm + ERB. Laptop agent does Playwright.
4. **`gh` CLI** (GitHub CLI) on both. Lets agents create PRs, leave review comments, fetch issues without you copying URLs around.

---

## What I'd watch for next

- **Cheaper / faster browser MCPs.** The Playwright MCP returns chunky DOM snapshots that eat tokens. Anything that returns smaller representations (accessibility tree only, screenshots only, etc.) extends your session budget.
- **New model releases**, especially anything claiming better tool-use efficiency. The bottleneck is rarely raw intelligence — it's how many tool calls you can make before context fills up.
- **"Agent" frameworks** beyond plain CLIs (custom MCPs, IDE-embedded copilots that auto-orchestrate Codex+Claude). These are early but worth checking quarterly.
- **OnDemand upstream changes.** OOD itself ships new versions periodically. Keep an eye on what changes in the dynamic-form / batch-connect surface area — the dynamic-discovery pattern in `javi_jupyter` could need adjustment.

---

## Anti-patterns I've fallen into

- **Trusting a single model's confidence.** If GPT-5 says "this should work" and the test fails, ask Claude (or vice versa) for a second opinion. They make different mistakes.
- **Letting context get to 80% before summarizing.** The summary itself becomes degraded. Summarize at 50%.
- **Switching models mid-task without re-grounding.** If you switch CLIs, the new agent has no context. Always start with "read CLAUDE.md / summary.md and tell me what you understand before doing anything."
- **Paying via API "just to try it."** API pricing dwarfs subscription pricing for daily use. Subscriptions first.

Move on to **[06_projects/](06_projects/)** for project-by-project notes.
