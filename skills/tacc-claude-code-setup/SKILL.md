---
name: tacc-claude-code-setup
description: >-
  Install and run Claude Code on a TACC supercomputer login node. Use when the
  user asks how to install Node.js and/or Claude Code on TACC (Vista, Frontera,
  Stampede3, Lonestar6), gets npm global-install permission errors, hits the
  postinstall "allow-scripts" warning, or asks how to authenticate Claude Code
  over SSH. Triggers on "claude code on tacc", "install claude code", "node on
  tacc", "npm install -g permission", "EACCES npm", "claude code 설치",
  "TACC에서 claude", "nvm tacc", "claude code login node".
---

# Claude Code on TACC (login-node setup)

How to install and run **Claude Code** (`@anthropic-ai/claude-code`) on a TACC
login node. Verified on **Vista (Arm aarch64)** on 2026-07-05; the flow is the
same on other TACC systems (Frontera/Stampede3/Lonestar6 are x86). See
[[tacc-hpc-guide]] for login (SSH+MFA), filesystems, and Slurm.

## 0. Where to run — login node only ⚠️

Claude Code needs outbound HTTPS to `api.anthropic.com`, and on TACC **only
login nodes have internet**. Compute nodes (batch jobs / `idev`) are firewalled
off the public network, so `claude` will fail to connect there. Install **and
run** Claude Code on the login node. Do not launch it inside an `sbatch` job.

(Claude Code is an interactive coding assistant, not a compute job — running it
on the login node is the intended use and does not violate the "no heavy compute
on login nodes" rule. Keep actual builds/training on compute nodes.)

## 1. Get Node.js (18+)

TACC does **not** ship a Node.js Lmod module on every system — on Vista
`module avail node` returns nothing as of 2026-07-05. First check whether Node is
already on `PATH`; if not, install a user-local copy with **nvm** (no admin/sudo
needed).

```bash
# a) Is Node already available?
node --version && which node        # want v18+

# b) If not, search Lmod just in case (name varies by system)
module spider node ; module keyword node javascript

# c) Fallback that always works — nvm into your home dir (login node has internet)
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
source ~/.bashrc                    # or: source ~/.nvm/nvm.sh
nvm install --lts
node --version                      # confirm v18+
```

> Vista is **Arm (aarch64)** — nvm and npm fetch the correct arm64 Node build
> automatically, so no special flags are needed.

## 2. Point npm's global prefix at a writable dir ⚠️

On TACC the default global prefix is not user-writable, so `npm install -g`
fails with `EACCES`. Redirect it to your home (or `$WORK`) before installing.
Skip this step if you installed Node via nvm — nvm already uses a writable
per-version prefix.

```bash
mkdir -p $HOME/.npm-global
npm config set prefix "$HOME/.npm-global"
echo 'export PATH=$HOME/.npm-global/bin:$PATH' >> ~/.bashrc
source ~/.bashrc
```

> `$HOME` has a small quota. If you install many global npm tools, point the
> prefix at `$WORK` instead (`npm config set prefix "$WORK/.npm-global"` and add
> that `bin` to `PATH`). Do **not** use `$SCRATCH` — it is purged periodically.

## 3. Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
claude --version                    # confirm it's on PATH
```

**Expected warning (safe to proceed):** TACC's npm may block package install
scripts, so you'll see:

```
npm warn allow-scripts 1 package has install scripts not yet covered by allowScripts:
npm warn allow-scripts   @anthropic-ai/claude-code@... (postinstall: node install.cjs)
```

Claude Code still runs — the JS entrypoint is self-contained and the CLI works
without the postinstall. Only if `claude` errors at runtime with a missing
native binary (e.g. ripgrep) do you need to allow the script and reinstall:

```bash
npm approve-scripts @anthropic-ai/claude-code   # if your npm exposes it
npm install -g @anthropic-ai/claude-code
```

The `npm notice New minor version of npm available` line is informational —
ignore it.

## 4. Authenticate (over SSH)

First run of `claude` prompts for auth. Two options:

| Method | How it works on a remote login node |
|--------|-------------------------------------|
| **Subscription (Pro/Max)** | Claude Code prints a login URL. Open it in your **local** browser, approve, copy the returned code, and paste it back into the SSH terminal. |
| **API key** | `export ANTHROPIC_API_KEY="sk-ant-..."` before launching (add to `~/.bashrc` to persist). Pay-as-you-go billing. |

```bash
claude          # launch; follow the browser-code paste flow, or use the API key
```

## 5. Recommended working setup

- **Run from `$WORK` or `$SCRATCH`**, not `$HOME` — that's where your code/data
  live and where `$HOME` quota won't bite. `cd $WORK/myproject && claude`.
- Node/nvm and the npm prefix persist across logins via `~/.bashrc`, so setup is
  one-time; later sessions just need `cd <project> && claude`.

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| `npm ERR! EACCES` on `-g` install | Step 2 — set `npm config set prefix` to a writable dir |
| `claude: command not found` | `echo $PATH \| tr ':' '\n' \| grep -E 'npm-global\|nvm'`; re-`source ~/.bashrc`; check `ls $HOME/.npm-global/bin/` |
| `module avail node` finds nothing | Expected on Vista — use nvm (Step 1c) |
| Connection/timeout when running `claude` | You're on a **compute node** — Claude Code only works on login nodes (Step 0) |
| `node --version` shows < 18 | `nvm install --lts && nvm use --lts` |
| Missing native binary at runtime | Allow the postinstall script and reinstall (Step 3) |

## References

- Claude Code: https://docs.anthropic.com/en/docs/claude-code
- nvm: https://github.com/nvm-sh/nvm
- TACC docs: https://docs.tacc.utexas.edu/
- Companion skill: `tacc-hpc-guide` (login, filesystems, Slurm)
