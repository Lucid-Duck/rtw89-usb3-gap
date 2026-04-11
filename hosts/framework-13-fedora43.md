# Framework 13 Test Host

## Operating system and kernel

- Fedora 43
- Kernel: `6.19.11-200.fc43.x86_64` (stable, not mainline rc)
- Architecture: x86_64
- Secure Boot: disabled

## USB host controllers

```
00:0d.0  Intel Corporation Tiger Lake-LP Thunderbolt 4 USB Controller (rev 01)
00:0d.2  Intel Corporation Tiger Lake-LP Thunderbolt 4 NHI #0 (rev 01)
00:0d.3  Intel Corporation Tiger Lake-LP Thunderbolt 4 NHI #1 (rev 01)
00:14.0  Intel Corporation Tiger Lake-LP USB 3.2 Gen 2x1 xHCI Host Controller (rev 20)
```

Two host controller complexes: the Intel Tiger Lake-LP standalone xHCI at 00:14.0 and the Thunderbolt 4 controller at 00:0d.x. Expansion slots route through different controllers depending on slot position and module type.

## Sysfs bus map (as observed 2026-04-11)

```
Bus 1  USB 2.0 root hub
Bus 2  USB 3.0 root hub (Intel Tiger Lake xHCI, 10000M root)
Bus 3  USB 2.0 root hub (12 ports, internal)
Bus 4  USB 3.0 root hub
```

The slot under test for this campaign is the bottom-right USB-A module, which routes through **Bus 2** (the 10000M root). Confirmed via Alfa control run: plugging into this slot enumerates at sysfs path `2-2` at SuperSpeed 5000 Mbit/s when the adapter supports USB 3.

## Adapter under test slot

- Slot: bottom-right expansion module, USB-A type
- sysfs port: `/sys/bus/usb/devices/2-2`
- Routed through: Intel Tiger Lake-LP USB 3.2 Gen 2x1 xHCI Host Controller

## Test network

- SSID: "8 Hertz WAN IP"
- 2.4 GHz BSSID: `00:00:00:00:00:A3` @ 2412 MHz
- 5 GHz BSSID: `00:00:00:00:00:A2` @ 5500 MHz
- 6 GHz BSSID: `00:00:00:00:00:A1` @ 5975 MHz
- Subnet: `192.168.1.0/24`
- Gateway: `192.168.1.254`

## iperf3 server

- IP: `192.168.1.70` (Windows host)
- Port: 5201 (default)
- Ping from Framework: 0.6 to 0.8 ms wired, 3.7 to 7.2 ms over 6 GHz via Alfa

## DKMS state

```
rtw89/7.1, 6.19.10-200.fc43.x86_64, x86_64: installed
rtw89/7.1, 6.19.11-200.fc43.x86_64, x86_64: installed
```

Both kernels have the morrownr `_git` variant built and ready. `/etc/modprobe.d/rtw89.conf` enables and configures the `_git` variant for Configuration B runs. For Configuration A runs the blacklist file is temporarily renamed to `.disabled`.
