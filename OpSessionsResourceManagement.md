# OpenClaw on WSL — Stop, Start & Resource Management Guide

> **Environment:** OpenClaw running inside a sandbox, inside WSL2, on Windows.

---

## Table of Contents

1. [Understanding the Layers](#1-understanding-the-layers)
2. [Safely Stopping OpenClaw](#2-safely-stopping-openclaw)
3. [Shutting Down WSL to Free RAM](#3-shutting-down-wsl-to-free-ram)
4. [Reconnecting Later](#4-reconnecting-later)
5. [One-Liner Shortcut](#5-one-liner-shortcut)
6. [Common Mistakes & Fixes](#6-common-mistakes--fixes)
7. [Quick Reference Cheatsheet](#7-quick-reference-cheatsheet)

---

## 1. Understanding the Layers

Your setup has three nested layers. Each must be handled separately:

```
Windows
  └── WSL2 (Ubuntu)
        └── Sandbox (Docker container)
              └── OpenClaw Gateway + TUI
```

Simply closing the terminal or typing `/exit` in the TUI **does not free RAM** — the processes keep running in the background until explicitly stopped.

---

## 2. Safely Stopping OpenClaw

### Step 1 — Exit the TUI
Inside OpenClaw TUI, type:
```
/exit
```
This brings you back to the sandbox shell.

### Step 2 — Exit the sandbox
```bash
exit
```
This brings you back to the WSL shell.

### Step 3 — Kill the gateway process

> ⚠️ **Known bug in OpenClaw 2026.x:** `openclaw gateway stop` reports _"Gateway service disabled"_ but the process keeps running and the port stays bound. Do not rely on it alone.

Kill the gateway properly:
```bash
pkill -f openclaw-gateway
```

Verify it is gone:
```bash
ps aux | grep openclaw-gateway
```

Verify the port is free:
```bash
ss -tlnp | grep 18789
```
No output = fully stopped.

### Step 4 — Stop Docker sandbox containers (if used)
```bash
docker ps -q | xargs -r docker stop
```

---

## 3. Shutting Down WSL to Free RAM

### ⚠️ Critical: Run this on Windows, NOT inside WSL

WSL keeps running in the background even after you close the terminal. The `vmmem` process in Windows Task Manager holds onto allocated RAM until WSL is fully shut down.

**Run this in PowerShell or CMD on Windows (not inside WSL):**
```powershell
wsl --shutdown
```

After this, `vmmem` disappears from Task Manager and all RAM is returned to Windows.

### How to tell if you are in WSL vs Windows

| Prompt looks like | You are in |
|---|---|
| `qasim@DESKTOP-VJB33QJ:~$` | WSL (Linux) |
| `qasim@DESKTOP-VJB33QJ:/mnt/c/...` | WSL (Linux) — even though you navigated to a Windows folder |
| `C:\Users\qasim>` | Windows CMD |
| `PS C:\Users\qasim>` | Windows PowerShell |

### Ways to run `wsl --shutdown` correctly

**Option 1 — Open PowerShell from Start menu:**
```powershell
wsl --shutdown
```

**Option 2 — From inside WSL using cmd.exe:**
```bash
cmd.exe /c "wsl --shutdown"
```

**Option 3 — Windows Run dialog (`Win + R`):**
```
wsl --shutdown
```

### ❌ Do NOT do this
```bash
# Wrong — this installs a completely unrelated Linux package
sudo apt install wsl
```

---

## 4. Reconnecting Later

Everything is saved on disk in `~/.openclaw/` — config, memory files, sessions. Nothing is lost on shutdown.

### Step 1 — Open WSL terminal
Open Windows Terminal or search **Ubuntu** in the Start menu.

### Step 2 — Start the gateway
```bash
openclaw gateway
```
Or if you installed the daemon:
```bash
openclaw gateway restart
```

### Step 3 — Open the TUI
```bash
openclaw tui
```

That's it — everything picks up exactly where you left off.

---

## 5. One-Liner Shortcut

Add an alias to your `~/.bashrc` so you can start everything with one command:

```bash
# Add this line to ~/.bashrc
alias oc-start='openclaw gateway & sleep 2 && openclaw tui'
```

Reload and use it:
```bash
source ~/.bashrc
oc-start
```

---

## 6. Common Mistakes & Fixes

### `openclaw gateway stop` does not actually stop the process
```bash
# This is a known bug — use pkill instead
pkill -f openclaw-gateway
```

### `wsl --shutdown` says "command not found" inside WSL
You are running it in the wrong place. Open a **Windows** PowerShell window and run it there.

### Config warning about `qqbot` plugin
```
plugins.entries.qqbot: plugin not installed
```
This is harmless. Either install it:
```bash
openclaw plugins install @openclaw/qqbot
```
Or remove the `qqbot` entry from your OpenClaw config if you do not need it.

### WSL using too much RAM at idle
Create `C:\Users\YourName\.wslconfig` on Windows with memory limits:
```ini
[wsl2]
memory=4GB
processors=2
swap=2GB
```
Then restart WSL:
```powershell
wsl --shutdown
```

---

## 7. Quick Reference Cheatsheet

### Stopping (full shutdown)

| Step | Where | Command |
|---|---|---|
| Exit TUI | OpenClaw TUI | `/exit` |
| Exit sandbox | Sandbox shell | `exit` |
| Kill gateway | WSL shell | `pkill -f openclaw-gateway` |
| Stop Docker containers | WSL shell | `docker ps -q \| xargs -r docker stop` |
| Free all RAM | Windows PowerShell | `wsl --shutdown` |

### Starting (reconnect)

| Step | Where | Command |
|---|---|---|
| Open WSL | Windows | Open Ubuntu / Windows Terminal |
| Start gateway | WSL shell | `openclaw gateway` |
| Open TUI | WSL shell | `openclaw tui` |

---

*OpenClaw version tested: 2026.5.22 (a374c3a) on WSL2 Ubuntu*
