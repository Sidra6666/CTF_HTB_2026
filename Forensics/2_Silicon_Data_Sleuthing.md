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

## 2. Understanding SquashFS and JFFS2

An important part of this challenge is understanding how OpenWrt organizes its filesystem.

### SquashFS – Base filesystem

- Contains the base operating system.
- Compressed and read-only.
- Stores system binaries, libraries, and default configurations.
- Mounted as the root filesystem.

### JFFS2 – Writable overlay

- Stores user-modified configurations.
- Provides writable storage on top of the read-only base filesystem.
- Can contain modified passwords, network settings, wireless credentials, and firewall rules.

OpenWrt uses an overlay filesystem to combine the read-only SquashFS filesystem with writable configuration changes.

**Important lesson:** The information in SquashFS is not necessarily the active configuration. When a file is modified, its updated version can exist in the JFFS2 overlay and take precedence.

This distinction becomes especially important when recovering the root password hash.

---

# 3. Challenge Walkthrough

## Q1. What version of OpenWrt runs on the router?

**Answer: `23.05.0`**

The OpenWrt version can be identified from:

```text
squashfs-root/etc/openwrt_release
```

Command:

```bash
cat squashfs-root/etc/openwrt_release
```

Relevant output:

```text
DISTRIB_ID='OpenWrt'
DISTRIB_RELEASE='23.05.0'
DISTRIB_REVISION='r23497-6637af95aa'
DISTRIB_TARGET='ramips/mt7621'
DISTRIB_ARCH='mipsel_24kc'
DISTRIB_DESCRIPTION='OpenWrt 23.05.0 r23497-6637af95aa'
```

The `DISTRIB_RELEASE` field identifies the OpenWrt version.

**Result:** `23.05.0`

## Q2. What is the Linux kernel version?

**Answer: `5.15.134`**

The kernel version can normally be identified using:

```bash
cat /proc/version
```

However, the extracted firmware's `/proc` directory is empty because `/proc` is a virtual filesystem populated at runtime.

An alternative is to inspect:

```bash
ls squashfs-root/lib/modules/
```

OpenWrt stores kernel modules in directories named after the kernel version.

The directory name reveals:

```text
5.15.134
```

**Result:** `5.15.134`

## Q3. What is the hash of the root account's password?

**Answer:**

```text
root:$1$YfuRJudo$cXCiIJXn9fWLIt8WY2Okp1:19804:0:99999:7:::
```

Initially, we inspect:

```bash
cat squashfs-root/etc/shadow
```

However, the hash found in SquashFS is not the correct answer.

Why?

SquashFS is read-only. If the router administrator changes the root password, the updated password hash cannot be written back into the original SquashFS filesystem.

Instead, OpenWrt stores modified configuration files in the writable JFFS2 overlay.

### Inspecting JFFS2

Navigate to:

```bash
cd jffs2-root
```

Inspect the directory structure:

```bash
ls -lah
```

The directory contains entries such as:

```text
1
upper
work
```

The `work` directory contains internal overlay filesystem data. Some files are represented using numbered entries and may not have their original filenames.

We can inspect readable content using:

```bash
cd work/work
strings *
```

Search for the root account:

```bash
strings * | grep 'root:'
```

This reveals the modified root password entry.

**Result:** The root hash must be recovered from the writable overlay rather than relying on the default SquashFS copy.

## Q4. What is the PPPoE username?

**Answer: `yohZ5ah`**

PPPoE stands for **Point-to-Point Protocol over Ethernet**. It is commonly used by internet service providers to authenticate customers connecting through a router.

OpenWrt stores network interface configurations in:

```text
/etc/config/network
```

Inside the extracted JFFS2 data, we locate the network configuration:

```bash
find . -type f -name network
```

Alternatively, search the extracted data for PPPoE-related settings:

```bash
strings * | grep -i -A5 -B5 pppoe
```

The relevant configuration is:

```text
config interface 'wan'
    option device 'wan'
    option proto 'pppoe'
    option username 'yohZ5ah'
    option password 'ae-h+i$i^Ngohroorie!bieng6kee7oh'
    option ipv6 'auto'
```

The `username` field contains the PPPoE login username.

**Result:** `yohZ5ah`

## Q5. What is the PPPoE password?

**Answer:**

```text
ae-h+i$i^Ngohroorie!bieng6kee7oh
```

The same WAN configuration contains the password:

```text
option password 'ae-h+i$i^Ngohroorie!bieng6kee7oh'
```

This demonstrates why router configuration files can contain highly sensitive information.

If an attacker obtains an unprotected firmware backup, they may recover ISP authentication credentials.

**Result:** `ae-h+i$i^Ngohroorie!bieng6kee7oh`

## Q6. What is the Wi-Fi SSID?

**Answer: `VLT-AP01`**

OpenWrt stores wireless settings in:

```text
/etc/config/wireless
```

The configuration contains two wireless radios:

- `radio0` – 2.4 GHz
- `radio1` – 5 GHz

Both interfaces use the same SSID.

Relevant configuration:

```text
config wifi-iface 'default_radio0'
    option device 'radio0'
    option network 'lan'
    option mode 'ap'
    option ssid 'VLT-AP01'
```

