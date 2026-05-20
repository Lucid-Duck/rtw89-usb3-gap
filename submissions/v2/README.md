# [PATCH rtw-next v2] wifi: rtw89: usb: Support switching to USB 3 mode

Mirror of the linux-wireless mailing list submission. Lore thread: https://lore.kernel.org/linux-wireless/20260511160811.17647-1-lucid_duck@justthetip.ca/T/#t

Sent 2026-05-11 by Devin Wittmayer carrying Bitterblue Smith's morrownr/rtw89 commit cd287ccf544b (2025-07-16) rebased onto pkshih/rtw rtw-next HEAD c1ed02655f91. Authorship preserved; Devin Signed-off-by as relayer + Tested-by per chip.

## Files

- [0000-cover-letter.eml](0000-cover-letter.eml) -- v2 cover letter, AX-generation scope (8852AU / 8852BU / 8852CU), 60 plug-cycles + 30+ throughput cells captured 2026-04-11 to 2026-05-07, evidence links to this repo's `evidence/` and `adapters/`.
- [0001-wifi-rtw89-usb-Support-switching-to-USB-3-mode.eml](0001-wifi-rtw89-usb-Support-switching-to-USB-3-mode.eml) -- the patch itself: `R_AX_PAD_CTRL2` register write in `rtw89_usb_switch_mode()` + `switch_usb_mode` module parameter, 45 insertions across `reg.h` and `usb.c`.

## Reviewer replies

In [replies/](replies/):

- [2026-05-11-johannes-module-param.eml](replies/2026-05-11-johannes-module-param.eml) -- Devin's reply to Johannes Berg's objection to the new module parameter. Cites the rtw88 grandfathered pattern (commit 315c23a64e99, 2024-07-10) and offers three v3 paths: drop, sysfs/nl80211, or keep for symmetry.
- [2026-05-11-bitterblue-test-verify.eml](replies/2026-05-11-bitterblue-test-verify.eml) -- Devin's reply to Bitterblue Smith confirming the test matrix; points to `evidence/may-2026-laptop/` and `adapters/` for per-cell evidence.
- [2026-05-12-bitterblue-dack-crash.eml](replies/2026-05-12-bitterblue-dack-crash.eml) -- Devin clarifying that the 2026-04-11 crash in `evidence/crash-2026-04-11/` is BE-gen DACK calibration, unrelated to the AX-gen switch-mode patch.
- [2026-05-12-pkshih-chip-list.eml](replies/2026-05-12-pkshih-chip-list.eml) -- Devin's reply to Ping-Ke Shih with the positive tested chip list for v3 (RTL8852A/B/C; RTL8851B excluded as no USB 3 hardware variant; WiFi 7 untestable since BE-gen AX-gen split).

## v3 gating

v3 is held pending Johannes's decision on the module parameter direction (drop / sysfs / keep for rtw88 symmetry). Once settled, v3 will also carry Ping-Ke's positive chip list in the commit body.

## Earlier versions

- v1 (2026-05-08): https://lore.kernel.org/linux-wireless/20260508054421.128938-1-lucid_duck@justthetip.ca/T/#t -- superseded by v2 (rebased onto rtw-next).
