# awto-pod-33 — Zephyr Workspace Setup

Target board: **stm32h750b_dk** (ST Discovery Kit, STM32H750B-DK)

## What we've done so far

| Step | Command | What it does |
|------|---------|--------------|
| 1 | `pip install --user west` | Installs **west** globally to `~/.local/bin/west` — available in all projects, no activation needed. Skip if already installed (`west --version`). |
| 2 | `west init .` | Marks this folder as a west workspace. Creates `.west/config` pointing at `zephyr/west.yml` as the manifest. Clones the `zephyr` repo. |
| 3 | `west update` | Reads `zephyr/west.yml` and clones every module (STM32/NXP/Nordic HALs, mbedTLS, littlefs, CMSIS, hal_stm32 — the lot). This is the big download. |
| 4 | `west zephyr-export` | Registers Zephyr's CMake package in `~/.cmake/packages/` so any out-of-tree project can `find_package(Zephyr)` without hard-coded paths. |
| 5 | `python3 -m venv .venv && source .venv/bin/activate && west packages pip --install` | One-time setup: installs Zephyr's Python build deps (pyelftools, cryptography, intelhex…) into a project venv. Only needed once per workspace. |

## Workspace layout now

```
~/git/awto-pod-33/
├── .west/          # west config (this is what marks the workspace root)
├── .venv/          # Python venv (west, pyelftools, etc.)
├── zephyr/         # Zephyr RTOS source + samples + west.yml manifest
├── modules/        # All vendor HALs and 3rd-party libs cloned by `west update`
├── tools/          # Helper tools pulled in by manifest
└── bootloader/     # MCUboot etc. (if listed in manifest)
```

---

## Next steps

### 7. Install Python build dependencies

```bash
west packages pip --install
```

Installs Python packages Zephyr's build system needs (pyelftools, cryptography, intelhex, etc.) into the active venv.

### 8. Install the Zephyr SDK (toolchain)

The SDK provides `arm-zephyr-eabi-gcc` (the cross-compiler), QEMU, OpenOCD, etc.

Check first:

```bash
ls ~/zephyr-sdk-* 2>/dev/null
echo "ZEPHYR_SDK_INSTALL_DIR=$ZEPHYR_SDK_INSTALL_DIR"
```

If nothing prints, install via west (recommended):

```bash
west sdk install
```

This downloads the latest Zephyr SDK to `~/zephyr-sdk-<version>/` and registers it. It's ~1–2 GB.

Optional minimal install (only ARM Cortex-M targets, faster):

```bash
west sdk install -t arm-zephyr-eabi
```

### 9. First build

```bash
west build -b stm32h750b_dk zephyr/samples/hello_world
```

This produces `build/zephyr/zephyr.elf` (and `.bin`, `.hex`).

Useful flags:
- `-p always` — pristine build (wipes `build/` first; use after changing board or sample)
- `-d build-h750b` — custom build dir (lets you keep multiple boards side-by-side)

### 10. Flash to the STM32H750B-DK

The STM32H750B-DK has an on-board ST-LINK/V2-1. Plug it in via USB (CN1), then:

```bash
west flash
```

By default this uses OpenOCD (bundled with the SDK). To use ST's tool instead:

```bash
west flash --runner stm32cubeprogrammer
```

### 11. View serial output

Hello-world prints over the board's virtual COM port (ST-LINK VCP), default 115200 8N1:

```bash
tio /dev/ttyACM0          # or: minicom, picocom, screen
```

You should see:

```
*** Booting Zephyr OS build v4.x.x ***
Hello World! stm32h750b_dk
```

---

## Daily workflow cheat-sheet

`west` is installed globally (`~/.local/bin/west`) — no venv activation needed.

```bash
cd ~/git/awto-pod-33

# Sync manifest (when zephyr/west.yml changes upstream)
west update

# Build
west build -b stm32h750b_dk code

# Rebuild same target
west build

# Flash & monitor
west flash
tio /dev/serial/by-id/usb-STMicroelectronics_STLINK-V3_003400223137510E33333639-if02
```

## Board-specific notes: stm32h750b_dk

- Full Zephyr support: LCD, audio, Ethernet, microSD, camera, USB OTG, and more.
- Onboard ST-LINK/V3 for flashing and serial (no external probe needed).
- Use `west build -b stm32h750b_dk code` for all Zephyr apps.
- Use `west flash` to program via ST-LINK.
- Serial console on USART3 via STLink VCP — use by-id path, 115200 baud.

## Troubleshooting

- **`CMake Error: Zephyr SDK not found`** → SDK not installed or `ZEPHYR_SDK_INSTALL_DIR` not set. Run `west sdk install` or `export ZEPHYR_SDK_INSTALL_DIR=~/zephyr-sdk-<ver>`.
- **`Permission denied: /dev/ttyACM0`** → add yourself to `dialout`: `sudo usermod -aG dialout $USER` then re-login.
- **`west: command not found`** → venv not activated. `source .venv/bin/activate`.
- **OpenOCD can't find ST-LINK** → install udev rules: copy from `~/zephyr-sdk-<ver>/sysroots/x86_64-pokysdk-linux/usr/share/openocd/contrib/60-openocd.rules` to `/etc/udev/rules.d/` and `sudo udevadm control --reload`.
