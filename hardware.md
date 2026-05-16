# awto-pod-33 Hardware

## Board: STM32H750B-DK

ST STM32H750B Discovery Kit — STM32H750XB MCU, Cortex-M7 @ 480 MHz.

| Resource | Detail |
|----------|--------|
| MCU | STM32H750XBH6 |
| Flash (internal) | 128 KB |
| RAM | 512 KB SRAM + 8 MB SDRAM |
| External Flash | 128 MB QSPI NOR (dual MT25QL512AB) |
| Display | 4.3" LCD 480×272, LTDC |
| Debugger | Embedded STLINK-V3 (on-board, no external probe needed) |

## Embedded STLINK-V3

Serial number: `003400223137510E33333639`

| Interface | by-id path | Current node |
|-----------|-----------|--------------|
| SWD / GDB | (STLINK control, no TTY) | — |
| VCP (serial console) | `/dev/serial/by-id/usb-STMicroelectronics_STLINK-V3_003400223137510E33333639-if02` | `ttyACM6` |

> Use the `by-id` path in scripts — `ttyACMx` numbers shift on replug.

## LEDs

| Alias | Label | GPIO | Logic |
|-------|-------|------|-------|
| `led0` | USER2 LD7 (green) | PJ2 | ACTIVE_LOW |
| `led1` | USER1 LD6 (red) | PI13 | ACTIVE_LOW |

## Console

USART3 — PB10 (TX) / PB11 (RX) — 115200 8N1, routed through STLINK VCP.
