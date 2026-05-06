# Future Intern Guide — Tufts TTS / Research Computing OnDemand Work

Welcome. This is a survival guide written by Javier (the intern who came before you) for whoever picks up this work next. It is not a textbook; it is the set of things I wish someone had handed me on day one so I didn't have to discover them painfully.

The work itself is roughly: **build, modify, and test web apps that run inside the Tufts Open OnDemand portal** (the cluster's web UI for launching jobs and monitoring the cluster). Some apps are launch forms (Jupyter, Ansys, Matlab), some are dashboards (job monitor, system status), and some are framework-level (OpenComposer).

This guide is organized so you can read it linearly on day one, then jump back to the relevant chapter as you start each kind of task.

---

## Read in this order

1. **[01_setup.md](01_setup.md)** — VPN, SSH key, SFTP-with-VS-Code so you can edit files locally and have them auto-sync to the cluster. You cannot work productively until this is done.
2. **[02_ai_agents.md](02_ai_agents.md)** — Codex CLI, Claude Code, why you should pay $20/mo for ChatGPT (and probably another $20 for Claude), OAuth vs API billing, context rot, the `summary.md` handoff trick.
3. **[03_testing.md](03_testing.md)** — How to actually verify your code works: launching apps in OnDemand, the Playwright MCP browser-testing setup, the Ruby-not-installed problem and Javier's portable-Ruby workaround, frontend `?debug=1` and backend `DEBUG_*` flags.
4. **[04_git_workflow.md](04_git_workflow.md)** — Adding your SSH key to GitHub, branches, PRs, the `host` trick for multiple GitHub accounts, what to commit vs `.gitignore`.
5. **[05_models_and_tools.md](05_models_and_tools.md)** — Which model to use when, how to read your session limits, MCPs to install, things to keep an eye on as the field moves.
6. **[06_projects/](06_projects/)** — One file per app I worked on, in rough priority order. **Start with `06_projects/javi_jupyter.md`** — it is the foundation of the dynamic-form pattern that every other launch form copies.

There is also a **[GLOSSARY.md](GLOSSARY.md)** at the root for OnDemand / Slurm / Tufts-specific jargon.

---

## The single most important habit

**Test every change, no matter how small, before you say it works.** AI agents are confident liars; a green diff is not a green test. Most of the headaches I had came from skipping this step. The whole testing chapter exists because of this one rule.

---

## Quick context: what the cluster looks like

- You SSH into a login node like `login-prod.pax.tufts.edu` (alias: pax cluster).
- Your home there is `/cluster/home/<your-utln>`.
- The OnDemand portal lives at `https://ondemand-p01.pax.tufts.edu` and lets users (and you) launch interactive jobs, see files, monitor jobs, etc.
- The "sandbox apps" you build live in `/cluster/home/<your-utln>/ondemand/prod/<app-name>/`. Anything you put there appears automatically in your personal OnDemand dashboard under "Sandbox Apps." This is how you test changes without pushing to production.
- Production apps for all Tufts users live elsewhere (managed by the RT team and deployed from the `TuftsRT` GitHub org). Your sandbox is a private copy.

If any of those sentences were confusing, that's fine — the setup chapter walks through it.

---

## A note on this guide

I wrote the prose; an AI agent helped me organize it and pulled exact details out of the project directories. If something looks stale, trust the code in the project directory over this guide and update the doc.

Good luck. The work is interesting and the people are nice. Ping me if you get stuck.

— Javier
