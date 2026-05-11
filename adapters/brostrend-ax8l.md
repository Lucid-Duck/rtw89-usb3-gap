# BrosTrend AX8L AXE5400

## Identity

| Field | Value |
|---|---|
| Marketing name | BrosTrend AX8L (AXE5400) |
| Form factor | USB stick with external antennas, Wi-Fi 6E (tri-band) |
| Chip family | RTL8852CU family (binds to `rtw89_8852cu_git`) |
| Generation | AXE (Wi-Fi 6E) |
| Marketing AP speed | AXE5400 |
| Firmware | `rtw89/rtw8852c_fw-1.bin` |

## Test results (May 2026)

Framework 13, Fedora 43, kernel 6.19.13, morrownr/rtw89 `_git` with switch-mode commits `cd287cc` + `c8a8ac4`. Consumer multi-band router AP (SSID "8 Hertz WAN IP"), 6 GHz 160 MHz, BSSID `0a:27:a8:63:20:92`, freq 5975 MHz.

| Direction | P=8 mean | P=1 mean |
|---|---|---|
| TCP UL | 1440 Mbps | 1371 Mbps |
| TCP DL | 510 Mbps | 568 Mbps |

N=10 iterations of 30 s each per sub-cell. WiFi PHY rate 2401.9 MBit/s (HE160 MCS 11 NSS 2). USB enumeration confirmed at SuperSpeed (5000 Mbps).

Per-cell raw evidence: [`evidence/may-2026-laptop/patched-consumer-router/brostrend-ax8l/`](../evidence/may-2026-laptop/patched-consumer-router/brostrend-ax8l/)
