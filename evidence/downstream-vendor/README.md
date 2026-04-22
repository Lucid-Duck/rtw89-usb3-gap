# Downstream vendor evidence: BrosTrend shipping modprobe configs

## What is in this directory

Verbatim copies of two `modprobe.d` configuration files shipped by BrosTrend
for their Realtek RTL8852BU and RTL8852CU based Wi-Fi adapters:

- `8852bu.conf` -- extracted from `rtl8852bu-dkms.deb` package version `1.19.21`
- `8852cu.conf` -- extracted from `rtl8852cu-dkms.deb` package version `1.19.22`

Both files are installed by the vendor's DKMS package into
`/etc/modprobe.d/` at install time.

## Why this evidence matters

The switch-mode patches this repository documents (Bitterblue Smith's
`cd287cc` and `c8a8ac4` in `morrownr/rtw89`) are the exact mechanism
that the BrosTrend production conf files rely on.

Both vendor-shipped confs contain, at the top of the active-options block:

- `blacklist rtw89_8852cu` (or `rtw89_8852bu`)
- `blacklist rtw89_8852cu_git` (or `rtw89_8852bu_git`)
- `options 8852cu rtw_switch_usb_mode=1` (or `options 8852bu rtw_switch_usb_mode=1`)
- `options usb-storage quirks=0bda:1a2b:i`

In other words, the vendor of every BrosTrend Realtek Wi-Fi 6 USB adapter
currently on the market ships a modprobe conf that:

1. **Actively disables the in-kernel `rtw89_8852cu` / `rtw89_8852bu` drivers**
   by blacklisting them. This is not "we prefer the out-of-tree driver"
   phrasing. The shipping artifact makes the in-kernel driver unloadable
   for their customers, because at USB 2 High-Speed it cannot deliver the
   rated throughput the hardware is sold on (AX1800, AXE5400).

2. **Sets `rtw_switch_usb_mode=1`** so the out-of-tree driver performs the
   USB 2 to USB 3 switch at probe time. This is the vendor's production
   setting. The comments in the conf files document the parameter values
   inline.

3. **Disables the Realtek Virtual CD-ROM / ZeroCD mass-storage mode** via
   the standard `usb-storage` quirk for VID:PID `0bda:1a2b`, so the
   adapter enumerates directly as a Wi-Fi device rather than a CDROM on
   first plug.

All three of these behaviors are absent or broken in mainline's in-kernel
rtw89 driver today. The switch-mode patches in `morrownr/rtw89` are the
upstream resolution path for the first two.

## Provenance

The files in this directory were obtained strictly from BrosTrend's
public Linux driver distribution site. No private or non-public source
was used.

### URLs

- `https://linux.brostrend.com/install` -- POSIX shell installer
- `https://linux.brostrend.com/rtl8852cu-dkms.deb` -- Debian DKMS package (RTL8852CU)
- `https://linux.brostrend.com/rtl8852bu-dkms.deb` -- Debian DKMS package (RTL8852BU)

### SHA-256 (captured 2026-04-22)

```
c719c67c2eccb11a7c0006659a46a2a69fdf90d93d99acd9b338139b73fe4c98  rtl8852cu-dkms.deb
1132af42365509e039fb49d0dbb62286a4607da7ae116e17b7bf69494332a7de  rtl8852bu-dkms.deb
5b98d67571d8864bdf54f80a9c19e2a31aaf25059b93ec4d8f75d66fc5aac50f  install.sh
```

### Extraction procedure

Standard Debian package extraction, no privileged operations:

```sh
mkdir -p ~/builds/brostrend-inspect/{debs,8852cu-extract,8852bu-extract}
cd ~/builds/brostrend-inspect/debs
wget https://linux.brostrend.com/rtl8852cu-dkms.deb
wget https://linux.brostrend.com/rtl8852bu-dkms.deb
cd ../8852cu-extract
ar x ../debs/rtl8852cu-dkms.deb
tar xf data.tar.gz ./usr/src/rtl8852cu-1.19.22/8852cu.conf
cd ../8852bu-extract
ar x ../debs/rtl8852bu-dkms.deb
tar xf data.tar.gz ./usr/src/rtl8852bu-1.19.21/8852bu.conf
```

The `8852cu.conf` and `8852bu.conf` files extracted by this procedure
are byte-identical copies of the files committed to this directory.

## Installer authorship note

The `install` shell script is published under GPL-3.0-or-later with a
copyright header of `Copyright 2018-2025 Alkis Georgopoulos
<github.com/alkisg>`. This authorship credit is the public record at
the URL above and is reproduced here only to point to that public
record.

## License

The `.conf` files in this directory are verbatim copies of files shipped
under the same license as the rest of the `rtl8852bu-dkms` and
`rtl8852cu-dkms` packages (GPL-2.0). They are reproduced here under
fair use for technical documentation and upstream submission support.
