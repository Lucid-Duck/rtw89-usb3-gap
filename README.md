# rtw89 USB 2 to USB 3 Switch-Mode Gap

Empirical proof that mainline `drivers/net/wireless/realtek/rtw89/usb.c` is missing a USB 2 to USB 3 switch-mode code path that already exists in the `morrownr/rtw89` out-of-tree fork and in the sister `rtw88` driver upstream.

Without the switch-mode code, every Realtek RTL8832AU / RTL8852AU / RTL8832BU / RTL8852BU / RTL8832CU / RTL8852CU / RTL8912AU / RTL8922AU USB adapter enumerates at USB 2.0 high-speed on first plug and stays there for the life of the plug, capping real-world TCP throughput at roughly 260 Mbps regardless of the radio's actual capability.

The fix already exists. It was written by Bitterblue Smith in `morrownr/rtw89` commits `cd287cc` (2025-07-16) and `c8a8ac4` (2025-08-08). The same mechanism has been upstream in mainline `rtw88` since 2024-07-10 across four commits by the same author, all reviewed and accepted by Ping-Ke Shih (Realtek rtw89/rtw88 maintainer).

## TL;DR

| Adapter | Chip | Stock UL | morrownr UL | Upload delta |
|---|---|---|---|---|
| D-Link DWA-X1850 A1 | RTL8852AU | 258 Mbps | 802 Mbps | 3.1x |
| D-Link DWA-X1850 B1 | RTL8852AU | 257 Mbps | 803 Mbps | 3.1x |
| Brostrend AX1800 | RTL8852BU | 217 Mbps | 620 Mbps | 2.9x |
| TP-Link BE6500 | RTL8922AU | 260 Mbps (*) | 957 Mbps | 3.7x |

Tested on Framework 13 (Fedora 43, kernel 6.19.11, x86_64, Intel Tiger Lake xHCI) and Raspberry Pi 5 (Pi OS, kernel 6.12.47, aarch64, Broadcom RP1 xHCI). All four adapters also confirmed at USB 3.0 SuperSpeed on the Pi, with WiFi 7 (EHT-MCS 12) negotiated on the BE6500.

(*) The RTL8922AU (WiFi 7) has no stock mainline USB driver at all. "Stock" for that adapter was synthesized by cloning `morrownr/rtw89` at HEAD `2c2d99d` and reverting just commits `cd287cc` and `c8a8ac4` to isolate the effect of the switch-mode code on the same codebase.

See [FINDINGS.md](FINDINGS.md) for the full writeup.

## Directory layout

```
rtw89-usb3-gap/
├── README.md                     Headline summary (this file)
├── FINDINGS.md                   Full technical writeup
├── adapters/                     Per-adapter identity and behavioral notes
├── hosts/                        Per-host hardware and OS details
└── evidence/
    ├── iperf3/                   iperf3 logs from the full test matrix
    ├── dmesg/                    dmesg probe captures (framework-control + framework-morrownr-git)
    ├── sysfs/                    Per-plug USB state snapshots
    ├── iw-link/                  Per-plug radio link snapshots
    └── crash-2026-04-11/         xHCI hard lockup crash evidence (1235 kernel lines, recovered from /var/log/messages)
```

## Credits

- **Bitterblue Smith (@dubhater)** wrote the switch-mode code and all rtw89 USB driver support. This repository exists to support his work, not to compete with it.
- **louis-kotze** opened `morrownr/rtw89#76` addressing the separate RF calibration timeout issue; that fix is independently valuable and being handled via that PR.
- **Testing and writeup:** Devin Wittmayer (@Lucid-Duck).

## License

Test data and documentation are released under CC0. The no-switchmode branch used in the synthesized-stock test is derived from `morrownr/rtw89` and retains its original GPL-2.0 license.
