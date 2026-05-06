# 04 — Git, GitHub, branches, PRs

If you don't already know git basics, spend an hour with the Pro Git book chapter 2 first. This chapter covers the **Tufts-specific bits** and the patterns I wish someone had shown me.

---

## SSH key for GitHub

Same idea as the cluster SSH key (chapter 01). You can reuse one key for both, or generate per-service.

1. `ssh-keygen -t ed25519 -C "you@example.com"` (or reuse).
2. Paste `cat ~/.ssh/id_ed25519.pub` into GitHub → Settings → SSH and GPG keys → New SSH key.
3. Test: `ssh -T git@github.com` → `Hi <username>!`.

Add a key on **both** your laptop and the cluster — you'll push from either.

### Multiple GitHub accounts

If you juggle a personal and Tufts identity, use `~/.ssh/config`:

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

Then clone with `git clone git@github.com-tufts:TuftsRT/some-repo.git`. The hostname maps to the right key.

---

## The two GitHub orgs

- **`TuftsRT`** — official Tufts Research Technology org. Production repos (`tufts-jobmonitor`, `OpenComposer`, `ood-resource-discovery`, etc.). Ask RT staff to add you.
- **Your personal account** — fork-style copies for in-progress work that isn't ready upstream. Useful for showing your supervisor what you've been doing without polluting production.

---

## Branch + PR flow

Don't push directly to `main`:

```bash
git checkout main && git pull
git checkout -b fix/scontrol-regex-edge-case
# ... work, test, commit ...
git push -u origin fix/scontrol-regex-edge-case
# Open PR via GitHub UI or `gh pr create`
```

**Commit early and often** — every coherent change gets its own commit. Easier to bisect, roll back, and review.

**Don't let an AI agent batch six changes before you commit.** If the third broke things you'll be untangling unrelated edits to find the culprit.

---

## What to commit, what to `.gitignore`

**Don't commit:**
- `.tmp-ruby/` and other large binary blobs
- `~/javi_gpu_debug.log`-style debug artifacts
- AI session summaries unless polished and useful to others
- Anything with secrets (API keys, Codex `auth.json`, etc.)
- `.vscode/sftp.json` — has your username and key path

`git status -s` before every commit. Read every file you're about to add.

---

## The commands I actually use

```bash
git status                        # what changed (run constantly)
git add <file>                    # stage one file
git add .                         # stage everything in CWD
git commit -m "message here"      # commit staged changes
git log                           # commit history (--oneline -10 for compact)
git branch                        # list branches
git checkout -b "branch-name"     # new branch + switch to it
```

`git status` and `git log --oneline -10` answer 80% of "wait what state am I in." Run them constantly.

Anything fancier (`git diff`, `git show`, `git blame`, `git reset`) — ask the AI when you need it. Don't memorize what you won't use.

---

## Forks with an upstream (e.g., OpenComposer)

OpenComposer is a fork of RIKEN-RCCS. To pull upstream updates:

```bash
git remote add upstream git@github.com:RIKEN-RCCS/OpenComposer.git
git fetch upstream
git merge upstream/main
# resolve conflicts, push to your origin (TuftsRT)
```

`git remote -vv` should show both `origin` and `upstream`.

---

## Before destructive commands

Before `git reset --hard`, `git push --force`, or `git branch -D`, **stop and ask the AI**: *"What will I lose? Is there a safer way to get to the state I want?"* It will tell you and usually suggest a non-destructive alternative.

Move on to **[05_models_and_tools.md](05_models_and_tools.md)**.
