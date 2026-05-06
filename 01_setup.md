# 01 — Local setup

You can't start the real work until this is wired up. Block off a half day on day one.

## What you're setting up

1. **Tufts VPN** — required from off-campus.
2. **SSH key** for the cluster (so you don't type your password 50 times a day).
3. **VS Code + SFTP** so editing a file locally auto-uploads it to the cluster on save.

---

## 1. Tufts VPN

Install: <https://access.tufts.edu/vpn>. Required to reach the cluster, the OnDemand portal, or anything `*.pax.tufts.edu` from off-campus. Skip if you're on `Tufts_Secure` Wi-Fi.

---

## 2. SSH key for the cluster

An SSH key is a pair of files: a **private key** (stays on your laptop, never share) and a **public key** (you append it to `~/.ssh/authorized_keys` on the cluster). When you SSH, your laptop proves it owns the private key and the cluster checks for the matching public key — no password prompt.

The cleanest path: ask Codex/Claude on your laptop *"Generate me an ed25519 SSH key for `<utln>@login-prod.pax.tufts.edu` and walk me through installing the public half on the cluster."* The steps are the same on every Mac/Linux box.

Once installed, your daily SSH command looks like:

```bash
ssh -t <utln>@login-prod.pax.tufts.edu -p 22 -i ~/.ssh/id_ed25519_tufts
```

Drop a `Host pax` block into `~/.ssh/config` (ask the AI) so you can just type `ssh pax`.

### Shell history search — `Ctrl+R`

You don't need to retype that long SSH command. Hit **`Ctrl+R`** in your terminal (Mac and Linux, in `bash` or `zsh`), start typing — `ssh` is enough — and the prompt fills in with the most recent matching command from your history. Hit `Ctrl+R` again to step further back. Enter to run, arrow keys to edit first.

I use this every time I SSH. Type `Ctrl+R` → `ssh` → Enter, done. Same trick works for `git push origin <long-branch-name>`, `codex mcp add ...`, anything you've run recently. Windows PowerShell uses a different shortcut (typically `F8` or arrow-up filtering); ask the AI on Windows.

---

## 3. VS Code + SFTP — the real unlock

Editing on the cluster with `nano`/`vim` is a slog. With this setup you edit on your laptop, hit save, and the cluster has the new version in under a second.

### Install

1. Install **VS Code** (or **Cursor**).
2. Install the **SFTP** extension by *Natizyskunk* (the maintained fork; not the abandoned `liximomo` one).

### Configure

Cmd+Shift+P → `>SFTP: Config` → Enter. Replace the generated `<your-folder>/.vscode/sftp.json` with:

```json
{
  "name": "pax-cluster",
  "host": "login-prod.pax.tufts.edu",
  "protocol": "sftp",
  "port": 22,
  "username": "<your-utln>",
  "privateKeyPath": "/Users/<your-mac-user>/.ssh/id_ed25519_tufts",
  "remotePath": "/cluster/home/<your-utln>/ondemand/prod",
  "uploadOnSave": true,
  "useTempFile": false,
  "openSsh": false,
  "ignore": ["**/.vscode/**", "**/.git/**", "**/node_modules/**"]
}
```

Edit `username`, `privateKeyPath`, and `remotePath` to your values. The `ondemand/prod` path is the magic folder — every subdirectory becomes a sandbox app in your OnDemand dashboard.

The key knob is `"uploadOnSave": true`. Every save in VS Code is mirrored up to the cluster instantly.

### Operations you'll use

- **Edit locally → save → cluster updates.** Automatic.
- **Pull a cluster-side file down to your laptop.** Cmd+Shift+P → `SFTP: Sync Remote -> Local`, pick `pax-cluster`, choose the folder. Use this whenever you (or an AI agent on the cluster) created a new file there that you want to edit locally.

> ### ⚠ Only ever use `Sync Remote -> Local`. Never `Sync Local -> Remote`.
>
> `Sync Local -> Remote` mirrors your laptop **onto** the cluster: it overwrites every cluster file with your local copy (even if the cluster's is newer) and deletes cluster files that don't exist locally. An AI agent on the cluster is constantly creating files that aren't on your laptop yet — one wrong click can wipe hours of work. There is no undo.
>
> If you genuinely need to push a batch of local files up, save them one-by-one (each save uploads via `uploadOnSave`), or use `scp`/`rsync` from a terminal.

---

Move on to **[02_ai_agents.md](02_ai_agents.md)**.
