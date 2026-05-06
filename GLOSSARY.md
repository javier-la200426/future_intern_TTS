# Glossary

Tufts / OnDemand / Slurm terms that will trip you up the first month.

---

**OOD / Open OnDemand** — Open-source web portal that gives users a browser interface to an HPC cluster. Tufts: <https://ondemand-p01.pax.tufts.edu>.

**pax** — Name of the Tufts research cluster. Login: `login-prod.pax.tufts.edu`.

**UTLN** — Tufts Username (e.g., `jdoe01`). Used for SSH, OnDemand login.

**Sandbox app** — Any directory under `/cluster/home/<utln>/ondemand/prod/`. OOD auto-discovers them and shows them in your dashboard's "Sandbox Apps" section. Production deployments live elsewhere.

**Batch Connect app** — OOD app that *launches* an interactive Slurm job (Jupyter, Ansys, RStudio). Has `form.yml.erb` + `submit.yml.erb` + `template/`. Examples: `javi_jupyter`, `ansys`, `igv`.

**Passenger app** — OOD app that runs a long-lived web service per user, accessible at `/pun/sys/<app>` or `/pun/dev/<app>`. Used for dashboards. Examples: `tufts-jobmonitor`, `tufts-systemstatus`, `OpenComposer`.

**`/pun/sys/...` vs `/pun/dev/...`** — `sys` is production-style cached. `dev` re-renders from source on every request — use it while iterating.

**ERB** — Embedded Ruby templating. `<% ... %>` runs Ruby; `<%= ... %>` runs Ruby and inserts the result.

**Slurm** — The cluster's job scheduler. CLI tools you'll shell out to: `sinfo` (partitions/nodes), `squeue` (queue), `sacct` (history), `scontrol show <thing>`, `sbatch` (submit), `scancel` (kill), `seff` (efficiency).

**Partition** — Slurm-defined pool of nodes (`gpu`, `batch`, `largemem`). Different limits, hardware, access policies.

**GRES** (Generic Resource) — Slurm's GPU/etc. abstraction. Requested as `--gres=gpu:a100:1`.

**Constraint** — Slurm node-feature filter (`--constraint=a100-40G`). Often paired with `--gres` to pin to a specific GPU memory size.

**`MaxMemPerNode`** — Partition can cap memory below physical RAM. Always honor the cap.

**`CfgTRES`** — Configured trackable resources on a node (`cpu=128,mem=1008G,gres/gpu=4`). Source of truth for the discovery partial.

**Dashboard iframe** — Pattern (in `tufts-jobmonitor`, `tufts-systemstatus`) where the OOD dashboard embeds a passenger app via `<iframe>` so it appears as a panel in the dashboard rather than a separate page.

**Duo push** — 2FA prompt on every Tufts login.

**Tufts VPN** — Required to reach `*.pax.tufts.edu` from off-campus. <https://access.tufts.edu/vpn>.

---

**Codex CLI** — OpenAI terminal coding agent. `npm install -g @openai/codex`.

**Claude Code** — Anthropic terminal coding agent. `npm install -g @anthropic-ai/claude-code`.

**MCP** — Model Context Protocol. Plugin system both CLIs use to add tools (Playwright, GitHub, etc.).

**Context rot** — Quality degradation when a session's context window passes ~50%. Fix: `summary.md` + new session.

**OAuth (vs API)** — Sign in with subscription (cheap, quota-based) vs. API key (per-token, expensive). Use OAuth.
