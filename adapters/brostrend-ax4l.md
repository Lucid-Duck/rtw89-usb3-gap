# BrosTrend AX4L

## Identity

| Field | Value |
|---|---|
| Marketing name | BrosTrend AX4L (high-gain AX1800) |
| Form factor | High-gain USB stick with external antenna |
| Chip family | RTL8852BU family (binds to `rtw89_8852bu_git`) |
| Generation | AX (Wi-Fi 6) |
| Marketing AP speed | AX1800 |
| Firmware | `rtw89/rtw8852b_fw-2.bin` |

## Test results (May 2026)

Framework 13, Fedora 43, kernel 6.19.13, morrownr/rtw89 `_git` with switch-mode commits `cd287cc` + `c8a8ac4`.

### Patched (consumer multi-band router, SSID "8 Hertz WAN IP", 5 GHz 80 MHz, BSSID `a2:27:a8:63:20:80`, freq 5220 MHz)

| Direction | P=8 mean | P=1 mean |
|---|---|---|
| TCP UL | 597 Mbps | 552 Mbps |
| TCP DL | 695 Mbps | 650 Mbps |

USB enumeration confirmed at SuperSpeed (5000 Mbps) in pre/post `/sys/bus/usb/devices/<id>/speed`.

Per-cell raw evidence: [`evidence/may-2026-laptop/patched-consumer-router/brostrend-ax4l/`](../evidence/may-2026-laptop/patched-consumer-router/brostrend-ax4l/)

### Stock (BPi-R4 Pro single-band lab AP, SSID "Dev-Lab-5", 5 GHz 80 MHz, BSSID `00:0c:43:26:60:20`, freq 5180 MHz)

In-kernel rtw89 driver, no switch-mode code path. USB stays at High-Speed (480 Mbps), throughput capped at the USB 2.0 ceiling.

| Direction | P=8 mean | P=1 mean |
|---|---|---|
| TCP UL | 231 Mbps | 219 Mbps |
| TCP DL | 246 Mbps | 244 Mbps |

USB enumeration confirmed at High-Speed (480 Mbps).

Per-cell raw evidence: [`evidence/may-2026-laptop/stock-bpi-lab-ap/brostrend-ax4l/`](../evidence/may-2026-laptop/stock-bpi-lab-ap/brostrend-ax4l/)

## Delta from the patch

USB enumeration: 480 -> 5000 Mbps (10x).
Throughput, P=8 UL: 231 -> 597 Mbps (2.6x).
Throughput, P=8 DL: 246 -> 695 Mbps (2.8x).
