# BrosTrend BE6500

## Identity

| Field | Value |
|---|---|
| Marketing name | BrosTrend BE6500 |
| VID:PID | `0bda:8912` |
| Chip | **RTL8922AU** (confirmed via driver binding and firmware filename, resolves PLAN.md open question #3) |
| Driver binding | `rtw89_8922au_git` (BE-generation USB driver) |
| Generation | BE (Wi-Fi 7 / 802.11be / EHT) |
| Firmware | `rtw8922a_fw-4.bin`, version 0.35.80.3 (8ef4f0cf), plus a secondary firmware 0.1.0.0 (7b393818, cmd type 64) |
| Chip info from firmware | CID: 71, CV: 1, **AID: 6933**, ACV: 1, RFE: 1 |

Note: the chip info `AID: 6933` is distinctive. All AX-generation Realtek USB chips in this test set report `AID: 0`. The BE generation carries a real asset identifier in that field.

## Behavioral notes

- Exposes a `0bda:1a2b "DISK"` installer shim device for approximately 1 second before presenting the wireless device, same as the AX4L and A1.
- USB 2 descriptor string: **"802.11be WLAN Adapter"** (Wi-Fi 7 marketing string, distinctive in this test set). Descriptor is the same on both USB 2 and USB 3 sides.
- Switch-mode register write value: `0x90130f00` to register 0xc4. This is a **32-bit value**, distinctly wider than the 2-byte values used by AX-generation silicon (`0xb0f00` or `0xb0f10`). This reflects the use of Bitterblue Smith's `rtw89_usb_switch_mode_be()` function (morrownr/rtw89 commit `c8a8ac4`, 2025-08-08) instead of `rtw89_usb_switch_mode_ax()` (commit `cd287cc`, 2025-07-16).
- After successful USB 3 negotiation and firmware load, the driver logs `failed to wait RF DACK` and `failed to wait RF TSSI` warnings during RF front-end calibration. These warnings are unrelated to the USB subsystem and do not prevent USB 3 SuperSpeed negotiation. They indicate a separate RF init issue in the 8922AU support path that is out of scope for this campaign.

## Switch-mode sequence on Framework 13, Configuration B (morrownr _git)

Identical signature on both hub and direct-slot topologies, 2026-04-11:

```
usb: new high-speed device, 0bda:1a2b "DISK"               (installer shim)
[~1 second gap]
usb: new high-speed device, 0bda:8912 "802.11be WLAN Adapter"
rtw89_8922au_git: loaded firmware rtw89/rtw8922a_fw-4.bin
rtw89_8922au_git: usb write32 0xc4 fail ret=-71 value=0x90130f00 attempt=0
rtw89_8922au_git: usb write32 0xc4 fail ret=-71 value=0x90130f00 attempt=1
rtw89_8922au_git: usb write32 0xc4 fail ret=-71 value=0x90130f00 attempt=2
rtw89_8922au_git: usb write32 0xc4 fail ret=-71 value=0x90130f00 attempt=3
rtw89_8922au_git: usb write32 0xc4 fail ret=-71 value=0x90130f00 attempt=4
usb disconnect
[~3 to 4 second gap]
usb: new SuperSpeed USB device, 0bda:8912 "802.11be WLAN Adapter"
rtw89_8922au_git: Firmware 0.35.80.3 loaded, chip CID:71 CV:1 AID:6933
wlp0s...: renamed from wlan0
rtw89_8922au_git: failed to wait RF DACK                   (unrelated RF init warning)
rtw89_8922au_git: failed to wait RF TSSI                   (unrelated RF init warning)
```

## Identification pass results

### Hub topology (J5 Create powered USB 3.0 hub, 2026-04-11)

- Initial enumeration path: `3-1.3` (USB 2 high-speed via hub)
- Final path: `2-1.3` (USB 3 SuperSpeed via hub)
- Negotiated speed: **5000 Mbit/s SuperSpeed**
- Full dmesg: `evidence/dmesg/framework-morrownr-git/brostrend-be6500-hub-plug1-2026-04-11.log`

### Direct slot topology (Framework bottom-right USB-A, 2026-04-11)

- Initial enumeration path: `3-3` (USB 2 high-speed companion root)
- Final path: `2-2` (USB 3 SuperSpeed root)
- Negotiated speed: **5000 Mbit/s SuperSpeed**
- Full dmesg: `evidence/dmesg/framework-morrownr-git/brostrend-be6500-direct-plug1-2026-04-11.log`

## Topology invariance

Both hub and direct-slot runs produced identical switch-mode signatures. The switch-mode code is not sensitive to USB topology.

## BE-generation code path coverage

The BE6500 is the only BE-generation adapter in this test set. Its presence means the test matrix exercises **both** of Bitterblue's switch-mode code paths: `rtw89_usb_switch_mode_ax()` via the DWA-X1850 A1/B1 and BrosTrend AX4L, and `rtw89_usb_switch_mode_be()` via this adapter. Every switch-mode code path in morrownr/rtw89 is covered by at least one test adapter.

## Pending

- Configuration A (in-kernel rtw89) direct-slot run to confirm the adapter stays at USB 2.0 high-speed
- iperf3 throughput measurements under Configurations A and B
- Investigation of the RF DACK / RF TSSI init warnings (separate issue, out of scope)
