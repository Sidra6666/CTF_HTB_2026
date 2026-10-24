# 🛰️ Satellite Hijack — Hack The Box

**Category:** Reversing  
**Difficulty:** Easy  
**Platform:** Hack The Box (HTB)  
**Challenge:** Satellite Hijack

## 📖 Challenge Description

The crew discovers an abandoned bunker containing a satellite communication control panel. The software has been tampered with, preventing legitimate satellite control codes from being transmitted.

Our objective is to reverse-engineer the executable, identify the hidden instructions, and recover the flag.

## 🛠️ Tools Used

- macOS Terminal
- Python 3
- `strings`
- `grep`
- `xxd`
- XOR Decoding

## 🔍 Step 1: Extracting the Challenge Files

First, we extracted the downloaded ZIP file.

```bash
unzip rev_satellitehijack.zip
cd rev_satellitehijack
ls -la
```

The challenge contained two important files:

```text
satellite
library.so
```

- `satellite` — Main Linux executable.
- `library.so` — Shared library containing additional functionality.

## 🔍 Step 2: Searching for Hidden Strings

We started by searching for readable strings inside both files.

```bash
strings library.so | grep -Ei 'flag|satellite|code|key|secret'
```

Output:

```text
send_satellite_message
```

Next:

```bash
strings satellite | grep -Ei 'flag|satellite|code|key|transmit'
```

Output:

```text
send_satellite_message
READY TO TRANSMIT
send_satellite_message
```

No complete flag was visible.

This suggested that the flag or relevant instructions might be obfuscated.

## 🔐 Step 3: Identifying XOR Obfuscation

During analysis, we identified the use of:

```c
memfrob()
```

`memfrob()` performs XOR on data using the hexadecimal value `0x2A`.

The operation is reversible:

```python
decoded_byte = encoded_byte ^ 0x2A
```

Applying the same XOR operation again restores the original data.

## 💻 Step 4: Decoding the Hidden Section

We used Python to extract and decode an obfuscated section of `library.so`.

```python
from pathlib import Path

data = Path("library.so").read_bytes()

start = 0x11A9
end = 0x21A9

encrypted = data[start:end]

decoded = bytes(byte ^ 0x2A for byte in encrypted)

Path("decoded.bin").write_bytes(decoded)

print("Decoded", len(decoded), "bytes")
```

Output:

```text
Decoded 4096 bytes
```

The decoded content was saved as:

```text
decoded.bin
```

This allowed us to inspect instructions that were previously hidden.

## 🚩 Step 5: Recovering the First Part of the Flag

We searched the decoded binary for readable strings.

```bash
strings -a -t x decoded.bin
```

Output:

```text
0 AWAVAUATUSH
4b HTB{u
81 []A\A]A^A_
8e l5{0v0Y7H
98 fVf?u>|:H
ac >|:O!|LxH
b6 !o$j,;f
121 AVAUATUSA
147 D$0H
19b []A\A]A^
```

At hexadecimal offset `0x4b`, we discovered:

```text
HTB{u
```

This revealed the flag format:

```text
HTB{...}
```

**First recovered component:**

```text
HTB{
```

The additional `u` belonged to the surrounding binary data rather than the final flag.

## 🔓 Step 6: Recovering the Second Part of the Flag

The remaining flag characters were not stored as ordinary plaintext.

We identified encoded bytes in different sections of the decoded binary.

Using Python, we extracted these bytes and reversed their XOR encoding.

```python
from pathlib import Path

data = Path("decoded.bin").read_bytes()

key = (
    data[0x8e:0x96]
    + data[0x98:0x9d]
    + data[0xac:0xb4]
    + data[0xb6:0xbd]
)

result = bytes(byte ^ i for i, byte in enumerate(key))

print(result.decode())
```

### Explanation

The code extracts byte sequences from four locations:

```text
0x8e
0x98
0xac
0xb6
```

These byte sequences are combined into a single encoded string.

Each byte is then XORed with its index:

```python
byte ^ i
```

This reverses the encoding and reveals the hidden message.

**Second recovered component:**

```text
l4y3r5_0n_l4y3r5_0n_l4y3r5!}
```

## 🏁 Step 7: Combining Both Parts

After decoding, we had recovered two components.

**Part 1 — Flag prefix**

```text
HTB{
```

**Part 2 — Decoded string**

```text
l4y3r5_0n_l4y3r5_0n_l4y3r5!}
```

We combined them to reconstruct the candidate flag.

### 🚩 Final Flag

```text
HTB{l4y3r5_0n_l4y3r5_0n_l4y3r5!}
```

*Note: This candidate flag was recovered through static analysis and has not been confirmed by a successful HTB submission.*

## 🧠 What I Learned

Through this challenge, I explored several reverse-engineering concepts:

1. **Static Analysis:** Examining compiled executables without running them.
2. **Shared Libraries:** Understanding how executables depend on `.so` files.
3. **String Analysis:** Using `strings` and `grep` to locate interesting information.
4. **XOR Obfuscation:** Understanding how XOR can conceal data inside binaries.
5. **Binary Inspection:** Using `xxd` to examine hexadecimal data.
6. **Python Automation:** Writing scripts to extract and decode hidden bytes.
7. **Flag Reconstruction:** Combining recovered components to reconstruct a candidate flag.

## 💡 Conclusion

Satellite Hijack demonstrated how information can be concealed within compiled binaries using multiple layers of obfuscation.

By inspecting the executable and shared library, reversing XOR transformations, and analyzing the decoded bytes, we recovered the components of a potential flag.

**Key Takeaway:** Reverse engineering often requires looking beyond readable strings and examining the underlying binary data to understand hidden program behavior.

---

**Category:** Reverse Engineering | **Tools:** Python, Terminal, strings, xxd | **Platform:** Hack The Box
