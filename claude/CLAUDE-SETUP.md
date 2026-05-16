# Claude Code Setup for This Repo

How to get the same Claude Code experience as the original developer —
auto-approved commands, the `awtosafe` keyword, and session-persistent permissions.

---

## 1. Install Claude Code

```bash
npm install -g @anthropic-ai/claude-code
```

Or via the VS Code extension: search **Claude Code** in the Extensions panel.

---

## 2. Open this folder in VS Code

Claude Code reads `.claude/settings.json` from the project root automatically
when you open a folder. No extra configuration needed.

---

## 3. How the allow list works

This repo uses a two-file system so the master list is easy to edit and version-controlled:

| File | Purpose |
|------|---------|
| `.vscode/settings.example.json` | **Master list** — edit this to add/remove allowed commands |
| `.claude/settings.json` | Live settings read by Claude Code — auto-synced from the master list |

The `.claude/settings.json` contains a `SessionStart` hook that copies the
permissions from `.vscode/settings.example.json` into itself each time a new
Claude Code session starts. So edits to the master list take effect on the next
session.

**To add a command permanently:**
1. Open `.vscode/settings.example.json`
2. Add your rule to the `permissions.allow` array, e.g. `"Bash(west flash *)"`
3. Save — it will be active from the next Claude Code session

---

## 4. The `awtosafe` keyword

Tell Claude Code:

> **awtosafe** `your-command-here`

Claude will add it to `.vscode/settings.example.json` (the master list) AND
apply it immediately to the live `.claude/settings.json` so you don't have to
wait for the next session.

Example:

> awtosafe `west flash --runner openocd`

---

## 5. The allow list covers

- `git *` — all git operations
- `west *` — Zephyr build, flash, debug
- `cmake *`, `ninja *`, `make *` — build tools
- `arm-none-eabi-* *` — ARM toolchain (size, nm, objdump…)
- `gh *` — GitHub CLI
- `python3 *`, `python *` — scripts and venv
- `tio *`, `lsusb *`, `ls /dev/serial/*` — serial and USB
- `STM32_Programmer_CLI *`, `st-flash *` — STM32 flashing
- `ip *`, `ping *`, `candump *`, `cansend *` — networking and CAN
- `grep *`, `awk *`, `find *`, `timeout *` — common shell tools
- …and more — see `.vscode/settings.example.json` for the full list

---

## 6. One-session lag explained

When you edit `.vscode/settings.example.json` and save:

```
Edit master list
      │
      ▼
Restart / new Claude Code session
      │
      ▼
SessionStart hook fires → syncs into .claude/settings.json
      │
      ▼
New permissions active for this session
```

If you need a command active **right now** without restarting, just say
`awtosafe <command>` and Claude will apply it immediately.
