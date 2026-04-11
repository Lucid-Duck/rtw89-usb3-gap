# Alfa AWUS036AXML (Control)

## Identity

- Marketing name: Alfa AWUS036AXML
- VID:PID: `0e8d:7961`
- Silicon: MediaTek MT7921AU
- Driver: `mt7921u` (in-kernel, part of the mt76 family)
- Role in this campaign: reference control. Unaffected by rtw89 driver state. Any configuration in which the Alfa fails to negotiate USB 3.0 SuperSpeed on first plug is considered an invalid host and the run is discarded.

## Framework 13, plug 1, 2026-04-11

Host: Framework 13, kernel 6.19.11-200.fc43.x86_64. rtw89 state: `/etc/modprobe.d/rtw89.conf` in place but not loaded (no Realtek adapter plugged). This run is driver-agnostic from the rtw89 perspective.

### Enumeration state

| Field | Value |
|---|---|
| sysfs path | `/sys/bus/usb/devices/2-2` |
| Bus | 2 (Intel Tiger Lake-LP xHCI, 10000M root) |
| Device number | 5 |
| **Negotiated speed** | **5000 Mbit/s (USB 3.0 SuperSpeed)** |
| Descriptor version | 3.20 |
| Product string | `Wireless_Device` |
| Manufacturer string | `MediaTek Inc.` |
| Interface | `wlp0s13f0u2i3` |
| Driver bound | `mt7921u` (interface 3) |

Single-step negotiation. No EPROTO retries. No disconnect and re-enumerate. The adapter came up at SuperSpeed immediately.

### Firmware

```
HW/SW Version:    0x8a108a10, Build Time: 20251223091050a
WM Firmware:      ____010000, Build Time: 20251223091148
```

### Radio link

| Field | Value |
|---|---|
| BSSID | `00:00:00:00:00:A1` |
| SSID | "8 Hertz WAN IP" |
| Band | 6 GHz |
| Frequency | 5975 MHz |
| Channel width | 80 MHz |
| Signal | -33 dBm |
| tx bitrate | 1200.9 Mbit/s (HE-MCS 11, NSS 2, 80 MHz) |
| rx bitrate | 907.4 Mbit/s (HE-MCS 9, NSS 2, 80 MHz) |

### iperf3 TCP uplink, 30 seconds

| Metric | Value |
|---|---|
| Total transfer | 2.61 GBytes |
| **Bitrate (sender)** | **748 Mbit/s** |
| Bitrate (receiver) | 747 Mbit/s |
| Retransmits | 0 |
| Per-5s samples | 728, 741, 756, 755, 754, 754 Mbit/s |
| Byte-counter tx delta | 2932967095 bytes over 30s (roughly 782 Mbit/s wire, matches iperf3 within L2 overhead) |

### iperf3 TCP downlink, 30 seconds

| Metric | Value |
|---|---|
| Total transfer | 2.27 GBytes |
| **Bitrate** | **651 Mbit/s** |
| Per-5s samples | 597, 668, 662, 682, 681, 617 Mbit/s |
| Byte-counter rx delta | 2556139367 bytes over 30s (roughly 682 Mbit/s wire) |

### Interpretation

The control is healthy. The Framework's bottom-right USB-A slot is wired to the Intel xHCI SuperSpeed root, the 6 GHz radio path to the test access point is strong at -33 dBm, and the path to the iperf3 server at 192.168.1.70 is uncontested. Any Realtek adapter plugged into this slot under these conditions is being measured on a known-good host.

The Alfa does not exercise any rtw89 code. It is valid as a control for both Configuration A and Configuration B on the Framework. A second Alfa run under Configuration A is optional but not required.
