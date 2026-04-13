# Brostrend AX1800

## Identity

| Field | Value |
|---|---|
| Marketing name | Brostrend AX1800 |
| VID:PID | `0bda:b832` |
| Chip | RTL8832BU |
| Driver binding | `rtw89_8852bu_git` (the `_8852bu` module handles both RTL8852BU and RTL8832BU: same B-series USB family) |
| Generation | AX (Wi-Fi 6) |
| Firmware | `rtw8852b_fw-2.bin`, version 0.29.29.15 (6fb3ec41) |
| Chip info from firmware | CID: 0, CV: 2, AID: 0, ACV: 1, RFE: 1 |

## Behavioral notes

- Exposes a `0bda:1a2b "DISK"` installer shim device for approximately 3 seconds before presenting the wireless device, same as the DWA-X1850 A1.
- USB 2 descriptor string: "802.11ac WLAN Adapter" (matches A1). Switches to "802.11ax" on USB 3 re-enumeration.
- Switch-mode register write value: `0xb0f10` to register 0xc4, same as the A1 and different from the B1's `0xb0f00`.

## Switch-mode sequence on Framework 13, Configuration B (morrownr _git)

Identical signature on both hub and direct-slot topologies, 2026-04-11:

```
usb: new high-speed device, 0bda:1a2b "DISK"               (installer shim)
[~3 second gap]
usb: new high-speed device, 0bda:b832 "802.11ac WLAN Adapter"
rtw89_8852bu_git: loaded firmware rtw89/rtw8852b_fw-2.bin
rtw89_8852bu_git: usb write32 0xc4 fail ret=-71 value=0xb0f10 attempt=0
rtw89_8852bu_git: usb write32 0xc4 fail ret=-71 value=0xb0f10 attempt=1
rtw89_8852bu_git: usb write32 0xc4 fail ret=-71 value=0xb0f10 attempt=2
rtw89_8852bu_git: usb write32 0xc4 fail ret=-71 value=0xb0f10 attempt=3
rtw89_8852bu_git: usb write32 0xc4 fail ret=-71 value=0xb0f10 attempt=4
usb disconnect
[~4 second gap]
usb: new SuperSpeed USB device, 0bda:b832 "802.11ax WLAN Adapter"
rtw89_8852bu_git: Firmware 0.29.29.15 loaded, chip CV:2 ACV:1 RFE:1
wlp0s...: renamed from wlan0
```

## Identification pass results

### Hub topology (J5 Create powered USB 3.0 hub, 2026-04-11)

- Initial enumeration path: `3-1.3` (USB 2 high-speed via hub)
- Final path: `2-1.3` (USB 3 SuperSpeed via hub)
- Negotiated speed: **5000 Mbit/s SuperSpeed**
- Full dmesg: `evidence/dmesg/framework-morrownr-git/brostrend-ax4l-hub-plug1-2026-04-11.log`

### Direct slot topology (Framework bottom-right USB-A, 2026-04-11)

- Initial enumeration path: `3-3` (USB 2 high-speed companion root)
- Final path: `2-2` (USB 3 SuperSpeed root)
- Negotiated speed: **5000 Mbit/s SuperSpeed**
- Full dmesg: `evidence/dmesg/framework-morrownr-git/brostrend-ax4l-direct-plug1-2026-04-11.log`

## Topology invariance

Both hub and direct-slot runs produced identical switch-mode signatures. The switch-mode code is not sensitive to USB topology.

## Pending

- Configuration A (in-kernel rtw89) direct-slot run to confirm the adapter stays at USB 2.0 high-speed
- iperf3 throughput measurements under Configurations A and B
