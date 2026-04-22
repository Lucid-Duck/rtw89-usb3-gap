# Findings: rtw89 USB 2.0 to 3.0 Switch-Mode

A cross-platform empirical study demonstrating that mainline Linux's rtw89 USB driver enumerates Realtek WiFi adapters at USB 2.0 High-Speed, and that Bitterblue Smith's switch-mode patches in morrownr/rtw89 restore USB 3.0 SuperSpeed operation.

## TL;DR

Four Realtek USB WiFi adapters across three chipsets (RTL8852AU, RTL8852BU, RTL8922AU) and three manufacturers (D-Link, Brostrend, TP-Link) all enumerate at USB 2.0 (480 Mbps) on stock mainline rtw89. Real-world TCP throughput caps at 217 to 260 Mbps upload.

With Bitterblue Smith's switch-mode commits (`cd287cc` and `c8a8ac4`) applied via morrownr/rtw89, the same adapters re-enumerate at USB 3.0 SuperSpeed (5000 Mbps) and push 613 to 957 Mbps upload. The bug and fix reproduce on both x86_64 (Framework 13, Intel Tiger Lake xHCI) and aarch64 (Raspberry Pi 5, Broadcom RP1 xHCI).

## Test results summary

| Adapter | Chip | Platform | Stock UL | morrownr UL | UL delta |
|---|---|---|---|---|---|
| DWA-X1850 A1 | 8852AU | Framework | 258 | 802 | 3.1x |
| DWA-X1850 B1 | 8852AU | Framework | 257 | 803 | 3.1x |
| Brostrend AX1800 | 8852BU | Framework | 217 | 620 | 2.9x |
| BE6500 | 8922AU | Framework | 260 (*) | 957 | 3.7x |
| DWA-X1850 A1 | 8852AU | Pi 5 | N/A (**) | 682 | -- |
| DWA-X1850 B1 | 8852AU | Pi 5 | N/A (**) | 771 | -- |
| Brostrend AX1800 | 8852BU | Pi 5 | N/A (**) | 613 | -- |
| BE6500 | 8922AU | Pi 5 | N/A (**) | 774 | -- |

(*) RTL8922AU has no stock mainline USB driver, so "stock" for this adapter was synthesized by taking morrownr/rtw89 HEAD and reverting just commits `cd287cc` and `c8a8ac4`. This isolates the exact effect of the switch-mode code on the same codebase.

(**) Pi OS kernel 6.12 has no rtw89 USB modules in-tree at all. Users have no working driver without out-of-tree morrownr/rtw89.

## Test environment

### Framework 13 (x86_64)
- Fedora 43, kernel `6.19.11-200.fc43.x86_64`
- Intel Tiger Lake-LP USB 3.2 Gen 2x1 xHCI Host Controller
- Powered hub: j5Create USB 3.0 (5 Gbps)
- BE6500 direct to laptop on USB 3.2 Gen 2 port (10 Gbps negotiated)
- morrownr/rtw89 HEAD: `2c2d99d18a660a9c1506844bb822a4edf900accf`

### Raspberry Pi 5 (aarch64)
- Pi OS stable, kernel `6.12.47+rpt-rpi-2712`
- Broadcom RP1 xHCI (2 controllers, 2x USB 2.0 + 2x USB 3.0 root hubs)
- Adapters plugged direct to Pi USB 3.0 port
- morrownr/rtw89 HEAD: same

### Network
- AP: 5 GHz (5500 MHz), SSID "8 Hertz WAN IP"
- iperf3 server: Windows PC, 2.5 GbE wired, at 192.168.1.70
- All tests: 30 seconds, default MTU, TCP, forced via `/32` route on the adapter under test

## Bug reproduction on stock kernel

Stock rtw89 enumerates USB WiFi adapters at USB 2.0 High-Speed (480 Mbps) and never issues the mode-switch register write that would cause the device to re-enumerate at USB 3.0 SuperSpeed. The WiFi radio itself negotiates normal rates (1200 Mbps PHY on all adapters), but the USB bus cannot carry traffic at more than USB 2.0's practical ceiling (~280 Mbps TCP after overhead).

