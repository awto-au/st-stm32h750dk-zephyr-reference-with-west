# Flash & Debug — stm32h750b_dk

STLink serial: `003400223137510E33333639`
VCP (console): `/dev/serial/by-id/usb-STMicroelectronics_STLINK-V3_003400223137510E33333639-if02`

---

## Build

```bash
source .venv/bin/activate
west build -b stm32h750b_dk code   # first build / board change
west build                                    # incremental rebuild
```

## Flash

Primary runner is `stm32cubeprogrammer` (SWD, hardware reset).

```bash
# Single STLink attached — just run:
west flash

# Multiple STLinks attached — target by serial number:
west flash --runner stm32cubeprogrammer -- --sn 003400223137510E33333639
```

Alternative — OpenOCD:

```bash
west flash --runner openocd
```

## Serial Console (TTY)

Always use the `by-id` path so the port survives replug:

```bash
tio /dev/serial/by-id/usb-STMicroelectronics_STLINK-V3_003400223137510E33333639-if02
# 115200 8N1, Ctrl-T Q to quit
```

Or with picocom:

```bash
picocom -b 115200 /dev/serial/by-id/usb-STMicroelectronics_STLINK-V3_003400223137510E33333639-if02
```

## Flash + monitor in one go

```bash
west flash && tio /dev/serial/by-id/usb-STMicroelectronics_STLINK-V3_003400223137510E33333639-if02
```

## GDB / Live Debug

```bash
# Terminal 1 — start OpenOCD GDB server:
west debugserver --runner openocd

# Terminal 2 — connect GDB:
arm-zephyr-eabi-gdb build/zephyr/zephyr.elf
(gdb) target remote :3333
(gdb) monitor reset halt
(gdb) load
(gdb) continue
```
