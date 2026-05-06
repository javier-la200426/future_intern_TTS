# 03 — Testing

The most important chapter. Almost every painful day I had came from skipping or shortcutting testing — AI agents make code that *looks* right and is broken in non-obvious ways.

Three layers, all required:

1. **Browser** — render the form / hit the API in a browser to see actual user-facing behavior.
2. **Backend in isolation** — run Ruby/ERB in the terminal to verify Slurm queries and parsing.
3. **Real Slurm submission** — verify the job actually starts and the session works end-to-end.

Skipping layer 3 is how you ship a form that looks right but fails at `sbatch` because of a `gres` name mismatch.

---

## Layer 1 — Browser testing

### Manually

1. Open <https://ondemand-p01.pax.tufts.edu/pun/sys/dashboard>, Duo push.
2. Sandbox Apps → click your app's tile (every directory under `~/ondemand/prod/<app>/` shows up automatically).
3. Submit the form, watch DevTools Console + Network.

Useful URL patterns that re-render from source on every request (no caching):

- `https://ondemand-p01.pax.tufts.edu/pun/sys/dashboard/batch_connect/dev/<app>` — Batch Connect launch forms.
- `https://ondemand-p01.pax.tufts.edu/pun/dev/<app>` — passenger apps (jobmonitor, systemstatus, OpenComposer).

### Frontend `?debug=1` flag

Append `?debug=1` to any form URL to enable verbose `console.log`s. The `form.js` files in this repo all check for `URLSearchParams` containing `debug` and gate logging on it. **Add the same pattern to any new app you build** — three lines of code, saves hours.

### Automated browser testing with Playwright MCP

Once Playwright MCP is installed on your laptop (chapter 02), the workflow:

1. Cluster-side agent finishes editing.
2. Ask it: *"Write me a prompt for my laptop AI tester. URL: `<dev URL>`. Verify [behavior]. Include what to click, what to look for in the console, and which network calls to inspect."*
3. Copy that prompt to your **laptop** terminal where Codex/Claude is running. Add this preamble:

   > *"Use Playwright MCP to navigate to `<URL>`. I will log in manually first — pause after navigating and wait for me to confirm I've completed Duo. Then proceed with: [paste prompt]"*

4. The agent navigates. You do the Duo push (one human-required step). You say "logged in, continue."
5. Agent clicks around, takes screenshots, reads console logs, reports back.

You can watch the browser the whole time.

---

## Layer 2 — Backend in isolation (no Ruby on the cluster)

OOD uses Ruby ERB heavily, but login nodes don't have Ruby in `$PATH`. So when an agent says *"let me test this ERB in a `ruby` REPL"* — it can't.

**The fix:** stash a portable Ruby in your home directory. Extract Ruby 3.0.7 + the `psych` (YAML) and `json` gems from Rocky 9 RPMs (`rpm2cpio` + `cpio -idmv` on `ruby`, `ruby-libs`, `rubygem-psych`, `rubygem-json`). Any LLM walks you through this in 10 minutes. Result: ~30 MB at, say, `~/tmp-ruby/`.

Source these env vars (adjust paths to your extraction):

```bash
RUBY=~/tmp-ruby/root/usr/bin/ruby
LD_LIBRARY_PATH=~/tmp-ruby/root/usr/lib64
RUBYLIB=~/tmp-ruby/root/usr/share/ruby/vendor_ruby/psych-3.3.2/lib:~/tmp-ruby/root/usr/share/ruby/vendor_ruby/json-2.5.1/lib
```

Run with:

```bash
LD_LIBRARY_PATH="$LD_LIBRARY_PATH" RUBYLIB="$RUBYLIB" "$RUBY" -e '<your script>'
```

For ERB rendering specifically:

```ruby
require 'erb'
src = File.read('partials/slurm_discovery.erb')
puts ERB.new(src, trim_mode: "-").result(binding)
```

This renders the partial against your live Slurm controller without going through OOD. Fastest way to debug "is my Slurm query returning what I think." Must run on a login node (controller access).

The original setup write-up lives in `TESTING_REPORT.md` at the root of the `TuftsRT/OpenComposer` repo — read it once you've cloned that repo.

### The `DEBUG_*` flag pattern

The `gpu_discovery.erb` partial in `javi_jupyter` (and copies in other apps) starts with:

```ruby
DEBUG_GPU_DISCOVERY = false
```

Flip to `true`, render once, and the partial dumps every intermediate hash to `~/javi_gpu_debug.log`. Errors are swallowed so the form keeps rendering. **Adopt the same pattern in any new ERB partial:** a flag at the top, a `begin/rescue/nil` block that writes a structured dump to a file in `~/`. Don't `puts` — the form is streaming HTML.

Combined with frontend `?debug=1`, you can debug the whole request lifecycle: server-side state in a log file, client-side state in browser console.

---

## Layer 3 — Submit a real Slurm job

Even with a perfect-looking form and correct API JSON: the `gres` your form sends and the `gres` Slurm expects can differ silently (case, with/without `:`, wrong `--constraint=`).

After form testing:

1. Submit for real from OnDemand.
2. Wait for `RUNNING` (or a clear failure).
3. SSH into the running node (jobmonitor lets you click in) and verify the environment.
4. Use the app inside the session — don't just confirm the job started, confirm the *thing the user wants to do* works.

This is where regressions hide.

---

## Pre-commit checklist

- [ ] Form renders without 500.
- [ ] Browser console has no red errors with `?debug=1`.
- [ ] The specific change works.
- [ ] Edge case: GPU type with no available nodes shows the warning.
- [ ] Edge case: switch GPU↔CPU partitions, max values update.
- [ ] Real job submission enters `RUNNING`.
- [ ] If you touched a parser regex, run it against a few real Slurm outputs.

Skipping this list to "save time" always loses more time later.

Move on to **[04_git_workflow.md](04_git_workflow.md)**.