Confirmed on:
- RTL8852AU (DWA-X1850 A1, 2001:3321)
- RTL8852AU (DWA-X1850 B1, 2001:332c)
- RTL8852BU (Brostrend AX1800, 0bda:b832)
- RTL8922AU (BE6500, 0bda:8912) via synthesized no-switchmode build

## Fix verification

Bitterblue Smith's commits in morrownr/rtw89 `cd287cc` (AX chips) and `c8a8ac4` (BE chips) implement the USB 2 to 3 switch via register writes to `R_AX_PAD_CTRL2` and `R_BE_PAD_CTRL2`. After the writes, the device disconnects from the USB 2.0 root hub and re-enumerates on the USB 3.0 root hub.

With these commits present:
- USB enumeration: 5000 Mbps (SuperSpeed) or 10 Gbps (SuperSpeedPlus on USB 3.1 Gen 2 host ports)
- TCP throughput: 613 to 957 Mbps upload
- WiFi radio rates unchanged (radio was never the bottleneck)

Register-value note: the DWA-X1850 A1 and Brostrend AX1800 write `0xb0f10` to register 0xc4. The DWA-X1850 B1 writes `0xb0f00`. One-bit difference in the value. The different revisions of the 8852AU hardware require slightly different switch-mode register values, all handled correctly by morrownr/rtw89.

## Phase 2: synthesized stock for RTL8922AU

The RTL8922AU (WiFi 7) chip has no stock mainline USB driver. To get comparable "before" data, I cloned morrownr/rtw89 at HEAD `2c2d99d`, created a branch, and reverted commits `cd287cc` and `c8a8ac4`. One conflict in `reg.h` (`R_BE_SCOREBOARD` was added in the same commit as the switch-mode bits) was resolved manually to keep `R_BE_SCOREBOARD` (required by `rtw8922a.c`) while dropping just the switch-mode register definitions.

Built as out-of-tree modules and loaded via `insmod` directly, this "no-switchmode" driver demonstrates what mainline behavior would be if the 8922AU driver were upstreamed without Bitterblue's switch-mode work. Result: USB 2.0 High-Speed, 260 Mbps TCP, exactly mirroring the 8852AU and 8852BU stock behavior.

This Phase 2 test also accidentally revealed the RF DACK calibration timeout issue (addressed in `morrownr/rtw89#76` by louis-kotze) since the no-switchmode clone did not include that separate fix. That is a distinct bug and not part of this study.

## Cross-platform significance

The fix works identically on:
- x86_64 (Intel Tiger Lake xHCI)
- aarch64 (Broadcom RP1 xHCI)

Pi OS users have an additional stake: kernel 6.12 ships with no rtw89 USB modules at all. For Pi 5 users, morrownr/rtw89 is not just "the fix for USB 3.0" -- it's the only way to use these adapters at all.

WiFi 7 (EHT-MCS 12) was successfully negotiated on the BE6500 on both platforms, demonstrating the current and future value of this fix for 802.11be hardware.

## Downstream vendor evidence

Beyond the empirical test matrix above, a commercial Realtek USB Wi-Fi adapter vendor (BrosTrend) publishes their DKMS driver packages at `linux.brostrend.com`. The modprobe conf files shipped inside those packages document what the vendor ships to paying customers as the supported, production configuration. Both the installer script and the two `.deb` packages are publicly downloadable; no authentication or vendor contact is required to verify any of the following.

### Source artifacts

- `https://linux.brostrend.com/install` -- POSIX shell installer (GPL-3.0-or-later)
- `https://linux.brostrend.com/rtl8852cu-dkms.deb` -- package version `1.19.22`
- `https://linux.brostrend.com/rtl8852bu-dkms.deb` -- package version `1.19.21`

SHA-256 captured 2026-04-22 for reproducibility, see `evidence/downstream-vendor/README.md` for the checksums and the extraction procedure.

### What the vendor ships

Inside each `.deb`, at `/usr/src/rtl<chip>-<ver>/<chip>.conf`, is a modprobe conf that the installer symlinks into `/etc/modprobe.d/` at install time. Both `8852cu.conf` and `8852bu.conf` contain the same four directives at the top of the active-options block:

