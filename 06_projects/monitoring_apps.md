# tufts-jobmonitor + tufts-systemstatus — monitoring apps

**Local paths (after you clone):**
- `/cluster/home/<your-utln>/ondemand/prod/tufts-jobmonitor/`
- `/cluster/home/<your-utln>/ondemand/prod/tufts-systemstatus/`

**Repos:**
- `git@github.com:TuftsRT/tufts-jobmonitor.git`
- `git@github.com:TuftsRT/tufts-systemstatus.git`

Two sister Sinatra/Passenger apps embedded in the OnDemand dashboard. Unlike the launch-form apps (`javi_jupyter`, `ansys`), these are **passenger web apps** that shell out to Slurm on every request and render live data. Useful template for any new monitoring widget.

---

## Purpose

| App | What it shows |
|---|---|
| `tufts-jobmonitor` | Per-user view of active + historical Slurm jobs (`squeue` + `sacct`), with detail / efficiency / script / cancel actions |
| `tufts-systemstatus` | Cluster-wide partition / node / GPU availability (`sinfo` + `scontrol` + `squeue` across all users) |

---

## Architecture (same in both)

- `config.ru` boots `Sinatra::Application`.
- `app.rb` defines HTTP routes.
- `lib/*.rb` wraps every Slurm CLI call through `Open3.capture3` and parses output.
- `views/index.erb` renders the page.
- `public/script.js` polls JSON APIs on a timer for live updates.

If you can read one, you can read the other.

### Routes

**`tufts-jobmonitor/app.rb`:** `GET /` (page), `GET /api/current-jobs` (squeue --me), `GET /api/job-history` (sacct, 30-day cap), `GET /api/job/:jobid` (scontrol → sacct fallback for old jobs), `GET /efficiency` (seff), `GET /info` (bundled `jobinfo` script), `GET /script` (sacct --batch-script), `DELETE /api/job/:jobid` (scancel), plus `/api/summary`, `/health`, `/api/debug/sacct`.

**`tufts-systemstatus/app.rb`:** `GET /` (page), `GET /api/data`, `GET /api/nodes` (scontrol show node --oneliner), `GET /api/gpu` (parses AllocTRES), `GET /api/partitions` (sinfo + reservations + cluster-wide squeue), `GET /health`.

---

## Dashboard embedding

Both ship `views/dashboard_iframe.html.erb` that the OnDemand dashboard pulls in as a custom widget — `<iframe src="/pun/sys/tufts-jobmonitor">` with auto-resize via `resizeIframe()`. Manifests set `new_window: false` so the app loads inline.

---

## Representative commits

The work in these repos is mostly parser tuning and UX polish — the kind of small careful changes you'll be doing.

**`tufts-jobmonitor`:**
- `2bbe310` — Fix NodeList regex to avoid matching `ReqNodeList`/`ExcNodeList` (word boundary lookbehind)
- `37194e3` — Fix `scontrol` time field regex capturing adjacent key=value pairs
- `b84f60e` — Pending-job user-limit warning icon + tooltip for MaxPerUser
- `5eb3478` — Default job history sort to most recent first; push None/N/A to bottom
- `00c8c13` — Make node names clickable SSH links to OOD shell

**`tufts-systemstatus`:**
- `d68c796` — GPU Count column → GPUs (Free/Total) by parsing AllocTRES
- `29a5577` — Reserved filter for nodes with active SLURM reservations
- `532a8bd` — Reorder Down/In Use fields so the GPU progress bar associates with usage

The regex commits are warnings: Slurm text outputs are full of edge cases. Always test parsing against multiple real outputs (CPU job, GPU job, reservation, multi-node, pending, completed) before declaring victory.

---

## Read in this order

In the cloned `tufts-jobmonitor` repo: `ARCHITECTURE.md` → `app.rb` → `lib/job_parser.rb` → `views/index.erb` → `views/dashboard_iframe.html.erb` → `public/script.js`.

In the cloned `tufts-systemstatus` repo: `system-status-architecture.txt` → `app.rb` → `lib/slurm_parser.rb` → `views/index.erb` → `views/dashboard_iframe.html.erb`.

---

## Why clone this pattern

For any new read-only monitoring panel (queue stats, storage quotas, license usage, GPU temperature), this stack is the smallest path from a CLI tool to a live OnDemand panel:

```
Sinatra route → Open3.capture3 parser → JSON API → ERB + JS poller → optional dashboard_iframe.html.erb
```

Both repos are production-deployable templates under TuftsRT. **Copy the layout, swap the parser, you have a new tile.** Auth, embedding, polling, and dashboard glue are already worked out.

---

## Watch out for

- **Slurm output parsing is regex hell.** Test against varied real outputs.
- **`squeue --me` vs `squeue` (all users)** — easy to confuse.
- **`AllocTRES` GPU format** is `cpu=N,mem=NM,gres/gpu=N`. If GPU count is missing the field is just absent, not zero.
- **Old finished jobs aren't in `scontrol show job`** — only currently-known. The `sacct` fallback is what makes historical lookups work.
- **Polling intervals in `public/script.js`** — too aggressive will hammer the Slurm controller. Don't drop them without checking with RT.
