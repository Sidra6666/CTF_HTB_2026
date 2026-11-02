# HTB – Silicon Data Sleuthing

## Challenge Description

In the dust and sand surrounding the vault, a rusty PCB is discovered. The etched markings reveal that it is an OpenWrt router. Fortunately, the ROM chip is still intact, allowing the hardware team to extract its firmware image.

The objective is to analyze the firmware, understand its filesystem structure, recover router configuration details, and identify sensitive information that could help bypass the vault's security mechanisms.


# 1. Understanding the Firmware

The challenge provides an OpenWrt firmware image.

Before analyzing the firmware, we need tools to extract and inspect its filesystems.

### Tools used

| Tool | Purpose |
|---|---|
| Binwalk | Identifies and extracts embedded firmware components |
| Sasquatch | Extracts modified SquashFS filesystems |
| Jefferson | Extracts JFFS2 filesystems |
| grep | Searches for specific configuration values |
| strings | Extracts readable text from binary data |
| find | Locates files and directories |
| tree | Displays filesystem structure |

### Extracting the firmware

```bash
binwalk -e firmware.bin
```

Binwalk identifies embedded filesystem components and attempts to extract them.

Depending on the firmware, additional extraction tools such as Sasquatch and Jefferson may be required.

After extraction, the directory structure looks similar to:

```text
.
├── jffs2-root
├── squashfs-root
└── squashfs-root-0
```

## 2. Understanding SquashFS and JFF
