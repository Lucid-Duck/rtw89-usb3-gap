# rtw89 USB 2 to USB 3 Switch-Mode Gap

Empirical proof that mainline `drivers/net/wireless/realtek/rtw89/usb.c` is missing a USB 2 to USB 3 switch-mode code path that already exists in the `morrownr/rtw89` out-of-tree fork and in the sister `rtw88` driver upstream.

Without the switch-mode code, every Realtek RTL8832AU / RTL8852AU / RTL8832BU / RTL8852BU / RTL8832CU / RTL8852CU / RTL8912AU / RTL8922AU USB adapter enumerates at USB 2.0 high-speed on first plug and stays there for the life of the plug, silently capping throughput at roughly one third to one fifth of the hardware's actual ceiling.

The fix already exists. It was written by Bitterblue Smith in `morrownr/rtw89` commits `cd287cc` (2025-07-16) and `c8a8ac4` (2025-08-08). The same mechanism has been upstream in mainline `rtw88` since 2024-07-10 across four commits by the same author, all reviewed and accepted by Ping-Ke Shih (Realtek rtw89/rtw88 maintainer).

## Test matrix

Four Realtek adapters, one MediaTek control, three host configurations, multiple plug cycles, four measurements per cell.

### Adapters

| Adapter | VID:PID | Chip | Generation |
|---|---|---|---|
| D-Link DWA-X1850 Rev A1 | 2001:3321 | RTL8852AU | AX |
| D-Link DWA-X1850 Rev B1 | 2001:332c | 8852AU family | AX |
| BrosTrend AX4L | 0bda:b832 | RTL8832BU | AX |
| BrosTrend BE6500 | 0bda:8912 | RTL8922AU | BE |
| **CONTROL: Alfa AWUS036AXML** | 0e8d:7961 | MediaTek MT7921AU | N/A (mt7921u driver) |

The Alfa is the reference control. It uses a different silicon vendor and driver stack entirely. It should negotiate USB 3.0 SuperSpeed on first plug regardless of rtw89 state. If the Alfa fails to hit SuperSpeed, the host under test is compromised and the run is invalid.

### Configurations

| Config | Host | Kernel | Driver |
|---|---|---|---|
| A | Framework 13 | Fedora 43, kernel 6.19.11-200.fc43.x86_64 | In-kernel `rtw89_*` |
| B | Framework 13 | Fedora 43, kernel 6.19.11-200.fc43.x86_64 | `morrownr/rtw89` 7.1 via DKMS |
| C | Raspberry Pi 5 | Pi OS, kernel 6.12.47+rpt-rpi-2712 | `morrownr/rtw89` 7.1 via DKMS |

### Measurements per cell

1. USB enumeration state: speed, bus, sysfs path, descriptor version, product string
2. Full dmesg probe sequence from first enumeration through any retries and re-enumeration
3. iperf3 TCP uplink and downlink, 30 seconds each, byte-counter verified
4. `iw dev <iface> link` snapshot showing band, channel width, MCS, NSS, signal

## Directory layout

```
rtw89-usb3-gap/
├── README.md                              Headline summary and test matrix
├── adapters/                              Per-adapter profiles
├── hosts/                                 Per-host hardware and OS details
├── evidence/
│   ├── dmesg/
│   │   ├── framework-control/             Alfa probe logs on Framework
│   │   ├── framework-inkernel/            Bug reproduction
│   │   ├── framework-morrownr-git/        Fix verification
│   │   └── pi5-morrownr-git/              Cross-platform confirmation
│   ├── iperf3/
│   │   ├── matrix.tsv                     Aggregated throughput cells
│   │   └── per-adapter/                   Raw iperf3 stdout
│   ├── sysfs/                             Per-plug USB state snapshots
│   └── iw-link/                           Per-plug radio link snapshots
└── source-analysis/                       Mainline vs morrownr diff, rtw88 precedent
```

## Current status

Framework pre-flight complete. Alfa control run complete on Framework Configuration B (rtw89 state is irrelevant for the Alfa since it uses `mt7921u`). Realtek runs pending.