```
blacklist rtw89_8852cu          # or rtw89_8852bu
blacklist rtw89_8852cu_git      # or rtw89_8852bu_git
options 8852cu rtw_switch_usb_mode=1
options usb-storage quirks=0bda:1a2b:i
```

Verbatim copies are preserved at `evidence/downstream-vendor/8852cu.conf` and `evidence/downstream-vendor/8852bu.conf`.

### What this means

1. **The vendor's production conf blacklists the in-kernel `rtw89_8852cu` and `rtw89_8852bu` drivers.** This is not a preference hint. It makes the in-kernel drivers unloadable for their customers on every install. The reason is on the first page of this document: at USB 2.0 High-Speed the in-kernel driver delivers roughly one third of the rated throughput the adapters are sold on (AX1800, AXE5400). The vendor cannot ship a usable product with that behavior, so they disable the in-kernel driver and install their own DKMS-built module that does issue the switch-mode write.
2. **`rtw_switch_usb_mode=1` is the vendor default.** The out-of-tree driver from `morrownr/rtw89` exposes this as a module parameter. Commits `cd287cc` (AX chips) and `c8a8ac4` (BE chips) are what implement the actual register write that the parameter gates. BrosTrend's shipping conf turns the parameter on by default for every install; the comments in the conf file document the three possible values (`0` no switch, `1` USB 2 to USB 3, `2` USB 3 to USB 2) inline.
3. **The ZeroCD `usb-storage quirks=0bda:1a2b:i` directive** disables the Realtek Virtual CD-ROM mass-storage mode at VID:PID `0bda:1a2b`, which is the mode the raw silicon enumerates in on first plug before the firmware switches it to Wi-Fi mode. This is a separate issue from switch-mode, but it appears in the same shipping conf and is another piece of out-of-the-box behavior that mainline does not handle cleanly today.

### Third-party reproduction on kernel 7.0

The same vendor has independently tested the in-kernel `rtw89_8852bu` and `rtw89_8852cu` on Kubuntu 26.04 daily with kernel 7.0 and confirmed the USB 2.0 ceiling described in this document (~300 Mbps on both chips). After enabling `rtw_switch_usb_mode=1` in the out-of-tree driver on the same test host, USB 3.0 mode was restored and 6 GHz connectivity worked on the 8852cu. This is an independent cross-check of the Framework-13 + Pi-5 matrix in this repository, on a third distro and a kernel release later than either of the two originally tested.

### Implication for upstream submission

The switch-mode patches are not a speculative addition. They are the single mechanism that separates "hardware usable out of the box" from "modprobe conf actively blacklists the in-kernel driver because it is not usable." Shipping vendor artifacts document this directly. Upstream review reviewers can audit the exact artifacts cited above, verify the SHA-256, and confirm the claim without going through any private channel.

## rtw88 precedent

The same author (Bitterblue Smith) landed equivalent USB 2 to 3 switch code in mainline rtw88 across four commits from 2024-07-10 to 2025-04-02, reviewed and accepted by Ping-Ke Shih (Realtek, rtw88/rtw89 maintainer). The mechanism is identical; only the register addresses differ between the sister driver families. Upstream precedent for review approval exists.

## Conclusion

Every stock mainline rtw89 USB WiFi adapter on every supported distro is silently running at approximately one third of the hardware's real capability. The fix is already written, already tested across multiple adapters and platforms, already has direct upstream precedent, and is maintained in morrownr/rtw89. The remaining work is formal upstream submission.

## Reproducing these results

Raw iperf3 stdout, dmesg captures, sysfs snapshots, and `iw link` output are in `evidence/`.

The no-switchmode source build used for Phase 2 can be reproduced by cloning morrownr/rtw89 at HEAD `2c2d99d`, creating a branch, and reverting commits `cd287cc` and `c8a8ac4`. One conflict in `reg.h` should be resolved to keep the `R_BE_SCOREBOARD` definition while dropping the switch-mode register bits.
