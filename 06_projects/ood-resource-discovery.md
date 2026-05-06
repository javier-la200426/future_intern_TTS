# ood-resource-discovery — the shared widget

**Path:** `/cluster/home/jlavea01/ondemand/prod/ood-resource-discovery/`
**Repo:** <https://github.com/TuftsRT/ood-resource-discovery> (HTTPS remote)

This is the **cleaned-up, production-shared version** of the dynamic-Slurm-discovery pattern that was prototyped in `javi_jupyter`. It lives in its own repo so multiple OOD apps can include it as a single source of truth instead of each maintaining a drifting copy.

If you're starting a new dynamic-form app, **include this**, not a copy of `gpu_discovery.erb`.

---

## What's in the directory

```
form.js                 # client-side glue (reads data-* attributes, updates dropdowns/maxes)
resource_discovery.erb  # the renamed/cleaned gpu_discovery.erb
README.md               # how to wire it into your form.yml.erb
.gitignore
```

Note the rename: `gpu_discovery.erb` → `resource_discovery.erb`. The new name reflects that the widget handles CPU partitions and time/memory/cores too — not just GPU discovery.

---

## How it differs from `javi_jupyter/partials/gpu_discovery.erb`

Same architecture, same helper API. The diff is small and additive:

- Adds a `generate_num_gpus_field` helper plus `@max_gpus_by_partition_and_gpu` tracking (parsed from the `:N` count in `Gres=gpu:type:N`).
- Marks generated fields `cacheable: false` / `display: true` so OOD doesn't cache stale form state.
- Drops the legacy `'any'` GPU sentinel from concrete option lists.
- Tightens the no-GPU help text.
- Sets a sensible default partition based on `@partition_filter`.

If you've read `javi_jupyter/partials/gpu_discovery.erb`, you've effectively read this one — pay attention only to the additions above.

---

## How to wire it into a new app

1. Clone or symlink `ood-resource-discovery` into your app's `partials/` directory (or include via path).
2. In your `form.yml.erb`:

   ```yaml
   ---
   cluster: "pax"   # MUST come AFTER the include — see gotcha below

   <%
     @partition_filter = :gpu   # or :cpu, or :all
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

3. Pull in `form.js` via the standard OOD form-asset mechanism so the client-side behavior fires.
4. In `submit.yml.erb`, translate `gpu_type` to `--gres=gpu:<type>:<n>` and any necessary `--constraint=`.

The full README inside the repo has the up-to-date version of these instructions; trust that over this guide.

---

## The same gotchas apply

- `cluster: "pax"` must come *after* the partial include (ERB buffer reset bug).
- Set `@partition_filter` *before* the include.
- `MaxMemPerNode` partition cap can be lower than physical RAM — the widget honors it; don't second-guess it.
- Render on a login node, not a compute node, or you'll hit "Unable to contact slurm controller."

See [javi_jupyter.md](javi_jupyter.md#gotchas-dont-lose-a-day-to-these) for the full list.

---

## Migration plan for existing apps

The current state of the world (May 2026):

| App | Discovery source |
|---|---|
| `javi_jupyter` | own `partials/gpu_discovery.erb` (canonical historical) |
| `ansys` | byte-identical copy of `javi_jupyter`'s partial |
| `igv` | similar copy |
| `OpenComposer/partials/slurm_discovery.erb` | parallel re-implementation in OpenComposer's framework |

The intended end state: every app includes `ood-resource-discovery`'s `resource_discovery.erb`. Migrating each app is a small but careful task — the `OpenComposer` adoption is a worked example (commit `15fdb7e` "Adopt ood-resource-discovery improvements into OpenComposer").
