# ood-resource-discovery — the shared widget

**Local path (after you clone):** `/cluster/home/<your-utln>/ondemand/prod/ood-resource-discovery/`
**Repo:** <https://github.com/TuftsRT/ood-resource-discovery>

The cleaned-up, production-shared version of the dynamic-Slurm-discovery pattern prototyped in `javi_jupyter`. Lives in its own repo so multiple OOD apps can include it as one source of truth. **For any new dynamic-form app, include this** — not a copy of `gpu_discovery.erb`.

---

## What's in it

```
form.js                 # client-side glue
resource_discovery.erb  # the renamed/cleaned gpu_discovery.erb
README.md               # how to wire it into a form.yml.erb
```

Note the rename: `gpu_discovery.erb` → `resource_discovery.erb` reflects that the widget handles CPU partitions and time/memory/cores, not just GPUs.

---

## How it differs from `javi_jupyter`'s partial

Same architecture, same helper API. Additive diffs:

- New `generate_num_gpus_field` helper + `@max_gpus_by_partition_and_gpu` (parsed from the `:N` count in `Gres=gpu:type:N`).
- Generated fields marked `cacheable: false` / `display: true` so OOD doesn't cache stale form state.
- Drops the legacy `'any'` GPU sentinel.
- Tightens no-GPU help text.
- Sensible default partition based on `@partition_filter`.

If you've read `javi_jupyter`'s partial, you've effectively read this one.

---

## Wiring it into a new app

```yaml
---
cluster: "pax"   # MUST come AFTER the include — see gotchas

<%
  @partition_filter = :gpu   # or :cpu or :all
%>
<%= render(file: 'partials/resource_discovery.erb') %>
<%
  attributes:
    partition: <%= generate_partition_field %>
    gpu_type: <%= generate_gpu_type_field %>
    num_cores: <%= generate_num_cores_field %>
    num_memory: <%= generate_num_memory_field %>
    num_hours: <%= generate_num_hours_field %>
    num_gpus: <%= generate_num_gpus_field %>
%>
```

Pull in `form.js` via the standard OOD form-asset mechanism. In `submit.yml.erb`, translate `gpu_type` to `--gres=gpu:<type>:<n>` and any `--constraint=`.

The README inside the repo has the up-to-date version — trust it over this guide.

---

## Same gotchas as `javi_jupyter`

`cluster: "pax"` after the include; set `@partition_filter` first; `MaxMemPerNode` honored; render on a login node. Full list in [javi_jupyter.md](javi_jupyter.md#gotchas).

---

## Migration plan for existing apps

| App | Discovery source today |
|---|---|
| `javi_jupyter` | own `partials/gpu_discovery.erb` (canonical historical) |
| `ansys` | byte-identical copy of `javi_jupyter`'s partial |
| `igv` | similar copy |
| `OpenComposer` | own `partials/slurm_discovery.erb` (parallel re-implementation) |

End state: every app includes `resource_discovery.erb` from this repo. Commit `15fdb7e` in OpenComposer was the first step in that direction.
