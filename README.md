# Future Intern Guide — Tufts OnDemand work

A survival guide for whoever picks up this work next, written by Javier (the previous intern). The work itself: build, modify, and test web apps inside the Tufts Open OnDemand portal — some are launch forms (Jupyter, Ansys), some are dashboards (job monitor, system status), some are framework-level (OpenComposer).

## Read in this order

1. **[01_setup.md](01_setup.md)** — VPN, SSH key, VS Code + SFTP. Required before anything else.
2. **[02_ai_agents.md](02_ai_agents.md)** — Codex CLI, Claude Code, OAuth (not API), context rot, the `summary.md` handoff.
3. **[03_testing.md](03_testing.md)** — Three layers of testing, Playwright MCP, debug flags, the no-Ruby-on-the-cluster workaround.
4. **[04_git_workflow.md](04_git_workflow.md)** — SSH key for GitHub, branches, PRs, multi-account `~/.ssh/config`.
5. **[05_models_and_tools.md](05_models_and_tools.md)** — Model picks, reading `/status`, what to evaluate next.
6. **[06_projects/](06_projects/)** — One file per app. **Start with `javi_jupyter.md`** — it's the foundation that every other dynamic launch form copies.

Also: **[GLOSSARY.md](GLOSSARY.md)** for OnDemand / Slurm / Tufts jargon.

## The single most important habit

**Test every change before you say it works.** AI agents are confident liars; a green diff is not a green test. Most of my headaches came from skipping this step.

## Quick context

- SSH into a login node like `login-prod.pax.tufts.edu` (the "pax" cluster).
- Your home there: `/cluster/home/<your-utln>`.
- OnDemand portal: <https://ondemand-p01.pax.tufts.edu>.
- Anything you put under `/cluster/home/<your-utln>/ondemand/prod/<app>/` shows up automatically as a sandbox app in your personal OnDemand dashboard. This is your test surface.
- Production apps for all Tufts users are deployed elsewhere by the RT team from the `TuftsRT` GitHub org.

— Javier
