# 06 — Projects

One file per project I worked on, in the order an intern should encounter them.

## Read in this order

1. **[javi_jupyter.md](javi_jupyter.md)** — The foundation. The dynamic-Slurm-discovery pattern that the rest of this work copies. **Start here.**
2. **[ood-resource-discovery.md](ood-resource-discovery.md)** — The cleaned-up shared widget extracted from `javi_jupyter`. The path forward.
3. **[ansys.md](ansys.md)** — A cautionary tale: migrating a hardcoded launch app to the dynamic pattern, plus two lessons-learned docs (`WHY_GPU_ONLY.md`, `PERMISSIONS_TRAP.md`) you should read.
4. **[opencomposer.md](opencomposer.md)** — A bigger meta-app where the dynamic pattern was ported into a 3rd-party framework. Notable for the `TESTING_REPORT.md` portable-Ruby trick referenced throughout this guide.
5. **[monitoring_apps.md](monitoring_apps.md)** — `tufts-jobmonitor` and `tufts-systemstatus`: passenger (Sinatra) apps embedded in the OOD dashboard. A different pattern than launch forms; useful template for any new monitoring widget.

## Other directories in `/cluster/home/<utln>/ondemand/prod/`

The directories I have **not** worked with much:

- `cluster-dashboard`, `dashboard` — older / RT-managed dashboard variants.
- `igv` — uses a copy of the dynamic-discovery partial (good cross-reference for the pattern).
- `javier_job_tests` — scratch test directory.
- `JobApp` — older job-monitoring experiment, superseded by `tufts-jobmonitor`.
- `jl_matlab_server`, `matlab_server` — older Matlab launch attempts; check if still needed.
- `ood_v4_tufts_dev` — a clone of the upstream `TuftsRT/ood_v4_tufts_dev` repo containing many production OOD apps including the original "dynamicOptions" version of the discovery pattern.
- `Rstudio` — older / unused.
- `ansys_initial_commit` — pre-dynamic snapshot of `ansys`. Kept as reference; ignore for new work.

If git history is empty or the README is missing, treat the directory as legacy until you can verify otherwise.
