# Glossary

Tufts / OnDemand / Slurm terms that will trip you up the first month.

---

**OOD / Open OnDemand** — Open-source web portal that gives users a browser-based interface to an HPC cluster. Tufts runs it at <https://ondemand-p01.pax.tufts.edu>.

**pax** — The name of the Tufts research cluster (formerly "tufts hpc"). Login node is `login-prod.pax.tufts.edu`.

**UTLN** — Tufts Username, e.g., `jlavea01`. Used for SSH, OnDemand login, etc.

**Sandbox app** — Any directory under `/cluster/home/<utln>/ondemand/prod/`. OnDemand auto-discovers them and shows them as tiles in your personal dashboard's "Sandbox Apps" section. Production deployments live elsewhere; sandbox is your private copy for testing.

**Batch Connect app** — A flavor of OOD app that *launches* an interactive Slurm job (Jupyter, Ansys, RStudio). Comprises `form.yml.erb` (the launch form), `submit.yml.erb` (Slurm sbatch params), and a `template/` directory (the script that runs inside the job). Examples here: `javi_jupyter`, `ansys`, `igv`.

**Passenger app** — A flavor of OOD app that runs a long-lived web service (Sinatra/Rails/Flask) per user, accessible at `/pun/sys/<app>` or `/pun/dev/<app>`. Used for dashboards and tools, not for launching jobs. Examples here: `tufts-jobmonitor`, `tufts-systemstatus`, `OpenComposer`.

**`/pun/sys/...` vs `/pun/dev/...`** — `sys` is the production-style cached path. `dev` re-renders from source on every request — use it while iterating.

**ERB** — Embedded Ruby. The templating language OOD uses (`form.yml.erb`, `view.html.erb`). Tags: `<% ... %>` runs Ruby; `<%= ... %>` runs Ruby and inserts the result.

**Slurm** — The cluster's job scheduler. The CLI tools you'll shell out to constantly:
- `sinfo` — partition / node state.
- `squeue` — job queue.
- `sacct` — historical job accounting.
- `scontrol show <thing>` — detailed info on jobs/nodes/partitions/reservations.
- `sbatch` — submit a batch job.
- `scancel` — kill a job.
- `seff` — efficiency stats for a finished job.

**Partition** — A Slurm-defined pool of nodes (e.g., `gpu`, `batch`, `largemem`). Different partitions have different limits, hardware, and access policies.

**GRES** (Generic Resource) — Slurm's way of representing GPUs and other "countable" resources. Requested as `--gres=gpu:a100:1`.

**Constraint** — Slurm node-feature filter, e.g., `--constraint=a100-40G`. Often paired with `--gres` to pin to a specific GPU memory size.

**`MaxMemPerNode` vs physical memory** — The partition can cap memory below what's physically on the node. Always honor the partition cap (the dynamic-discovery code does this).

**`CfgTRES`** — Configured trackable resources on a node (string like `cpu=128,mem=1008G,gres/gpu=4`). Source of truth for the partial.

**Dashboard iframe** — The pattern (in `tufts-jobmonitor`, `tufts-systemstatus`) where the OOD dashboard embeds a passenger app via an `<iframe>` so the app appears as a panel directly inside the dashboard rather than its own page.

**Duo push** — The 2FA prompt you'll get on every Tufts login. Approve on your phone.

**Tufts VPN** — Required to reach `*.pax.tufts.edu` and `ondemand-p01.pax.tufts.edu` from off-campus. Install at <https://access.tufts.edu/vpn>.

---

**Codex CLI** — OpenAI's terminal coding agent. `npm install -g @openai/codex`.

**Claude Code** — Anthropic's terminal coding agent. `npm install -g @anthropic-ai/claude-code`.

**MCP** — Model Context Protocol. The plugin system both Codex and Claude Code use to add tools (Playwright, GitHub, file servers, etc.).

**Context rot** — Quality degradation that happens when a single chat session's context window fills past ~50%. The fix is `summary.md` + new session. See [02_ai_agents.md](02_ai_agents.md).

**OAuth (vs API)** — In Codex/Claude, "Sign in with my subscription" (OAuth) bills against your $20/mo plan quota. "Use API key" bills per token, much more expensive. Use OAuth.
