# ansys — migration story + two cautionary tales

**Local path (after you clone):** `/cluster/home/<your-utln>/ondemand/prod/ansys/`
**Repo:** `git@github.com:javier-la200426/ansys-dynamic.git`

A Batch Connect app that launches an ANSYS Workbench VNC desktop inside a Slurm job. The cleanest worked example of **migrating a hardcoded OOD app to the dynamic pattern**, and the two `.md` files in the repo (`WHY_GPU_ONLY.md`, `PERMISSIONS_TRAP.md`) capture incidents you should not repeat.

---

## Migration story

The whole arc in four commits:

| Commit | What |
|---|---|
| `cf75ff9` | Initial commit — hardcoded `--partition=gpu` and `--gres=gpu:1` |
| `23dd1cd` | Make form dynamic — adopt `gpu_discovery.erb` from `javi_jupyter` |
| `bac7cb8` | Fix Ansys version discovery — `module -t avail ansys` with `2025R2` fallback |
| `b04c26b` | Made GPU-only — the one-line `@partition_filter = :gpu` constraint |

The dynamic refactor accidentally exposed a runtime constraint that the hardcoded version had hidden.

---

## Lesson 1 — `WHY_GPU_ONLY.md` (read it)

> *"The original app was accidentally GPU-only because it hardcoded `--partition=gpu` and `--gres=gpu:1`. When we refactored it to be Slurm-driven, users could suddenly select CPU-only partitions like `batch` — and Ansys Workbench failed at startup with an internal addin error (`Ans.SceneGraphChart.SceneGraphAddin`). Workbench needs an actual GPU at runtime, not just the `-oglmesa` flag."*

Fix: `@partition_filter = :gpu` in `form.yml.erb`. Other apps (`javi_jupyter`, `igv`) keep the `:cpu`/`:all` modes.

**Takeaway:** when you generalize a previously-narrow app, **re-test every newly-reachable code path**. Hardcoded defaults often mask runtime requirements you didn't know existed.

---

## Lesson 2 — `PERMISSIONS_TRAP.md` (read it)

Two key quotes:

> *"A correct-looking `PATH` is not enough if the underlying directories are permission-blocked. `runwb2: command not found` can be caused by inaccessible directories, not just a missing module."*

Confirm with `namei -l <path>` before blaming the module or launcher. I lost half a day to this.

> *"If the app is written to tie the whole session to `runwb2`, closing Ansys ends the session by design."*

A `Failed to connect to server` in noVNC after the user closes Workbench is not a noVNC bug — the wrapper `wait`s on `runwb2` and tears VNC down on exit.

**Takeaway:** when something looks like a bug, first check whether it's the **expected teardown of a deliberate design**.

---

## Discovery partial relationship

`ansys/partials/gpu_discovery.erb` is byte-identical to `javi_jupyter`'s. Copied during migration. Long-term plan: replace all copies with the shared `ood-resource-discovery` partial. Until then, fix bugs in all copies.

---

## Notable files

- `form.yml.erb` (~38 lines) — sets `@partition_filter = :gpu`, includes shared partial, runs `module -t avail ansys`.
- `submit.yml.erb` — keeps a defensive `skip_gres` branch even though the form prevents reaching it.
- `template/` — VNC + Workbench wrapper. Historical copy/paste leftovers (`Launch fastqc` comment, `OnDemand/Paraview` references) are flagged in `PERMISSIONS_TRAP.md` as cleanup targets.

---

## Read in this order

1. `WHY_GPU_ONLY.md`
2. `PERMISSIONS_TRAP.md`
3. `README.md`
4. `form.yml.erb`
5. `template/`

---

## Before adding any Ansys feature

Two questions to ask first:

1. Does it affect partition / GPU constraint? Re-read `WHY_GPU_ONLY.md`.
2. Does it touch the launcher script or noVNC teardown? Re-read `PERMISSIONS_TRAP.md`.

Most "small" Ansys changes ended up touching one of those two pages.
