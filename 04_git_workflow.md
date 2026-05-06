# 04 — Git, GitHub, branches, PRs

You will use git constantly. If you don't already know the basics, spend an hour with a tutorial (the official "Pro Git" book chapter 2 is short and free) before reading this chapter — it covers things I am assuming you know.

This chapter focuses on the **Tufts-specific bits** and the few patterns I wish someone had shown me up front.

---

## SSH key for GitHub

Same idea as the SSH key you generated for the cluster (chapter 01) — different key per service is good hygiene, but you can also reuse one key for both. The GitHub setup:

1. Generate a key (or reuse your cluster one): `ssh-keygen -t ed25519 -C "you@example.com"`
2. Copy the **public** key (the `.pub` file): `cat ~/.ssh/id_ed25519.pub` and copy the output.
3. GitHub → Settings → SSH and GPG keys → "New SSH key" → paste, save.
4. Test: `ssh -T git@github.com` — you should see `Hi <your-username>!`.

You need this SSH key on **both** your laptop and the cluster (so you can `git push` from either). Generate one on each, add both to GitHub.

If you're managing multiple GitHub identities (personal + Tufts org), use `~/.ssh/config`:

```ssh-config
Host github.com-tufts
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_tufts

Host github.com-personal
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_personal
```

Then clone with `git clone git@github.com-tufts:TuftsRT/some-repo.git` instead of `git@github.com:...`. The hostname in the SSH config maps to the right key.

---

## The two GitHub orgs you care about

- **`TuftsRT`** — the official Tufts Research Technology org. Production-ready repos live here (e.g., `TuftsRT/tufts-jobmonitor`, `TuftsRT/OpenComposer`, `TuftsRT/ood-resource-discovery`). You'll need to be added as a member by RT staff.
- **`javier-la200426`** (your personal account) — fork-style copies for in-progress work that isn't ready to merge upstream (e.g., `javier-la200426/ansys-dynamic`, `javier-la200426/jupyterNotebookDynamicFormYml`). Useful for showing your supervisor what you've been doing without polluting the production repo.

---

## Branch + PR flow

Don't push directly to `main`. The pattern for any non-trivial change:

```bash
# Start clean
git checkout main
git pull

# Branch
git checkout -b fix/scontrol-regex-edge-case

# Work, commit incrementally
git add lib/job_parser.rb
git commit -m "Fix scontrol regex capturing adjacent key=value pairs"
# ... more commits ...

# Push the branch
git push -u origin fix/scontrol-regex-edge-case

# Open a PR via the GitHub web UI, or `gh pr create` if you have the gh CLI
```

I commit early and often — every coherent change is its own commit. Easier to bisect later, easier to roll back, easier for a reviewer to understand.

---

## What to commit, what to `.gitignore`

**Commit:**
- Code changes
- Config that is meant to be shared
- Documentation, READMEs, CHANGELOGs
- Lock files for shared dependencies

**Don't commit (add to `.gitignore`):**
- `.tmp-ruby/` and other big binary blobs you set up locally
- `~/javi_gpu_debug.log`-style debug artifacts
- AI session summaries (`summary.md`, `session-notes.md`) unless they're polished and useful to others
- Anything with secrets (API keys, the `auth.json` files Codex creates, etc.)
- `.vscode/sftp.json` — has your username and key path

A `.gitignore` line for each. When in doubt, `git status -s` before committing and read every file you're about to add.

---

## Incremental commits — the rhythm

The rhythm I land on after a session of AI-assisted work:

1. Agent makes a change.
2. I test it (chapter 03).
3. If it works: `git add <files>` + `git commit -m "<short message>"`. Push periodically.
4. If it doesn't work: discuss with the agent, iterate, then commit only when actually green.

Don't let an AI agent batch up six changes before you commit. If the third change broke things you'll be sorting through unrelated edits to find the culprit. Commit-as-you-go is the discipline that pays off.

---

## Useful commands you'll forget

```bash
# Compare your branch to main
git diff main...HEAD

# Show the file at an older commit
git show <hash>:path/to/file

# Find which commit introduced a line
git log -S "the_exact_string" -- path/to/file

# Undo your last commit but keep the changes staged
git reset --soft HEAD~1

# Find who/why a line was written
git blame path/to/file
```

`git log --oneline -20` is the most-used command in my workflow. Always do it before you start touching a repo for the first time — the recent commits tell you the story.

---

## Hostnames and remotes — when one repo lives in two places

Some of these projects have weird ancestry: `OpenComposer` is a fork of RIKEN-RCCS's upstream. To pull updates from upstream while pushing your changes to TuftsRT:

```bash
git remote add upstream git@github.com:RIKEN-RCCS/OpenComposer.git
git fetch upstream
git merge upstream/main
# ... resolve conflicts, then push to your origin (TuftsRT) as usual
```

Verify with `git remote -vv` — should show `origin` (TuftsRT) and `upstream` (RIKEN-RCCS).

---

## When in doubt

`git status` and `git log --oneline -10` answer 80% of "wait what state am I in." Run them constantly.

If you're about to do something destructive (`git reset --hard`, `git push --force`, `git branch -D`), **stop and ask the AI**: *"I'm about to run `git reset --hard origin/main`. What will I lose? Is there a safer way to get to the state I want?"* It will tell you and often suggest a non-destructive alternative.

Move on to **[05_models_and_tools.md](05_models_and_tools.md)**.
