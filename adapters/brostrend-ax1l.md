# BrosTrend AX1L

## Identity

| Field | Value |
|---|---|
| Marketing name | BrosTrend AX1L (compact AX1800) |
| Form factor | Compact USB dongle |
| Chip family | RTL8852BU family (binds to `rtw89_8852bu_git`) |
| Generation | AX (Wi-Fi 6) |
| Marketing AP speed | AX1800 |
| Firmware | `rtw89/rtw8852b_fw-2.bin` |

## Test results (May 2026)

Framework 13, Fedora 43, kernel 6.19.13, morrownr/rtw89 `_git` with switch-mode commits `cd287cc` + `c8a8ac4`. Consumer multi-band router AP (SSID "8 Hertz WAN IP"), 5 GHz 80 MHz, BSSID `a2:27:a8:63:20:80`, freq 5220 MHz.

| Direction | P=8 mean | P=1 mean |
|---|---|---|
| TCP UL | 608 Mbps | 531 Mbps |
| TCP DL | 843 Mbps | 754 Mbps |

N=10 iterations of 30 s each per sub-cell. USB enumeration confirmed at SuperSpeed (5000 Mbps) in pre/post `/sys/bus/usb/devices/<id>/speed`.

Per-cell raw evidence: [`evidence/may-2026-laptop/patched-consumer-router/brostrend-ax1l/`](../evidence/may-2026-laptop/patched-consumer-router/brostrend-ax1l/)
