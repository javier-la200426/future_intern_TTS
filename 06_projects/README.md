# 06 — Projects

One file per project I worked on, in the order to read them.

1. **[javi_jupyter.md](javi_jupyter.md)** — The foundation. The dynamic-Slurm-discovery pattern that the rest of this work copies. **Start here.**
2. **[ood-resource-discovery.md](ood-resource-discovery.md)** — Cleaned-up shared widget extracted from `javi_jupyter`. The path forward.
3. **[ansys.md](ansys.md)** — Cautionary tale: migrating a hardcoded launch app to the dynamic pattern, plus two lessons-learned docs you should read.
4. **[opencomposer.md](opencomposer.md)** — Bigger meta-app where the dynamic pattern was ported into a 3rd-party framework. Contains the `TESTING_REPORT.md` portable-Ruby story.
5. **[monitoring_apps.md](monitoring_apps.md)** — `tufts-jobmonitor` and `tufts-systemstatus`: passenger (Sinatra) apps embedded in the OOD dashboard. Different pattern from launch forms.

## Other directories under `ondemand/prod/`

Ones I haven't worked with much:

- `cluster-dashboard`, `dashboard` — older / RT-managed dashboard variants.
- `igv` — uses a copy of the dynamic-discovery partial (cross-reference for the pattern).
- `javier_job_tests` — scratch test directory.
- `JobApp` — older job-monitoring experiment, superseded by `tufts-jobmonitor`.
- `jl_matlab_server`, `matlab_server` — older Matlab attempts; check if still needed.
- `ood_v4_tufts_dev` — clone of `TuftsRT/ood_v4_tufts_dev` containing many production OOD apps including the original "dynamicOptions" version of the discovery pattern.
- `Rstudio` — older / unused.
- `ansys_initial_commit` — pre-dynamic snapshot of `ansys`. Reference only.

If git history is empty or the README is missing, treat the directory as legacy until you can verify otherwise.