The SSID is the name displayed when searching for available Wi-Fi networks.

**Result:** `VLT-AP01`

## Q7. What is the Wi-Fi password?

**Answer:**

```text
french-halves-vehicular-favorable
```

The wireless configuration also contains the encryption type and password.

Relevant configuration:

```text
option encryption 'sae-mixed'
option key 'french-halves-vehicular-favorable'
```

The `key` option stores the wireless network password.

Both the 2.4 GHz and 5 GHz interfaces use the same password.

**Result:** `french-halves-vehicular-favorable`

## Q8. What are the three WAN ports that redirect traffic from WAN to LAN?

**Answer: `1778,2289,8088`**

For this question, we need to inspect the firewall configuration.

OpenWrt firewall rules are stored in:

```text
/etc/config/firewall
```

The JFFS2 working directory does not necessarily represent `/etc/config` as a straightforward directory structure. Files may be stored as internal overlay entries.

We can inspect the readable content using:

```bash
strings *
```

Look for firewall-related keywords:

```bash
strings * | grep -iE 'config redirect|src_dport|dest_ip|DNAT'
```

The relevant firewall rules are:

### Database redirect

```text
config redirect
    option dest 'lan'
    option target 'DNAT'
    option name 'DB'
    option src 'wan'
    option src_dport '1778'
    option dest_ip '192.168.1.184'
    option dest_port '5881'
```

### Web redirect

```text
config redirect
    option dest 'lan'
    option target 'DNAT'
    option name 'WEB'
    option src 'wan'
    option src_dport '2289'
    option dest_ip '192.168.1.119'
    option dest_port '9889'
```

### NAS redirect

```text
config redirect
    option dest 'lan'
    option target 'DNAT'
    option name 'NAS'
    option src 'wan'
    option src_dport '8088'
    option dest_ip '192.168.1.166'
    option dest_port '4431'
```

These rules use DNAT (Destination Network Address Translation) to forward incoming traffic from WAN ports to internal LAN devices.

The three external source ports are:

| Service | WAN port | Internal destination | LAN port |
|---|---:|---|---:|
| DB | 1778 | 192.168.1.184 | 5881 |
| WEB | 2289 | 192.168.1.119 | 9889 |
| NAS | 8088 | 192.168.1.166 | 4431 |

**Result:** `1778,2289,8088`

---

# 4. Alternative Method – Recovering the Configuration Backup

There is another useful discovery in the JFFS2 partition.

Inside:

```text
jffs2-root/upper/
```

we find:

```text
sysupgrade.tgz
```

This is an OpenWrt system upgrade configuration backup.

It can contain the router's configuration files with their original filenames and directory structure.

Extract it using:

```bash
tar -xvzf sysupgrade.tgz
```

Then inspect:

```bash
etc/config/network
etc/config/wireless
etc/config/firewall
etc/shadow
```

Useful commands:

```bash
cat etc/config/network
cat etc/config/wireless
cat etc/config/firewall
cat etc/shadow
```

This method makes it much easier to recover the router's active configuration without manually interpreting the internal JFFS2 overlay structures.

---

# 5. Final Answers

| Question | Answer |
|---|---|
| OpenWrt version | `23.05.0` |
| Linux kernel version | `5.15.134` |
| Root password hash | `root:$1$YfuRJudo$cXCiIJXn9fWLIt8WY2Okp1:19804:0:99999:7:::` |
| PPPoE username | `yohZ5ah` |
| PPPoE password | `ae-h+i$i^Ngohroorie!bieng6kee7oh` |
| Wi-Fi SSID | `VLT-AP01` |
| Wi-Fi password | `french-halves-vehicular-favorable` |
| WAN-to-LAN forwarded ports | `1778,2289,8088` |

---

# 6. Key Takeaways

This challenge demonstrates how much sensitive information can be exposed through an improperly protected router firmware image.

The most important lessons:

1. **Firmware extraction:** Binwalk helps identify embedded filesystems and firmware components.
2. **Filesystem analysis:** Understanding SquashFS and JFFS2 is essential when investigating OpenWrt devices.
3. **Overlay filesystem:** Modified configurations may exist in the writable overlay rather than the read-only root filesystem.
4. **Credential exposure:** Router backups can expose root password hashes, ISP credentials, and Wi-Fi passwords.
5. **Network enumeration:** OpenWrt configuration files reveal WAN, LAN, wireless, and firewall settings.
6. **Port forwarding:** Firewall DNAT rules can expose internal services to external networks.
7. **IoT security:** Firmware images should be treated as sensitive assets because they may contain credentials and configuration secrets.

## Conclusion

The challenge was solved by extracting the OpenWrt firmware, understanding its filesystem layout, and examining both the base filesystem and writable overlay.

The most important discovery was that the active router configuration was not always present in SquashFS. The JFFS2 overlay and the `sysupgrade.tgz` backup provided access to the modified configuration, including the root password hash, PPPoE credentials, Wi-Fi credentials, and firewall forwarding rules.

This challenge highlights why firmware analysis is an important part of IoT and embedded-device security.
