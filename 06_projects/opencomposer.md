# OpenComposer — the meta-app

**Path:** `/cluster/home/jlavea01/ondemand/prod/OpenComposer/`
**Repo:** `git@github.com:TuftsRT/OpenComposer.git` (forked from `RIKEN-RCCS/OpenComposer`)

OpenComposer is a 3rd-party Open OnDemand framework forked from RIKEN-RCCS. Unlike a single-purpose Batch Connect sandbox app (e.g., `javi_jupyter`, which launches one interactive session), OpenComposer is a **meta-app** that hosts multiple sub-apps under a shared Sinatra backend, with a richer JS/CSS form engine.

This is also the directory that contains **`TESTING_REPORT.md`** — the canonical write-up of the "no Ruby on the cluster" workaround referenced throughout this guide.

---

## How it differs from a sandbox app like `javi_jupyter`

| | `javi_jupyter` (Batch Connect) | OpenComposer (meta-app) |
|---|---|---|
| Launches | One interactive Slurm job | Multiple sub-apps (`apps/GPU`, `apps/CPU`, `apps/alphafold3`) |
| Backend | OOD's batch_connect dispatcher | Sinatra (`run.rb`) |
| Form engine | YAML widgets via OOD core | Custom widget engine (`lib/form.rb` + `public/form.js`, with `data-*` attrs and `init_dw`/`exec_dw` JS) |
| Schedulers | Slurm only | Slurm + PBS + GE + Fujitsu TCS (abstracted in `lib/schedulers/`) |
| Add a new app | New top-level directory under `ondemand/prod/` | Add a directory under `OpenComposer/apps/` with its own `form.yml` |

---

## What I contributed (theme: porting the dynamic-discovery pattern)

Tufts contributions visible in `git log` are all variations on the same theme — bringing the `javi_jupyter` dynamic-Slurm-discovery pattern into OpenComposer's framework:

- `50b269e` — Add dynamic SLURM resource discovery to GPU app (2D partition+GPU lookups, gres translation).
- `abfc9ac` — Add dynamic discovery to CPU app, filter out GPU-only partitions.
- `4aa0972` — Extract shared helpers/fallbacks into `partials/slurm_discovery.erb` for reuse.
- `84d895b`, `4c40abf`, `821a6cb` — Remove hardcoded GPU label map; use raw Slurm GPU labels; unify A100 variants.
- `15fdb7e` — Adopt `TuftsRT/ood-resource-discovery` improvements into OpenComposer (the consolidation step).

If your task is "add X to an OpenComposer sub-app and make it dynamic," these commits are your reading list.

---

## TESTING_REPORT.md — the "no Ruby on the cluster" technique

**This is the most important file in this directory.** Open it: `/cluster/home/jlavea01/ondemand/prod/OpenComposer/TESTING_REPORT.md`.

Login nodes don't have Ruby in `$PATH`, but you constantly need to render ERB and exercise Slurm queries from the terminal. The fix: a portable Ruby extracted from Rocky 9 RPMs, ~30MB, stashed in `javi_jupyter/.tmp-ruby/`. Includes Ruby 3.0.7 + libs + `psych` (YAML) + `json`.

Verbatim env vars to source:

```bash
RUBY=/cluster/home/jlavea01/ondemand/prod/javi_jupyter/.tmp-ruby/root/usr/bin/ruby
LD_LIBRARY_PATH=/cluster/home/jlavea01/ondemand/prod/javi_jupyter/.tmp-ruby/root/usr/lib64
RUBYLIB=...psych-3.3.2/lib:...json-2.5.1/lib   # full paths in TESTING_REPORT.md
```

Run with:

```bash
LD_LIBRARY_PATH="$LD_LIBRARY_PATH" RUBYLIB="$RUBYLIB" "$RUBY" -e '<your script>'
```

The use case: render `partials/slurm_discovery.erb` via `ERB.new(src, trim_mode: "-").result(binding)` to validate your Slurm queries and computed values **without going through the OOD web stack**. Must run on a login node (with Slurm controller access).

> **TODO I left:** move `.tmp-ruby/` from inside `javi_jupyter/` to `/cluster/home/<utln>/ondemand/prod/.tmp-ruby/` so it's app-agnostic. Easy refactor, just paths to update.

The full chapter on testing is in **[../03_testing.md](../03_testing.md)** — this file is just the OpenComposer-specific story.

---

## TESTING.md — the two-layer philosophy

`/cluster/home/jlavea01/ondemand/prod/OpenComposer/TESTING.md` codifies a two-layer testing approach:

1. **Terminal ERB rendering** (using the portable Ruby above) catches backend logic and Ruby exceptions fast.
2. **Browser testing** at `/pun/dev/OpenComposer/{GPU,CPU}` catches JS timing/ordering bugs that the backend can't see (e.g., `updateArea` overwriting 2D overrides because of script execution order).

Neither layer alone is sufficient. And **both** layers can pass while a real `sbatch` submission still fails — usually because of a `gres` name mismatch. Always test layer 3 too (a real job submission).

---

## CLAUDE.md — instructions to AI agents

`/cluster/home/jlavea01/ondemand/prod/OpenComposer/CLAUDE.md` documents architectural quirks specifically for AI agents. The highlights are useful for humans too:

- ERB files evaluate under `run.rb`'s binding, so `__FILE__` is `run.rb`. Use `./` paths from repo root, not `File.dirname(__FILE__)`.
- Slurm gres names differ from internal IDs in OpenComposer's config (`a100_40` is internal; submission uses `--gres=gpu:a100 --constraint=a100-40G`). Mismatches here are silent at form-render time and explode at `sbatch`.
- Script-template syntax in OpenComposer: `#{key}` hides the line if the value is empty; `#{:key}` keeps the line. Easy to confuse.
- 2D JS overrides must `setTimeout(0)` to run after OpenComposer's synchronous 1D `set-max-*` directives, or the override gets clobbered.

If you're modifying anything inside OpenComposer, read `CLAUDE.md` first.

---

## Read in this order

1. `CLAUDE.md` — the gotchas.
2. `TESTING_REPORT.md` — the portable-Ruby setup.
3. `TESTING.md` — the testing philosophy.
4. `CHANGES-dynamic-slurm-gpu.md` — change log of my dynamic-discovery work.
5. `run.rb` (line ~60, the `read_yaml` function) — see how ERB evaluation is wired.
6. `partials/slurm_discovery.erb` — OpenComposer's version of the discovery partial.
7. `apps/GPU/form.yml.erb` — example sub-app using the partial.
8. `lib/form.rb` and `public/form.js` — the custom widget engine.
9. `lib/schedulers/` — multi-scheduler abstraction (only matters if you target non-Slurm).

---

## Where this app sits in the long-term plan

`OpenComposer` was the first big "real" port of the dynamic-discovery pattern beyond a sandbox. The eventual direction is for `OpenComposer/partials/slurm_discovery.erb` to be replaced by including the shared `TuftsRT/ood-resource-discovery` widget. Commit `15fdb7e` was the first step in that direction.
