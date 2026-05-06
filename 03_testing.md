# 03 — Testing

Almost every painful day I had came from skipping or shortcutting testing. AI agents are extremely good at making code that *looks* right and is broken in a non-obvious way. This chapter is the single most important one in the guide.

There are three layers to testing OnDemand work:

1. **Render the launch form / hit the API in a browser** to see the actual user-facing result.
2. **Run backend logic in isolation** (Ruby in the terminal) to verify Slurm queries, ERB rendering, parsing.
3. **Submit a real Slurm job** and verify it actually starts and the session works end-to-end.

Skipping layer 3 is how you ship a form that looks right but fails at `sbatch` time because of a `gres` name mismatch.

---

## Layer 1 — Browser testing

### Manually

1. Open the Tufts OnDemand dashboard: <https://ondemand-p01.pax.tufts.edu/pun/sys/dashboard>. Log in with your UTLN + Duo push.
2. **Sandbox Apps** menu → click your app's tile (every directory in `/cluster/home/<utln>/ondemand/prod/<app>/` shows up here automatically).
3. The launch form appears. Fill it out, submit, watch what happens. Open browser DevTools (F12) and watch the Console + Network tabs.

The "dev" URL pattern is useful for OpenComposer-style apps:

- `https://ondemand-p01.pax.tufts.edu/pun/sys/dashboard/batch_connect/dev/<app-name>` for batch_connect launch forms.
- `https://ondemand-p01.pax.tufts.edu/pun/dev/<app-name>` for passenger apps (jobmonitor, systemstatus, OpenComposer).

The `/dev/` paths re-render from the source directory every request — no caching. Use these while iterating.

### Frontend `?debug=1` flag

For form pages, append `?debug=1` (e.g., `.../batch_connect/dev/javi_jupyter/?debug=1`) to enable verbose `console.log` output in the browser console. The `form.js` files in this repo all check for `URLSearchParams` containing `debug` and gate logging on it. Add the same pattern to any new app you build — three lines of code, saves hours.

### Automated browser testing with Playwright MCP

This is the unlock. Once you've installed the Playwright MCP on your **laptop's** Codex/Claude (see [02_ai_agents.md](02_ai_agents.md)), the agent can drive a Chromium browser for you.

#### My workflow

1. Cluster-side agent finishes editing a file.
2. Ask it: *"Write me a prompt for my laptop AI tester. The tester has Playwright MCP. The cluster URL to test is `<dev URL>`. The tester needs to verify [specific behavior]. Include what to click, what to look for in the console, and which API calls to inspect in the network tab."*
3. The cluster agent prints a multi-paragraph testing prompt.
4. Copy that prompt into the **laptop** terminal where Codex/Claude is running.
5. Add this preamble:

   > *"Please use Playwright MCP to navigate to `<URL>`. I will log in manually first — pause after navigating and wait for me to confirm I've completed Duo before you continue. Then proceed with the test plan below: [paste prompt]"*

6. The agent navigates. You manually do the Duo push (one human-required step). You tell the agent "logged in, continue."
7. The agent clicks around, takes screenshots, reads console logs, and reports back.

You can watch the browser window the whole time, which is mesmerizing the first few times.

#### One-liner to install

```bash
codex mcp add playwright npx "@playwright/mcp@latest"
# or
claude mcp add playwright npx "@playwright/mcp@latest"
```

---

## Layer 2 — Backend logic in isolation

### The problem: no Ruby on the cluster

Open OnDemand uses Ruby ERB heavily, but the login nodes don't have Ruby in `$PATH` by default. So when your AI agent says *"let me test this ERB by running it in a `ruby` REPL"* — it can't.

### The solution: portable Ruby in your home directory

I extracted a portable Ruby from Rocky 9 RPMs (Ruby 3.0.7 + the `psych` YAML and `json` gems + their shared libs) and stashed it under my home directory. About 30 MB total. You'll need to do the same setup once.

**You won't have access to my copy** — my home is `/cluster/home/jlavea01/...` and yours will be `/cluster/home/<your-utln>/...`. Two options:

