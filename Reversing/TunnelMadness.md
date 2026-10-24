# 🔐 TunnelMadness — Reverse Engineering CTF Write-up



## 📝 Challenge Description

Within Vault 8707 are master keys capable of accessing any vault in the country. Unfortunately, the entrance collapsed long ago. Rumors suggest that a few survivors escaped through an underground tunnel.

Our objective is to reverse-engineer the provided executable, understand the underground maze, and discover a valid route into the vault.

## 🎯 Objective

Analyze the compiled binary, identify the maze structure, calculate a valid path, and retrieve the flag from the remote server.

Unlike binary exploitation challenges, this challenge focuses on understanding the program's internal logic rather than exploiting a memory corruption vulnerability.

## 🔍 Step 1: Binary Analysis

After downloading and extracting the challenge files, we inspected the executable.

```bash
file tunnel
```

The binary was identified as a 64-bit Linux ELF executable.

Next, we extracted readable strings:

```bash
strings tunnel
```

Interesting strings included:

```text
/flag.txt
HTB{fake_flag_for_testing}
You break into the vault and read the secrets within...
```

The fake flag was included for local testing.

To inspect the available symbols:

```bash
nm tunnel
```

We identified a function named:

```text
get_flag
```

This indicated that the program contained functionality for retrieving the flag.

## 🧩 Step 2: Understanding the Maze

The challenge contains a three-dimensional maze with dimensions:

```text
20 × 20 × 20
```

Total positions:

```text
20 × 20 × 20 = 8000
```

The starting position is:

```python
start = (0, 0, 0)
```

The destination is:

```python
goal = (19, 19, 19)
```

The program accepts six movement commands:

| Command | Direction |
|---|---|
| L | Left |
| R | Right |
| F | Forward |
| B | Backward |
| U | Up |
| D | Down |

The challenge is to navigate through the maze while avoiding blocked positions.

## 🧠 Step 3: Understanding Breadth-First Search

Instead of manually guessing the correct movements, we used **Breadth-First Search (BFS)**.

BFS is a graph traversal algorithm that explores positions level by level.

It is particularly useful for maze-solving problems because it finds the shortest path when every movement has the same cost.

The algorithm works as follows:

1. Start at the initial position.
2. Add the starting position to a queue.
3. Explore all accessible neighboring positions.
4. Mark visited positions to avoid repeated exploration.
5. Continue exploring until the destination is reached.
6. Reconstruct the path from the start to the destination.

### Why BFS?

The maze contains 8,000 possible positions.

Trying every movement combination manually would be inefficient.

BFS systematically explores accessible positions without repeatedly visiting the same cells.

## 💻 Step 4: Implementing BFS in Python

The following code demonstrates the core BFS algorithm used for three-dimensional maze navigation.

```python
from collections import deque

directions = {
    "R": (1, 0, 0),
    "L": (-1, 0, 0),
    "F": (0, 1, 0),
    "B": (0, -1, 0),
    "U": (0, 0, 1),
    "D": (0, 0, -1)
}

def bfs(start, goal, is_open):

    queue = deque([(start, "")])
    visited = {start}

    while queue:

        (x, y, z), path = queue.popleft()

        if (x, y, z) == goal:
            return path

        for direction, (dx, dy, dz) in directions.items():

            nx = x + dx
            ny = y + dy
            nz = z + dz

            next_position = (nx, ny, nz)

            if (
                0 <= nx < 20
                and 0 <= ny < 20
                and 0 <= nz < 20
                and next_position not in visited
                and is_open(nx, ny, nz)
            ):

                visited.add(next_position)

                queue.append(
                    (next_position, path + direction)
                )

    return None
```

**Note:** This demonstrates the BFS logic. The `is_open()` function must be implemented using the maze data extracted from the binary. The coordinate mapping must also match the program's movement logic.

## 🚀 Step 5: Connecting to the Server

We connected to the remote challenge using Netcat:

```bash
nc 154.57.164.64 32747
```

The program accepted directional movement commands.

After analyzing the maze, the following route was calculated:

```text
UUURFURURRFRRFFUUFURRUFUFFRFUFUUUUFFRRUUUFURFDFFUFFRRRRRFRR
```

## 🐍 Step 6: Automating the Solution

Rather than entering each movement manually, we created a Python script to send the calculated path to the server.

```python
import socket

HOST = "154.57.164.64"
PORT = 32747

path = "UUURFURURRFRRFFUUFURRUFUFFRFUFUUUUFFRRUUUFURFDFFUFFRRRRRFRR"

with socket.create_connection((HOST, PORT), timeout=10) as s:

    s.settimeout(10)

    s.sendall(("\n".join(path) + "\n").encode())

    while True:
        try:
            data = s.recv(4096)

            if not data:
                break

            print(data.decode(errors="replace"), end="")

        except socket.timeout:
            break
```

Run the script using:

```bash
python3 solve.py
```

The script establishes a TCP connection and sends each movement command on a separate line.

## 🏁 Step 7: Retrieving the Flag

The objective was to reach the destination coordinates:

```text
(19, 19, 19)
```

Upon reaching the destination, the program executes the functionality responsible for retrieving the flag.

The flag itself is intentionally omitted from this write-up.

## 📚 Key Concepts Learned

### 1. Reverse Engineering

Reverse engineering involves analyzing a compiled program to understand its internal functionality without relying on its original source code.

### 2. ELF Binary Analysis

ELF (Executable and Linkable Format) is commonly used for executable files on Linux systems.

### 3. Three-Dimensional Arrays

A 3D maze can be represented using three coordinates:

```python
(x, y, z)
```

Each coordinate identifies a specific position in the maze.

### 4. Breadth-First Search

BFS explores a graph level by level and finds a shortest path in an unweighted graph.

### 5. Queue Data Structure

BFS uses a queue to process positions in first-in, first-out (FIFO) order.

### 6. Python Socket Programming

Python's `socket` module allows communication with remote services using TCP connections.

## ✅ Conclusion

TunnelMadness demonstrates how reverse engineering and algorithmic problem-solving can be combined to solve complex CTF challenges.

By analyzing the executable, understanding the maze representation, and applying Breadth-First Search, we can calculate a route through the underground tunnel.

**Key Takeaway:** Reverse engineering is not always about finding vulnerabilities. Sometimes, the challenge is understanding a program's internal logic and using algorithms to solve it.

---

**Category:** Reverse Engineering  
**Technique:** Binary Analysis + BFS  
**Language:** Python  
**Platform:** Hack The Box
