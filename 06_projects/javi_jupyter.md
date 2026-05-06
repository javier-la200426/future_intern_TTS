# javi_jupyter — the foundation

**Local path (after you clone):** `/cluster/home/<your-utln>/ondemand/prod/javi_jupyter/`
**Repo:** `git@github.com:javier-la200426/jupyterNotebookDynamicFormYml.git`

The foundation app for everything else. The dynamic Slurm-aware launch-form pattern was prototyped here. Every other launch form (`ansys`, `igv`, OpenComposer's apps) copies from it.

---

## What the "dynamic form" pattern does

OOD Batch Connect apps normally hardcode their launch-form choices in static YAML — same options for every user, regardless of access or current node state.

The dynamic pattern replaces those static lists with values **discovered live from Slurm at form-render time, scoped to the logged-in user**. When the user opens the form, the server runs `sinfo`/`scontrol` *as that user*, and emits a form whose dropdowns and `max=` values reflect what they can actually request right now. `form.js` re-runs the same lookups in the browser whenever partition or GPU type changes.

---

## How `partials/gpu_discovery.erb` works

1. Shells out: `scontrol show node --oneliner`, `sinfo -o "%P %l"`, `scontrol show partition --oneliner`, `sinfo` (no `--all`, so Slurm filters to user-accessible partitions).
2. Builds Ruby hashes: partition→GPU types, (partition, gpu)→max cores/memory, partition→max hours, plus an "unavailable GPUs" set (matching nodes all DOWN/DRAIN/MAINT).
3. Defines helper methods (`generate_partition_field`, `generate_num_cores_field`, etc.) that emit YAML field definitions.
4. Serializes hashes to JSON in `data-*` HTML attributes (`data-partition-gpu-max-cores`, etc.).
5. `form.js` reads those blobs, listens for `change` on partition/GPU, rewrites dropdowns, sets `max`/help text, shows a red warning when the chosen GPU type is unavailable.

---

## Why "the foundation"

Every dynamic feature was prototyped here first. From `git log --oneline`:

- `39fc9de` — dynamic per-user partition discovery
- `ac0b06e` — per-architecture cores/memory
- `264418c` — max-hours
- `23945ab` — unavailable-GPU warning
- `c54cf07` / `59da4df` — `:cpu`/`:gpu`/`:all` `@partition_filter`
- `caa8dc3` — exclude DOWN/DRAIN/MAINT nodes from per-partition aggregation

Once a feature stabilized here, it was copied (or extracted) elsewhere.

---

## DEBUG_GPU_DISCOVERY flag

Near the top of `partials/gpu_discovery.erb`:

```ruby
DEBUG_GPU_DISCOVERY = false
```

Flip to `true`, render once, the partial dumps `@detected_types`, `@user_partitions`, per-partition cores/memory/hours, `@unavailable_gpu_types` to `~/javi_gpu_debug.log`. Errors swallowed. Most useful debug tool in the codebase — re-use the pattern in any new partial.

---

## Slurm commands relied on

| Command | What it gives us |
|---|---|
| `scontrol show node --oneliner` | GPU GRES, features, `CfgTRES` mem/cpu, node state |
| `sinfo -o "%P %l"` | Partition wall-time limits |
| `sinfo` (no `--all`) | User-accessible partitions |
| `scontrol show partition --oneliner` | `MaxMemPerNode`, `MaxCPUsPerNode` caps |

---

## Read in this order

1. `README.md` — already a migration guide, best summary in my voice.
2. `form.yml.erb` — minimal integration.
3. `partials/gpu_discovery.erb` — the engine.
4. `form.js` — client-side glue.
5. `submit.yml.erb` — how `gpu_type` becomes `--gres=` / `--constraint=`.
6. `TESTING.md` — terminal vs. browser test recipes.

---

## Gotchas

- **ERB output buffer reset wipes `cluster:`** (commit `d4697a5`). `ERB.new(...).result(binding)` resets the outer buffer, silently dropping any YAML emitted before the include. **`cluster: "pax"` MUST come *after* the partial include.**
- **`@partition_filter` must be set BEFORE the include** — helpers read it at call time via Ruby default-arg evaluation.
- **GPU dropdown is hidden for CPU partitions.** `form.js` sets `gpuFormGroup.style.display = 'none'` and forces `gpu_type='none'` so cores/memory fall back to the no-GPU branch (commit `257ac80`).
- **Node-state matching uses `String#include?`.** `"DOWN+DRAIN"` matches correctly but a hypothetical `"DRAINABLE"` would too. Keep the unavailable-state list aligned with current Slurm docs.
- **`MaxMemPerNode` vs `CfgTRES`** — the partition cap can be far below physical memory (e.g., 503 GB cap on 1008 GB nodes). Always honor the partition cap.
- **In-sandbox renders fail** with `Unable to contact slurm controller` on non-controller nodes. Render on a login node when testing.
- **`partials/gpuDiscWorkingOCt4.erb`** is an old snapshot (~156 lines vs. ~597). Don't copy from it.

---

## Migrating new apps

When you start a new dynamic-form app, **copy from the shared `ood-resource-discovery` widget**, not from `javi_jupyter`. `javi_jupyter` keeps working but is no longer canonical.
