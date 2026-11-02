# 🕵️ Phreaky — Network Forensics

**Platform:** Hack The Box  
**Category:** Forensics  
**Difficulty:** Easy  
**Tools:** Wireshark · Terminal · Base64 · 7-Zip

---

## 📌 Challenge Overview

A suspicious network capture contains evidence of confidential information being transferred between two organizations. The objective is to investigate the captured traffic, identify unusual email activity, recover the transferred files, and uncover the hidden information.

## 🎯 Objective

- Examine the supplied PCAP file.
- Investigate SMTP communication.
- Identify suspicious email attachments.
- Recover password-protected ZIP archives.
- Reconstruct the original document and locate the flag.

## 🛠️ Investigation

### 1. Preparing the Capture

The challenge provided a ZIP archive containing a packet capture file.

After extracting the archive, I opened `phreaky.pcap` in Wireshark to inspect the recorded network traffic.

### 2. Inspecting Network Conversations

I focused on SMTP traffic because email communication can expose message contents and attachments.

Useful Wireshark filter:

```wireshark
smtp
```

By following individual TCP streams, I discovered multiple email messages containing ZIP attachments.

The messages also included passwords required to access the attached archives.

### 3. Recovering the Attachments

The email attachments were encoded using Base64.

I examined the email streams and collected the encoded attachment data along with their corresponding passwords.

The recovered data was decoded and saved as individual ZIP files.

Example:

```bash
echo "BASE64_DATA" | base64 -d > recovered.zip
```

To extract a password-protected archive:

```bash
7z x recovered.zip -pPASSWORD
```

### 4. Reconstructing the Document

Each successfully extracted archive contained a separate fragment of the same PDF document.

After recovering all the fragments, I arranged them in numerical order and combined them.

```bash
ls phreaks_plan.pdf.part* | sort -V | xargs cat > phreaks_plan_combined.pdf
```

This produced the complete PDF document.

### 5. Finding the Flag

I opened the reconstructed PDF and examined its contents.

The document revealed the information required to complete the challenge.

---

## 🧠 Key Takeaways

This challenge helped me practise:

- Analysing PCAP files using Wireshark.
- Understanding SMTP traffic and TCP streams.
- Identifying email attachments in network captures.
- Working with Base64 encoding and decoding.
- Extracting password-protected ZIP files.
- Reconstructing files from fragmented data.

## 🏁 Challenge Result

**Flag:**

```text
HTB{PASTE_YOUR_FLAG_HERE}
```

---

*Personal lab notes — documenting my hands-on learning in network forensics and packet analysis.*
