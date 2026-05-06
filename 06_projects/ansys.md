# ansys — a migration story with two cautionary tales

**Local path (after you clone):** `/cluster/home/<your-utln>/ondemand/prod/ansys/`
**Repo:** `git@github.com:javier-la200426/ansys-dynamic.git`

A Batch Connect Open OnDemand app that launches an ANSYS Workbench VNC desktop session inside a Slurm job on the `pax` cluster. It appears as a tile on the OOD dashboard under *Interactive Apps → Apps* and is the GUI entry point for users who want Workbench without ssh + X11.

This app is the cleanest worked example of **migrating a hardcoded OOD app to the dynamic-discovery pattern** — and the two `.md` files in the directory (`WHY_GPU_ONLY.md`, `PERMISSIONS_TRAP.md`) are required reading because each captures a debugging incident I don't want you to repeat.

---

## Migration story (hardcoded → dynamic → GPU-only)

The `git log --oneline` tells the whole arc in four commits:

| Commit | What |
|---|---|
| `cf75ff9` | Initial commit of Ansys OnDemand app — hardcoded `--partition=gpu` and `--gres=gpu:1` |
| `23dd1cd` | Make form dynamic with Slurm-driven partition/GPU discovery — adopted the `gpu_discovery.erb` pattern from `javi_jupyter` |
| `bac7cb8` | Fix Ansys version discovery in form render — `module -t avail ansys` parsing in `form.yml.erb` with a `2025R2` fallback |
| `b04c26b` | "made this gpu only app" — the one-line `@partition_filter = :gpu` constraint |

The dynamic refactor accidentally exposed a runtime constraint that the original hardcoded version had hidden. Which leads to…

---

## Lesson 1 — `WHY_GPU_ONLY.md`

Read it. The key sentence:

> *"The original app was accidentally GPU-only because it hardcoded `--partition=gpu` and `--gres=gpu:1`. When we refactored it to be Slurm-driven and dynamic, users could suddenly select CPU-only partitions like `batch` — and Ansys Workbench failed at startup with an internal addin error (`Ans.SceneGraphChart.SceneGraphAddin`). Workbench needs an actual GPU at runtime, not just the `-oglmesa` flag."*

The fix is one line in `form.yml.erb`:

```ruby
@partition_filter = :gpu
```

The shared partial's `:cpu`/`:all` modes are still valid for `javi_jupyter` and `igv` — only Ansys is constrained.

**Takeaway for the intern:** when you generalize a previously-narrow app, **re-test every newly-reachable code path**. Hardcoded defaults often mask runtime requirements you didn't know existed.

---

## Lesson 2 — `PERMISSIONS_TRAP.md`

Two important quotes from this file:

> *"A correct-looking `PATH` is not enough if the underlying directories are permission-blocked. `runwb2: command not found` can be caused by inaccessible directories, not just a missing module."*

Confirm with `namei -l <path>` before blaming the module or the launcher. I lost half a day to this.

> *"If the app is written to tie the whole session to `runwb2`, closing Ansys ends the session by design."*

A `Failed to connect to server` error in noVNC after the user closes Workbench is not a noVNC bug — the wrapper script `wait`s on the `runwb2` process and tears VNC down on exit. The fix is to message it more clearly to the user, not to "fix" the disconnect.

**Takeaway:** when something looks like a bug, first check whether it's the **expected teardown of a deliberate design**.

---

## Discovery partial relationship

As of when I left, `ansys/partials/gpu_discovery.erb` is **byte-identical** to `javi_jupyter/partials/gpu_discovery.erb`. It was copied when ansys was migrated. The long-term plan is to replace it with the shared `ood-resource-discovery` partial so both apps (and `igv`) consume one source of truth. Until then, if you fix a bug in one copy, fix it in all three.

---

## Other notable files

- `form.yml.erb` (~38 lines) — sets `@partition_filter = :gpu`, calls the shared partial, runs `module -t avail ansys` for version discovery.
- `submit.yml.erb` — still keeps the `skip_gres` defensive branch even though the form now prevents reaching it. Not strictly necessary; harmless.
- `template/` — VNC wrapper (`script.sh.erb`, `before.sh.erb`). Historical copy/paste leftovers (a `Launch fastqc` comment, an `OnDemand/Paraview` reference in older commits) are flagged in `PERMISSIONS_TRAP.md` as cleanup targets.

---

## Read in this order

1. `WHY_GPU_ONLY.md` — the "why we constrained partitions" story.
2. `PERMISSIONS_TRAP.md` — the "PATH lies" debugging story.
3. `README.md` — user-facing overview.
4. `form.yml.erb` — see the `@partition_filter = :gpu` line in context.
5. `template/` — how the VNC + Workbench launcher script is wired.

---

## If you're asked to add a new Ansys feature

The first two questions to ask:

1. Does the change affect the partition / GPU constraint? If so, re-read `WHY_GPU_ONLY.md`.
2. Does the change touch the launcher script or the noVNC teardown? If so, re-read `PERMISSIONS_TRAP.md`.

Most "small" Ansys changes ended up touching one of those two pages.
