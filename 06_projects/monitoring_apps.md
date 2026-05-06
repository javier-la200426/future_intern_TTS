# tufts-jobmonitor + tufts-systemstatus — monitoring apps

**Paths:**
- `/cluster/home/jlavea01/ondemand/prod/tufts-jobmonitor/`
- `/cluster/home/jlavea01/ondemand/prod/tufts-systemstatus/`

**Repos:**
- `git@github.com:TuftsRT/tufts-jobmonitor.git`
- `git@github.com:TuftsRT/tufts-systemstatus.git`

Two sister Sinatra/Passenger apps embedded in the OnDemand dashboard. Unlike `javi_jupyter` or `ansys` (which are **batch_connect** launch forms that submit a Slurm job), these are **passenger web apps** that shell out to Slurm on every request and render live data.

This is a useful pattern any time you want a read-only monitoring widget inside OnDemand.

---

## Purpose

| App | What it shows |
|---|---|
| `tufts-jobmonitor` | Per-user view of active and historical Slurm jobs (`squeue` + `sacct`), with detail / efficiency / script / cancel actions |
| `tufts-systemstatus` | Cluster-wide partition / node / GPU availability dashboard (`sinfo` + `scontrol` + `squeue` across all users) |

---

## Architecture (same shape in both)

- `config.ru` boots a stock `Sinatra::Application`.
- `app.rb` defines HTTP routes.
- `lib/*.rb` wraps every Slurm CLI call through `Open3.capture3` and parses the output.
- `views/index.erb` renders the page.
- `public/script.js` polls the JSON APIs on a timer for live updates.

If you can read one, you can read the other.

### Routes — `tufts-jobmonitor/app.rb`

- `GET /` — main page
- `GET /api/current-jobs` — `squeue --me`
- `GET /api/job-history` — `sacct`, capped at 30 days
- `GET /api/job/:jobid` — `scontrol show jobid -dd`, falls back to `sacct` for old jobs
- `GET /efficiency` — `seff`
- `GET /info` — bundled `jobinfo` script
- `GET /script` — `sacct --batch-script`
- `DELETE /api/job/:jobid` — `scancel`
- `GET /api/summary`
- `GET /health`
- `GET /api/debug/sacct`

### Routes — `tufts-systemstatus/app.rb`

- `GET /` — main page
- `GET /api/data`
- `GET /api/nodes` — `scontrol show node --oneliner`
- `GET /api/gpu` — parses `AllocTRES` for GPU usage
- `GET /api/partitions` — `sinfo` + `scontrol show reservations` + `squeue` across all users
- `GET /health`

---

## Dashboard embedding

Both ship a `views/dashboard_iframe.html.erb` that the OnDemand dashboard pulls in as a custom widget. It renders an `<iframe src="/pun/sys/tufts-jobmonitor">` (or `.../cluster-dashboard`) and auto-resizes via a `resizeIframe()` listener. Manifests set `new_window: false` so the app loads inline rather than popping out into its own page.

This is how a passenger app becomes a panel inside the main dashboard rather than a separate menu item.

---

## Representative commits (what the polish actually looks like)

The interesting work in these repos is parser tuning and UX polish — the same kind of small, careful changes you'll be doing.

**`tufts-jobmonitor`:**
- `2bbe310` — *"Fix NodeList regex to avoid matching `ReqNodeList`/`ExcNodeList` by adding a word boundary lookbehind"*
- `37194e3` — *"Fix `scontrol` time field parsing regex that was capturing adjacent key=value pairs"*
- `b84f60e` — *"Add pending-job user-limit warning icon with custom tooltip for MaxPerUser reasons"*
- `5eb3478` — *"Default job history sort to most recent first; push None/N/A to the bottom"*
- `00c8c13` — *"Make node list names clickable SSH links to OOD shell, using batch host for multi-node jobs"*

**`tufts-systemstatus`:**
- `d68c796` — *"Change GPU Count column to GPUs (Free/Total) by parsing AllocTRES"*
- `29a5577` — *"Reserved filter for nodes with active SLURM reservations"*
- `532a8bd` — *"Reorder Down/In Use fields so the GPU progress bar visually associates with usage"*

Notice the regex commits — Slurm's text outputs are full of edge cases. Always test parsing against several real outputs (CPU job, GPU job, job with a reservation, multi-node job, pending job, completed job…) before declaring victory.

---

## Read in this order (jobmonitor first)

`/cluster/home/jlavea01/ondemand/prod/tufts-jobmonitor/`:
1. `ARCHITECTURE.md`
2. `app.rb`
3. `lib/job_parser.rb` (or whichever lib file holds the parsers — there are a couple)
4. `views/index.erb`
5. `views/dashboard_iframe.html.erb`
6. `public/script.js`

`/cluster/home/jlavea01/ondemand/prod/tufts-systemstatus/`:
1. `system-status-architecture.txt`
2. `app.rb`
3. `lib/slurm_parser.rb`
4. `views/index.erb`
5. `views/dashboard_iframe.html.erb`

---

## Why clone this pattern for new monitoring widgets

For *any* new read-only monitoring panel (queue stats, storage quotas, license usage, GPU temperature, whatever), this stack is the smallest path from a CLI tool to a live OnDemand panel:

```
Sinatra route → Open3.capture3 parser → JSON API
        ↓
ERB + JS poller → optional dashboard_iframe.html.erb
```

Both repos live under `TuftsRT` as production-deployable templates. **Copy the layout, swap the parser, you have a new tile.** The auth, embedding, polling, and dashboard-iframe glue are all already worked out.

---

## Watch out for

- **Slurm output parsing is regex hell.** Always test against a *variety* of real outputs (see commits `2bbe310` and `37194e3` for cautionary examples).
- **`squeue --me` vs `squeue` (all users).** Easy to confuse. The jobmonitor wants per-user; systemstatus wants cluster-wide.
- **`AllocTRES` parsing for GPUs** — the format is `cpu=N,mem=NM,gres/gpu=N`. If GPU count is missing the field is just absent, not zero.
- **Old finished jobs** are not in `scontrol show job` (only currently-known jobs are). The fallback to `sacct` in `/api/job/:jobid` is what makes the app work for historical lookups.
- **Polling intervals** in `public/script.js` — too aggressive will hammer the Slurm controller. The current intervals are tuned; don't drop them without checking with RT.
