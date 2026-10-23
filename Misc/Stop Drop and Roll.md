# 🎮 HTB — Stop Drop and Roll

**Category:** Misc

<img src="stopdropandroll.png" alt="Stop Drop and Roll" width="500">

## 📝 Challenge Description

In this challenge, we connect to a remote game server where we must react to different scenarios. The server sends one or more keywords, and we must respond with the correct actions to progress through the game and retrieve the flag.

## 🎯 Objective

Connect to the challenge service, interpret the scenarios, and automate responses until the flag is received.

## 🔑 Scenario Mappings

| Scenario | Required Action |
|---|---|
| `GORGE` | `STOP` |
| `PHREAK` | `DROP` |
| `FIRE` | `ROLL` |

When multiple scenarios appear, combine the corresponding actions using hyphens (`-`).

**Example:**

```text
Input:  GORGE, FIRE, PHREAK
Output: STOP-ROLL-DROP
```

## 🛠️ Tools Used

- Python 3
- TCP sockets
- Regular expressions
- Linux/macOS terminal
- Basic network protocol analysis

## 💻 Solution

The challenge can be automated using Python and the `socket` library to communicate with the remote server. Regular expressions extract the scenarios, and a dictionary maps each scenario to its required action.

### Python Script

```python
import socket
import re

HOST = "154.57.164.82"
PORT = 38220

actions = {
    "GORGE": "STOP",
    "PHREAK": "DROP",
    "FIRE": "ROLL"
}

with socket.create_connection((HOST, PORT), timeout=15) as s:
    s.settimeout(30)
    buffer = b""

    while True:
        try:
            chunk = s.recv(4096)
            if not chunk:
                break

            buffer += chunk
            text = buffer.decode("utf-8", errors="ignore")

            if "Are you ready? (y/n)" in text:
                s.sendall(b"y\n")
                buffer = b""
                continue

            if "What do you do?" in text:
                prompt, buffer = buffer.rsplit(
                    b"What do you do?", 1
                )

                scenarios = re.findall(
                    r"\b(GORGE|PHREAK|FIRE)\b",
                    prompt.decode("utf-8", errors="ignore")
                )

                answer = "-".join(actions[x] for x in scenarios)

                print("Answer:", answer)
                s.sendall((answer + "\n").encode())

        except socket.timeout:
            print("Timed out waiting for server.")
            print(buffer.decode("utf-8", errors="ignore"))
            break
```

### Execution

Save the script as `solve.py` and run:

```bash
python3 solve.py
```

### How It Works

1. **Connect:** Establishes a TCP connection to the challenge server.
2. **Start:** Sends `y` to begin the game.
3. **Extract:** Uses regular expressions to identify `GORGE`, `PHREAK`, and `FIRE`.
4. **Map:** Converts each scenario into `STOP`, `DROP`, or `ROLL`.
5. **Respond:** Joins multiple actions with hyphens and sends the answer.
6. **Repeat:** Continues responding until the challenge finishes.

## 🧠 Key Takeaways

- Automating repetitive CTF tasks with Python.
- Using TCP sockets for client-server communication.
- Extracting keywords with regular expressions.
- Understanding simple protocol-based challenges.

## 🏁 Flag

```text
HTB{...}
```

**Challenge completed!** 🚀
