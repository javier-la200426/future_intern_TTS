# OpenComposer — the meta-app

**Local path (after you clone):** `/cluster/home/<your-utln>/ondemand/prod/OpenComposer/`
**Repo:** `git@github.com:TuftsRT/OpenComposer.git` (forked from `RIKEN-RCCS/OpenComposer`)

A 3rd-party OOD framework I forked into TuftsRT. Unlike single-purpose Batch Connect apps (`javi_jupyter`, `ansys`), OpenComposer is a **meta-app** hosting multiple sub-apps under a shared Sinatra backend, with its own widget engine.

This directory also holds **`TESTING_REPORT.md`**, the canonical write-up of the no-Ruby-on-the-cluster workaround referenced in [../03_testing.md](../03_testing.md).

---

## How it differs from a sandbox app

| | `javi_jupyter` | OpenComposer |
|---|---|---|
| Launches | One interactive Slurm job | Multiple sub-apps (`apps/GPU`, `apps/CPU`, `apps/alphafold3`) |
| Backend | OOD's batch_connect dispatcher | Sinatra (`run.rb`) |
| Form engine | YAML widgets via OOD core | Custom (`lib/form.rb` + `public/form.js`, with `init_dw`/`exec_dw` JS) |
| Schedulers | Slurm only | Slurm + PBS + GE + Fujitsu TCS (`lib/schedulers/`) |
| Add a new app | New top-level dir under `ondemand/prod/` | New dir under `OpenComposer/apps/` with its own `form.yml` |

---

## What I contributed (theme: porting the dynamic pattern)

All Tufts commits port the `javi_jupyter` dynamic-discovery pattern into OpenComposer's framework:

- `50b269e` — Add dynamic SLURM discovery to GPU app (2D partition+GPU lookups, gres translation)
- `abfc9ac` — Add dynamic discovery to CPU app, filter out GPU-only partitions
- `4aa0972` — Extract shared helpers into `partials/slurm_discovery.erb`
- `84d885b` / `4c40abf` / `821a6cb` — Remove hardcoded GPU label map; use raw Slurm GPU labels; unify A100 variants
- `15fdb7e` — Adopt `TuftsRT/ood-resource-discovery` improvements

If your task is "add X to an OpenComposer sub-app and make it dynamic," these commits are your reading list.

---

## TESTING_REPORT.md — the no-Ruby workaround

Login nodes lack Ruby in `$PATH`. I extracted a portable Ruby + `psych`/`json` gems into a `~/tmp-ruby/` directory; sourcing a few env vars lets you `ruby -e '...'` and render ERB partials standalone. Use case: render `partials/slurm_discovery.erb` via `ERB.new(src, trim_mode: "-").result(binding)` to validate Slurm queries without going through OOD.

You won't have my `tmp-ruby/`. Set up your own — full recipe in [../03_testing.md](../03_testing.md). Then read the actual `TESTING_REPORT.md` in this repo for the troubleshooting log.

---

## TESTING.md — the two-layer philosophy

Two-layer testing:

1. **Terminal ERB rendering** (using portable Ruby) catches backend logic / Ruby exceptions fast.
2. **Browser testing** at `/pun/dev/OpenComposer/{GPU,CPU}` catches JS timing/ordering bugs the backend can't see (e.g., `updateArea` overwriting 2D overrides).

Neither layer alone is sufficient. And **both** can pass while real `sbatch` still fails — usually a `gres` name mismatch. Always test layer 3 too.

---

## CLAUDE.md — gotchas (useful for humans too)

- ERB files evaluate under `run.rb`'s binding, so `__FILE__` is `run.rb`. Use `./` paths from repo root, not `File.dirname(__FILE__)`.
- Slurm `gres` names differ from internal IDs in OC's config (`a100_40` is internal; submission uses `--gres=gpu:a100 --constraint=a100-40G`). Mismatches are silent at form-render time and explode at `sbatch`.
- Script-template syntax: `#{key}` hides the line if value is empty; `#{:key}` keeps it. Easy to confuse.
- 2D JS overrides must `setTimeout(0)` to run after OC's synchronous 1D `set-max-*` directives, or the override gets clobbered.

Read `CLAUDE.md` first before touching anything inside OpenComposer.

---

## Read in this order

1. `CLAUDE.md`
2. `TESTING_REPORT.md`
3. `TESTING.md`
4. `CHANGES-dynamic-slurm-gpu.md`
5. `run.rb` (line ~60, `read_yaml`) — see how ERB evaluation is wired
6. `partials/slurm_discovery.erb`
7. `apps/GPU/form.yml.erb`
8. `lib/form.rb` and `public/form.js`
9. `lib/schedulers/` (only if you target non-Slurm)

---

## Long-term direction

`OpenComposer/partials/slurm_discovery.erb` should eventually be replaced by including the shared `TuftsRT/ood-resource-discovery` widget. Commit `15fdb7e` was the first step.
