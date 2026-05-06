# javi_jupyter — the foundation

**Path:** `/cluster/home/jlavea01/ondemand/prod/javi_jupyter/`
**Repo:** `git@github.com:javier-la200426/jupyterNotebookDynamicFormYml.git`

This is **the foundation app** for everything else. It's where the dynamic Slurm-aware launch-form pattern was prototyped. Every other launch form (`ansys`, `igv`, the OpenComposer apps) copies from it. Before touching anything else, read this directory.

---

## What is the "dynamic form" pattern

Open OnDemand Batch Connect apps normally hardcode their launch-form choices (partitions, GPU types, max cores/memory/hours) in static YAML. You'd write something like:

```yaml
partition:
  widget: select
  options:
    - gpu
    - batch
    - largemem
```

…and it would render the same three options for every user, regardless of whether they actually have access to those partitions or whether nodes are currently up.

The dynamic pattern replaces the static list with values **discovered live from Slurm at form-render time, scoped to the logged-in user**. When the user opens the form, the server runs `sinfo` and `scontrol` *as that user*, parses the output, and emits a form whose dropdowns and `max=` values reflect exactly what they can request right now. `form.js` then re-runs the same lookups in the browser whenever the user changes partition or GPU type, keeping cores/memory/hours in sync.

---

## How `gpu_discovery.erb` works

The ERB partial at `partials/gpu_discovery.erb` does five things:

1. **Shells out to Slurm:** `scontrol show node --oneliner`, `sinfo -o "%P %l"`, `scontrol show partition --oneliner`, and `sinfo` (no `--all`, so Slurm filters to user-accessible partitions).
2. **Builds Ruby hashes:** partition→GPU types, (partition, gpu)→max cores, (partition, gpu)→max memory MB, partition→max hours, plus an "unavailable GPUs" set (where all matching nodes are DOWN/DRAIN/MAINT/etc.).
3. **Defines helper methods** (`generate_partition_field`, `generate_num_cores_field`, `generate_num_memory_field`, `generate_num_hours_field`, `generate_gpu_type_field`) that emit YAML field definitions for the form.
4. **Serializes the hashes to JSON** and embeds them in `data-*` HTML attributes (`data-partition-gpu-max-cores`, etc.).
5. `form.js` reads those `data-*` blobs, listens for `change` events on `partition` and `gpu_type`, rewrites the GPU dropdown, sets `max`/help text on cores/memory/hours, and shows a red warning when the chosen GPU type is currently unavailable.

---

## Why "the foundation"

Every dynamic feature was prototyped here first. Trace it through `git log --oneline`:

- `39fc9de` — dynamic per-user partition discovery
- `ac0b06e` — per-architecture cores/memory
- `264418c` — max-hours
- `23945ab` — unavailable-GPU warning
- `c54cf07` / `59da4df` — `:cpu`/`:gpu`/`:all` `@partition_filter`
- `caa8dc3` — exclude DOWN/DRAIN/MAINT nodes from per-partition aggregation

Once a feature stabilized here it was copied (or extracted) elsewhere.

---

## The DEBUG_GPU_DISCOVERY flag

Near the top of `partials/gpu_discovery.erb`:

```ruby
DEBUG_GPU_DISCOVERY = false
```

Flip it to `true`, render the form once, and the partial dumps every intermediate hash (`@detected_types`, `@user_partitions`, per-partition cores/memory/hours, `@unavailable_gpu_types`) to `~/javi_gpu_debug.log`. Errors are swallowed so the form keeps rendering.

This is the single most useful debug tool in the codebase. Re-use the pattern in any new partial.

---

## Slurm commands the discovery relies on

| Command | What it gives us |
|---|---|
| `scontrol show node --oneliner` | GPU GRES, features, `CfgTRES` mem/cpu, node state |
| `sinfo -o "%P %l"` | Partition wall-time limits |
| `sinfo` (no `--all`) | User-accessible partitions (Slurm enforces the filter) |
| `scontrol show partition --oneliner` | `MaxMemPerNode`, `MaxCPUsPerNode` partition caps |

---

## Read in this order

1. `README.md` — already a migration guide; the best summary of the pattern in Javier's voice.
2. `form.yml.erb` — minimal integration: set `@partition_filter`, include the partial, call the helper methods.
3. `partials/gpu_discovery.erb` — the engine.
4. `form.js` — client-side glue.
5. `submit.yml.erb` — how the form's `gpu_type` field becomes `--gres=` / `--constraint=` on `sbatch`.
6. `TESTING.md` — terminal vs. browser test recipes, including how to render the partial standalone with the portable Ruby.

---

## Gotchas (don't lose a day to these)

- **ERB output buffer reset wipes `cluster:`** (commit `d4697a5`). `ERB.new(...).result(binding)` resets the outer template's output buffer, silently dropping any YAML emitted before the include. **`cluster: "pax"` MUST come *after* the partial include block.**
- **`@partition_filter` must be set BEFORE the include.** Helpers read it at call time via Ruby default-arg evaluation — set the variable first or you'll get the wrong scope.
- **GPU architecture dropdown is hidden for CPU partitions.** `form.js` sets `gpuFormGroup.style.display = 'none'` and forces `gpu_type='none'` so cores/memory lookups fall back to the no-GPU branch (commit `257ac80`). If your new app has a GPU dropdown that should always show, you'll need to override this.
- **Node-state matching uses `String#include?`.** `"DOWN+DRAIN"` matches correctly, but a hypothetical `"DRAINABLE"` would *also* match. Keep the unavailable-state list aligned with current Slurm docs.
- **`MaxMemPerNode` vs `CfgTRES`**: the partition cap can be far below physical memory (e.g., 503 GB cap on 1008 GB nodes). Always honor the partition cap.
- **In-sandbox renders fail with `Unable to contact slurm controller`** if you try to render the form on a non-controller node. Render on a login node when testing.
- **`partials/gpuDiscWorkingOCt4.erb`** is an old snapshot (~156 lines vs. ~597). It predates per-partition memory/cores/hours, the user-partition filter, and the unavailable-GPU warnings. Don't copy from it; use the current `gpu_discovery.erb` (or better, the shared `ood-resource-discovery` widget).

---

## When to migrate to the shared widget

The long-term direction is `TuftsRT/ood-resource-discovery` (see [ood-resource-discovery.md](ood-resource-discovery.md)). When you start a new dynamic-form app, **copy from the shared widget, not from `javi_jupyter`**. `javi_jupyter` will keep working but is no longer the canonical implementation.