1. **Ask me (or whoever handed you this guide) to `tar` up our `.tmp-ruby/` directory** and untar it into your home. Fastest.
2. **Follow the recipe** to extract a fresh portable Ruby from Rocky 9 RPMs (`rpm2cpio` + `cpio -idmv` on `ruby`, `ruby-libs`, `rubygem-psych`, `rubygem-json`, then patch `LD_LIBRARY_PATH`). Any LLM can walk you through this in 10 minutes.

Once it lives at, say, `~/tmp-ruby/`, the env vars to source look like:

```bash
RUBY=~/tmp-ruby/root/usr/bin/ruby
LD_LIBRARY_PATH=~/tmp-ruby/root/usr/lib64
RUBYLIB=~/tmp-ruby/root/usr/share/ruby/vendor_ruby/psych-3.3.2/lib:~/tmp-ruby/root/usr/share/ruby/vendor_ruby/json-2.5.1/lib
```

(Adjust paths to wherever you actually extracted the RPMs; gem version numbers may drift.)

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

This renders the partial against your live Slurm controller and prints the output, **without going through the OOD web stack**. It is the fastest way to debug "is my Slurm query returning what I think."

The original write-up of this technique lives in `TESTING_REPORT.md` at the root of the `TuftsRT/OpenComposer` repo on GitHub — read it once you've cloned that repo, the troubleshooting log is useful.

### The `DEBUG_GPU_DISCOVERY` flag pattern

In `javi_jupyter/partials/gpu_discovery.erb` (and the copies in other apps) there's a constant near the top:

```ruby
DEBUG_GPU_DISCOVERY = false
```

Flip it to `true`, render the form once, and the partial dumps every intermediate hash (`@detected_types`, `@user_partitions`, per-partition cores/memory/hours, `@unavailable_gpu_types`) to `~/javi_gpu_debug.log`. Errors are swallowed so the form keeps rendering even if the debug write fails.

This is the single most useful debug tool I built. Adopt the same pattern in any new ERB partial:

- A constant flag at the top.
- A `begin / rescue / nil` block that writes a structured dump to a file in `~/`.
- Don't `puts` — the form is already streaming HTML, your debug output will land inside the page.

Combined with the frontend `?debug=1`, you can debug the full request lifecycle: server-side state in `javi_gpu_debug.log`, client-side state in browser console.

---

## Layer 3 — Submit an actual Slurm job

Even if the form looks perfect and the API returns the right JSON, you have not tested it until a job actually starts. The `gres` name your form sends and the `gres` name Slurm expects can differ silently (lowercase vs. uppercase, with or without a `:`, with the wrong `--constraint=`).

After form testing:

1. Submit the form for real from the OnDemand UI.
2. Wait for it to enter `RUNNING` (or fail with a clear reason).
3. SSH into the running node (jobmonitor lets you click into the node) and verify the environment is what you expected.
4. Use the app inside the session (open Workbench, run a notebook, etc.) — don't just confirm the job started; confirm the *thing the user actually wants to do* works.

This is where regressions hide.

---

## Bonus — testing without Ruby: the `.erb` rendering trick

If you don't want to use the portable Ruby, an alternative is to render ERB through a script that runs inside an actual Slurm job on a compute node (where Ruby is available via the OOD Passenger app's bundled stack). Slower than `.tmp-ruby`, but works as a fallback. Most of the time, just use the portable Ruby.

---

## Pre-commit check list

Before I push any change to one of these apps, I tick through:

- [ ] Form renders without 500.
- [ ] Browser console has no red errors with `?debug=1`.
- [ ] The specific scenario I changed works (clicked the new dropdown, saw the new value, etc.).
- [ ] Edge case: pick a GPU type with no available nodes — does the warning show?
- [ ] Edge case: switch between GPU and CPU partitions — do max values update?
- [ ] Submit a real job and confirm it enters `RUNNING`.
- [ ] If I touched a parser regex, run the parser standalone with a few real Slurm outputs.

When I skip this list to "save time" I always lose more time later.

Move on to **[04_git_workflow.md](04_git_workflow.md)**.
