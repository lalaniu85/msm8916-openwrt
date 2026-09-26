# 410 / UF02 verified firmware backup

This directory documents the two firmware assets published in the GitHub Release
`uf02-410-firmware-backup-2026-09-26`. Download the assets from that release,
then verify them against [SHA256SUMS](SHA256SUMS) before use.

## Exact device scope

- **Device:** UF02 4G Modem Stick (also referred to as “410” in the hardware setup)
- **Board name reported by the device:** `uf02,250605v0s`
- **OpenWrt normalized board:** `uf02-250605v0s`
- **SoC:** Qualcomm **MSM8916** (Snapdragon 410 family), 64-bit ARM

The SoC is not inferred only from the OpenWrt target name: the UF02 device-tree
source declares `model = "UF02 4G Modem Stick"` and
`compatible = "uf02,250605v0s", "qcom,msm8916"`. Do not use either asset for
UZ801 or another MSM8916-family dongle merely because the chip family is similar.

## Firmware 1 — base connectivity / Wi-Fi image

| Field | Verified value |
| --- | --- |
| Release asset | `openwrt_firmware_uf02_v25.12.5_202607062018.zip` |
| SHA256 | `01cb253adcd73d70cfe413086932b0312e131e8b2104a08d46b6ee1722b2d250` |
| OpenWrt version | 25.12.5 (archive name); manifest `base-files` revision `f5dae5ece4` |
| Format | Full EDL package: GPT, Android boot image, SquashFS system image, Qualcomm boot-firmware bundle, and flash script |
| Verified functions in manifest | ModemManager, QMI utilities, Qualcomm modem remoteproc, WCN36xx Wi-Fi, firewall4 |
| OpenClash | Not included |

This is the only complete, device-specific base EDL package found in the local
`410-openwrt-flash` recovery workspace. It best matches the earlier firmware
that the owner reports was used successfully for normal network access and Wi-Fi
configuration. The archive itself cannot prove a particular SIM, carrier, or
past Wi-Fi session; that prior successful use remains a runtime observation.

This is **not** a `sysupgrade` file. Its included EDL script rewrites the GPT,
boot and read-only system image, and erases `rootfs_data`. It should only be used
in a deliberate recovery procedure with EDL access and a known-good backup. It
must never be applied through `sysupgrade`, and it is not a routine repair for a
working device.

## Firmware 2 — OpenClash / TProxy sysupgrade image

| Field | Verified value |
| --- | --- |
| Release asset | `openwrt-msm89xx-msm8916-generic-uf02-squashfs-sysupgrade.bin` |
| SHA256 | `5b58604252da00776a4ce4c9200feb9b6ec1ecc4163b68677ef57bb18e91eab3` |
| OpenWrt version | 25.12.5 `r33051-f5dae5ece4` |
| Target / board | `msm89xx/msm8916`, `uf02-250605v0s` |
| Supported device metadata | `uf02,250605v0s` |
| Verified packages | `dnsmasq-full`, `kmod-inet-diag`, `kmod-nf-conntrack-netlink`, `kmod-nft-tproxy`, `luci-app-openclash 0.47.156` |

The sysupgrade archive was checked for the exact required members:
`sysupgrade-uf02-250605v0s/CONTROL`, `kernel`, and `root`. It also carries
matching fwtool metadata for `uf02,250605v0s`.

For an already healthy matching UF02 installation, validate first with
`sysupgrade -T <file>` and only then run ordinary `sysupgrade <file>`. Do not
use `sysupgrade -F` to override a failed compatibility check. This asset is not
an EDL recovery image.

## Observations after Firmware 2 was previously run

The device was later observed to report EXT4 filesystem errors on
`/dev/mmcblk0p15` and temperatures around 97°C. These are runtime observations;
they are **not** evidence that this firmware caused either condition. Do not
flash either asset as a substitute for diagnosing storage integrity, cooling, or
power delivery.

## Privacy and archive boundary

Only the two reproducible firmware assets and their checksums are released.
The following local data is deliberately excluded: the 3.9 GB full eMMC backup,
radio partitions (`modem`, `modemst*`, `fsg`, `fsc`, `persist`, `sec`), QCN files,
and any runtime overlay. Those files can contain IMEI, SIM/radio state, Wi-Fi
SSID/passwords, or other private device data.

Firmware 1 contains no `rootfs_data`, overlay, or saved radio-partition files;
its own EDL script erases `rootfs_data`. Consequently, a Wi-Fi SSID/password set
after installing Firmware 1 was runtime configuration and is not embedded in
the released archive. No SSID, password, subscription URL, token, SIM identifier,
or IMEI is included in this release.
