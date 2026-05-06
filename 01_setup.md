# 01 — Local setup

You cannot start the actual work until this is wired up. Block off a half day on day one and grind through it.

## What you are setting up

Three things, in order:

1. **Tufts VPN** — required to reach the cluster from anywhere off-campus.
2. **SSH access to the login node** with a key (so you don't type your password 50 times a day).
3. **VS Code (or Cursor) + SFTP plugin** so editing a file locally auto-uploads it to the cluster on save.

That last one is the unlock. Once it works you will edit code on your laptop with full IDE comforts and the cluster will see your changes in under a second.

---

## 1. Tufts VPN

If you are off-campus or on a non-Tufts network, install the Tufts VPN first. Without it, neither SSH nor the OnDemand portal will reach you.

Install link: <https://access.tufts.edu/vpn>

You will need to be on the VPN whenever you:
- SSH into the cluster
- Open the OnDemand portal at <https://ondemand-p01.pax.tufts.edu>
- Have Playwright drive a browser session to either of the above

If you are physically on Tufts Wi-Fi (`Tufts_Secure`), you can skip the VPN.

---

## 2. SSH key for the cluster

You don't strictly *need* a key — you can SSH with a password — but you'll be SSHing dozens of times a day and 2FA gets old fast. With a key it's one command.

### What an SSH key actually is (60-second version)

An SSH key is a pair of files: a **private key** (stays on your laptop, never share) and a **public key** (you paste this onto the cluster). When you SSH, your laptop proves it owns the private key without sending it across the wire; the cluster checks that the matching public key is on the approved list. Result: no password prompt.

You'll do this exact same dance with GitHub later (chapter 04).

### Generating and installing the key

The easiest path is to ask Codex or Claude Code to walk you through it on your laptop — say *"Generate me a new ed25519 SSH key for connecting to my Tufts cluster account `<utln>@login-prod.pax.tufts.edu` and walk me through copying the public key over."* The steps are the same on every Mac/Linux box and the AI will adapt to your shell, so I won't reproduce a stale recipe here. Mine ended up at `/Users/javierlaveaga/.ssh/id_ed25519_tufts`.

The one part to know by hand: once the key exists locally, the cluster needs the **public** half (the `.pub` file) appended to `~/.ssh/authorized_keys` in your cluster home. If you've never SSHed before, you'll need to do this first connect with your password, then add the key.

### Your daily SSH command

Once that's set up, my exact command is:

```bash
ssh -t jlavea01@login-prod.pax.tufts.edu -p 22 -i "/Users/javierlaveaga/.ssh/id_ed25519_tufts"
```

Substitute your UTLN and your private-key path. You can also drop most of those flags into `~/.ssh/config` so you can just type `ssh pax` — ask the AI to set that up for you too.

---

## 3. VS Code + SFTP — the killer setup

Editing files directly on the cluster with `nano` or `vim` works but is a slog. The setup below makes the cluster invisible: you edit on your laptop, hit save, and 0.5 seconds later the cluster has the new version.

### Install

1. Install **VS Code** (or **Cursor**, which is a VS Code fork with built-in AI — same SFTP setup).
2. In the Extensions panel, install the **SFTP** plugin by *Natizyskunk* (the maintained fork; the original by `liximomo` is abandoned). If you install the wrong one things still mostly work but the maintained one has fewer bugs.

### Configure

1. In VS Code, hit **Cmd+Shift+P** (Mac) / **Ctrl+Shift+P** (Linux/Windows).
2. Type `>SFTP: Config` and hit Enter.
3. It will create a file at `<your-folder>/.vscode/sftp.json`. Replace its contents with this template, edited for your username and key path:

```json
{
  "name": "pax-cluster",
  "host": "login-prod.pax.tufts.edu",
  "protocol": "sftp",
  "port": 22,
  "username": "jlavea01",
  "privateKeyPath": "/Users/javierlaveaga/.ssh/id_ed25519_tufts",
  "remotePath": "/cluster/home/jlavea01/ondemand/prod",
  "uploadOnSave": true,
  "useTempFile": false,
  "openSsh": false,
  "ignore": [
    "**/.vscode/**",
    "**/.git/**",
    "**/node_modules/**"
  ]
}
```

Things to change:
- `username` — your UTLN, not `jlavea01`.
- `privateKeyPath` — wherever your laptop's private key actually lives.
- `remotePath` — your equivalent path. `ondemand/prod` is the magic folder where any subdirectory becomes a sandbox app in your OnDemand dashboard automatically.

The key knob is `"uploadOnSave": true`. With that on, every save in VS Code is mirrored up to the cluster instantly.

### The two operations you'll use daily

- **Edit locally → save → cluster updates.** Automatic. Just save the file.
- **New file appeared on the cluster → pull it down to your laptop.** Cmd+Shift+P → `SFTP: Sync Remote -> Local`, pick `pax-cluster`, choose the folder. This happens whenever you (or an AI agent on the cluster) creates a new file via the cluster terminal that you then want to edit locally.

There is also `SFTP: Sync Local -> Remote` for the reverse direction (useful after a bulk local change). Use it sparingly — it's easy to overwrite cluster-side edits you forgot about.

### Workflow gotcha

The setup is one-way "live" only on save. If you're running an AI agent on the cluster *and* editing locally at the same time, both can write to the same file and you'll race. My rule: **only one writer per file at a time**, and pull remote changes before you start editing.

---

## 4. Open a terminal split

Once SFTP is wired up, my screen layout is:

- Left half: VS Code with the project open (local copy).
- Right half: a terminal SSHed into the cluster, where I run AI agents and Slurm commands.
- Background: a browser tab on the OnDemand portal for testing.

Move on to **[02_ai_agents.md](02_ai_agents.md)**.
