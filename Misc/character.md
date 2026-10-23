# Character — Misc CTF

## Challenge Details
- **Category:** Misc
- 
## Commands

Connect to the challenge manually:

```bash
nc 154.57.164.71 32169
```

Create the Python script:

```bash
nano sn.py
```

Run the script:

```bash
python3 sn.py
```

## Python Exploit

The challenge returns one flag character for each integer index. This script reconnects for each index, requests the character, and builds the flag.

```python
import socket
import re
import time

HOST = "154.57.164.71"
PORT = 32169

flag = ""

for i in range(150):
    try:
        with socket.create_connection((HOST, PORT), timeout=10) as s:
            s.settimeout(10)

            data = b""
            while b"Enter an index:" not in data:
                chunk = s.recv(4096)
                if not chunk:
                    raise ConnectionError("Disconnected")
                data += chunk

            s.sendall(f"{i}\n".encode())

            response = b""
            while True:
                try:
                    chunk = s.recv(4096)
                except socket.timeout:
                    break

                if not chunk:
                    break

                response += chunk

                if b"Enter an index:" in response:
                    break

            text = response.decode(errors="replace")
            match = re.search(
                rf"Character at Index {i}:\s*(.)",
                text
            )

            if not match:
                print(f"\nCouldn't read index {i}: {text!r}")
                break

            char = match.group(1)
            flag += char
            print(char, end="", flush=True)

            if char == "}":
                break

            time.sleep(0.1)

    except (OSError, ConnectionError) as e:
        print(f"\nError at index {i}: {e}")
        break

print("\nFlag:", flag)
```

## Key Takeaway
Automate repetitive index-based requests with Python sockets and regular expressions instead of entering each index manually.
