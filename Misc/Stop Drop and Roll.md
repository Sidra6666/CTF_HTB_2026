# 🎮 HTB — Stop Drop and Roll

**Category:** Misc

<img src="stopdropandroll.png" alt="Stop Drop and Roll" width="500">

## 📝 Challenge Description

In this challenge, we connect to a remote game server where we must react to different scenarios. The server sends one or more keywords, and we must respond with the correct actions to progress through the game and retrieve the flag.

## 🎯 Objective

Connect to the challenge service, interpret the scenarios, and automate responses until the flag is received.

## 🔑 Scenario Mappings

| Scenario | Required Action |
| -------- | --------------- |
| `GORGE`  | `STOP`          |
| `PHREAK` | `DROP`          |
| `FIRE`   | `ROLL`          |

When multiple scenarios appear, combine the corresponding actions using hyphens (`-`).

**Example:**

```text
Input:  GORGE, FIRE, PHREAK
Output: STOP-ROLL-DROP
```

## 💻 Solution

### How It Works

1. **Connect:** Python's `socket` library establishes a TCP connection to the remote challenge server.
2. **Start:** The script sends `y` when the server asks whether we are ready.
3. **Receive:** The script reads the data sent by the server.
4. **Extract:** Regular expressions identify the scenario keywords: `GORGE`, `PHREAK`, and `FIRE`.
5. **Map:** A dictionary converts each scenario into its corresponding action.
6. **Respond:** The actions are joined with hyphens (`-`) and sent back to the server.
7. **Repeat:** The script continues answering until the game ends or the connection closes.

### Python Script

```python
import socket
import re

HOST = "154.57.164.65"
PORT = 40160

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


## 🧠 Key Takeaways

- Automating repetitive CTF tasks with Python.
- Using TCP sockets for client-server communication.
- Extracting keywords with regular expressions.
- Mapping inputs to outputs using a dictionary.
- Understanding basic protocol automation.

## 🏁 Flag

```text
HTB{...}
```

**Challenge completed!** 🚀
