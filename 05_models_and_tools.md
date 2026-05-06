# 05 — Picking models, watching limits

The model landscape moves fast. Treat this chapter as principles, not recipe.

---

## Current defaults

- **Codex CLI** — GPT-5.4. Strong all-around for refactors, careful multi-file edits, methodical reasoning.
- **Claude Code** — Opus 4.7 (or 4.6 in fast mode). Strong for UI/CSS, frontend, prose.

When one hits its 5-hour quota, switch to the other. Re-evaluate every couple of months — earlier in the year I used Sonnet heavily for UI; Opus 4.7 mostly replaced it.

---

## Reading `/status`

**Session quota (5-hour rolling window):**
- <50%: keep going.
- 50–80%: budget what's left.
- 80–100%: finish what you're on, then break or switch CLIs.

**Context window (current conversation):**
- <50%: fine.
- At 50%: start a new session via the `summary.md` handoff (chapter 02).
- >50%: model drifts and forgets earlier instructions (context rot). Quality degrades; not a hard cliff but noticeable.

Quota and context are independent.

---

## Tools to install on every new agent setup

1. **Playwright MCP** — non-negotiable for OnDemand work (chapter 03).
2. Codex/Claude on **both** laptop and cluster. Cluster does Slurm + ERB. Laptop does Playwright.
3. **`gh` CLI** — lets agents create PRs, leave review comments, fetch issues without copy-paste.
4. Worth evaluating: a newer "agent browser" MCP that's reportedly more token-efficient than Playwright.

---

## Anti-patterns I fell into

- **Trusting one model's confidence.** If GPT says "this should work" and the test fails, ask Claude (or vice versa). They make different mistakes.
- **Letting context hit 80% before summarizing.** The summary itself becomes degraded. Summarize at 50%.
- **Switching CLIs mid-task without re-grounding.** Always start the new agent with "read CLAUDE.md / summary.md and tell me what you understand before doing anything."
- **Paying via API "just to try it."** API pricing dwarfs subscription pricing for daily use.

Move on to **[06_projects/](06_projects/)**.
