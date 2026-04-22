# Downstream vendor evidence: BrosTrend shipping modprobe configs

## What is in this directory

Verbatim copies of the two `modprobe.d` configuration files installed by
BrosTrend's DKMS packages for their Realtek RTL8852BU and RTL8852CU based
Wi-Fi adapters:

- `8852bu.conf` -- extracted from `rtl8852bu-dkms.deb` package version `1.19.21`
- `8852cu.conf` -- extracted from `rtl8852cu-dkms.deb` package version `1.19.22`

Both files are installed by the vendor's DKMS package into
`/etc/modprobe.d/` at install time.

## Attribution

These shipped conf files are NOT wholly authored by BrosTrend. They are
Nick's (`@morrownr`) OOT driver conf templates with two vendor-specific
`sed` tweaks applied during BrosTrend's DKMS packaging.

### Upstream source of the conf templates

The base templates live in Nick's out-of-tree driver source repos:

- `https://github.com/morrownr/rtl8852bu-20250826` -- source of `8852bu.conf`
- `https://github.com/morrownr/rtl8852cu-20251113` -- source of `8852cu.conf`

Everything in the shipped `.conf` files that isn't one of the two lines
called out below is Nick's work: the blacklist directives, the option
templates, the extensive documentation comments, the hostapd-mode notes,
and the list of exposed `/sys/module/<driver>/parameters/` entries.

### Vendor tweaks applied by BrosTrend

BrosTrend's contribution to the shipped conf is limited to two lines,
applied via `sed` in their packaging pipeline:

1. **`options 8852cu rtw_switch_usb_mode=1`** (same for `8852bu`).
   The upstream template sets this to `=0` (switch off by default).
   BrosTrend flips it to `=1` so every customer install gets the USB 2
   to USB 3 switch by default. This is a commercial choice: customers
   buy the adapters for USB 3.0 speeds and shouldn't need to edit a
   modprobe file to get them.
2. **`options usb-storage quirks=0bda:1a2b:i`.** Not present in the
   upstream template at all. Added by BrosTrend. Disables the Realtek
   Virtual CD-ROM (ZeroCD) mass-storage mode at VID:PID `0bda:1a2b`,
   which is the mode the silicon enumerates in on first plug before
   the firmware switches to Wi-Fi mode. Addresses a first-plug
   customer-facing usability issue that is independent of switch-mode.

### Diffing the shipped conf against Nick's upstream

To verify the attribution yourself, compare the files in this directory
against the conf templates in Nick's source repos:

```sh
curl -sSL https://raw.githubusercontent.com/morrownr/rtl8852cu-20251113/main/8852cu.conf -o /tmp/nick-8852cu.conf
diff /tmp/nick-8852cu.conf evidence/downstream-vendor/8852cu.conf
```

The diff will show exactly the two vendor tweaks and nothing else of
substance.

## Why this evidence matters

Strong signal: a commercial vendor ships the USB 2-to-3 switch-mode
parameter enabled by default on every customer install. That default
is only meaningful because Bitterblue Smith's switch-mode commits in
`morrownr/rtw89` (`cd287cc` for AX chips, `c8a8ac4` for BE chips)
implement the register write that the parameter gates. Without those
commits in the loaded module, the parameter is academic. BrosTrend's
DKMS pipeline compiles against Nick's OOT rtw89 fork precisely because
it carries that implementation.

Weak signal (do not over-read): the `blacklist rtw89_8852cu` and
`blacklist rtw89_8852bu` directives in the shipped conf are NOT a
vendor statement about the in-kernel driver. They are part of Nick's
OOT driver packaging hygiene: the in-kernel driver must be unloaded
for the DKMS-built OOT module to take over a matching device, and
that is standard OOT-driver practice rather than a commercial
judgment.

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
