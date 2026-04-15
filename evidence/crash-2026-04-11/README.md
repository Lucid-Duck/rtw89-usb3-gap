# xHCI Hard Lockup -- 2026-04-11

Full kernel log capture of a hard LOCKUP triggered by an RTL8922AU
(BrosTrend BE6500) DACK calibration failure cascade on mainline
`rtw89` with all four Realtek USB adapters plugged in (8852AU x2,
8852BU, 8922AU) on a Framework Laptop (13, 11th-gen Intel, Fedora 43,
kernel `6.19.11-200.fc43.x86_64`).

Relevant to louis-kotze's upstream
[PATCH v2] wifi: rtw89: phy: increase RF calibration timeouts for USB
transport
(<https://lore.kernel.org/linux-wireless/20260415111339.453602-1-loukot@gmail.com/>)
as a severity-multiplier data point: the same DACK-timeout failure
path that louis observed as "repeated DACK fail -> connection drop"
on a single 8922AU can, with multiple Realtek USB adapters on the
same xHCI controller, escalate to an `xhci_hcd`-declared HC death
and a hard LOCKUP in `usb_unanchor_urb` IRQ context that requires
a hard reboot to recover.

## Timeline (all times Apr 11 2026, PDT)

- `11:04:29` -- NetworkManager deauthenticates the 8852AU client.
- `11:04:31` -- `rtw89_8852au` and `rtw89_8852bu` interface drivers
  deregistering (module reload in progress).
- `11:04:39` -- `rtw89_8922au_git 2-1.4.4:1.0: failed to wait RF DACK`
  (BE6500 DACK calibration timeout, the same failure class
  addressed by louis's v2 timeout multiplier).
- `11:04:45` -- `rtw89_8852au_git 2-1.1:1.0: loaded firmware rtw89/rtw8852a_fw-1.bin`
- `11:04:51` -- `xhci_hcd 0000:00:0d.0: xHCI host not responding to stop endpoint command` ->
  `xHCI host controller not responding, assume dead` ->
  `HC died; cleaning up`.
  All USB devices on controller `0000:00:0d.0` force-disconnected.
- `11:05:14` -- `watchdog: CPU5: Watchdog detected hard LOCKUP on cpu 5`.
  RIP: `native_queued_spin_lock_slowpath+0x21a/0x310`.
  Call Trace (IRQ context):
  `usb_hcd_giveback_urb` ->
  `__raw_spin_lock_irqsave` ->
  `usb_unanchor_urb+0x31/0x70` ->
  `__usb_hcd_giveback_urb+0x81/0x120` ->
  `usb_giveback_urb_bh+0xb1/0x140`.
- `11:05:32` onward -- soft lockup cascade on CPU3/4/7
  (`firefox:22168`, `StreamT~ns #218:30393`,
  `rcu_exp_gp_kthr:18`) stuck progressively 22s -> 179s.
- System unrecoverable. Hard reboot required (next boot in
  `journalctl --list-boots` starts at `08:22` the following morning,
  consistent with a hard power cycle by the user).

## Host

- Framework Laptop (FRANBMCP0C), BIOS 03.17 10/27/2022.
- Fedora 43, kernel `6.19.11-200.fc43.x86_64 PREEMPT(lazy)`.
- Loaded `rtw89_*_git` modules (morrownr/rtw89 out-of-tree build)
  for all four Realtek adapters, `mt7921u` also loaded
  (Alfa AWUS036AXML as control). Full `Modules linked in:` list
  preserved in the log.
- Four Realtek USB adapters concurrently attached:
  - D-Link DWA-X1850 A1 (8852AU)
  - D-Link DWA-X1850 B1 (8852AU)
  - BrosTrend AX4L (8832BU on 8852b driver family)
  - BrosTrend BE6500 (8922AU) <- triggering adapter
- Alfa AWUS036AXML (MT7921AUN) attached as control.

## Files

- `messages-kernel-11-04-to-11-12.log` -- verbatim kernel-only
  lines from `/var/log/messages-20260412`, covering
  `11:04:00` through `11:12:00`. Includes the full
  `Modules linked in`, register dump (RAX/RBX/...R15, CR0-CR4,
  FS, GS, PKRU), IRQ stack Call Trace, and the subsequent
  per-CPU soft lockup cascade. 1235 lines total.

## Why this matters upstream

louis-kotze's v2 already fixes the proximate cause (DACK timeout too
short on USB transport; 4x multiplier resolves it). This capture
demonstrates that without the fix the failure mode is not bounded
to a single-adapter connection drop -- on a host with multiple
Realtek USB WiFi adapters on a shared xHCI controller, a
DACK-timeout cascade can kill the entire USB host controller and
deadlock a CPU in IRQ context. That is a materially more severe
failure class than the per-adapter symptom documented in v2's
commit message and strengthens the case for merging the timeout
fix promptly.
