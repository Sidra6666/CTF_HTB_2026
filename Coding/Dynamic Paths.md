# 🗺️ Dynamic Paths — Coding Challenge

## 📌 Overview

- **Challenge:** Dynamic Paths
- **Category:** Coding
- **Target:** `154.57.164.75:32507`
- **Language:** Python 3
- **Status:** ✅ Solved

The challenge involves finding the minimum-cost path through a grid of tunnel distances. Starting at the top-left corner, we must reach the bottom-right corner while moving only right or down.

There are 100 grids to solve. Each grid requires calculating and submitting its minimum path sum to obtain the final flag.

## 🎯 Challenge Objective

For every grid, we are given its dimensions and the distance values of its cells. We must determine the path with the smallest total distance.

Example:

**Input**
```text
4 3
2 5 1 9 2 3 9 1 3 11 7 4
```

**Grid**
```text
 2  5  1
 9  2  3
 9  1  3
11  7  4
```

**Output**
```text
17
```

The optimal path is `2 → 5 → 2 → 1 → 3 → 4`, with a total cost of 17.

## 🧠 Solution Approach

I used **Dynamic Programming (DP)** to solve the minimum path sum problem.

For each cell, the algorithm calculates the minimum cost of reaching it from either the top or the left:

```python
dp[j] = grid[i][j] + min(dp[j], dp[j - 1])
```

- `dp[j]` stores the cost from above.
- `dp[j - 1]` stores the cost from the left.
- The smaller cost is selected, and the current cell's distance is added.

The implementation uses a one-dimensional array to reduce extra space complexity.

## 💻 Full Python Program

Save this code as `dynamic_paths.py`.

```python
import socket
import re

HOST = "154.57.164.75"
PORT = 32507
TOTAL = 100


def minimum_path(grid):
    rows = len(grid)
    cols = len(grid[0])

    dp = [0] * cols
    dp[0] = grid[0][0]

    for c in range(1, cols):
        dp[c] = dp[c - 1] + grid[0][c]

    for r in range(1, rows):
        dp[0] += grid[r][0]

        for c in range(1, cols):
            dp[c] = grid[r][c] + min(
                dp[c],
                dp[c - 1]
            )

    return dp[-1]


def main():
    with socket.create_connection(
        (HOST, PORT), timeout=30
    ) as sock:

        sock.settimeout(30)
        f = sock.makefile("rwb", buffering=0)

        for test in range(1, TOTAL + 1):

            # Wait for the test marker
            while True:
                line = f.readline()

                if not line:
                    print("Connection closed.")
                    return

                text = line.decode(
                    errors="replace"
                ).strip()

                print(text)

                if re.search(
                    r"Test\s+\d+\s*/\s*100",
                    text,
                    re.I
                ):
                    break

            # Read grid dimensions
            while True:
                line = f.readline()

                if not line:
                    print("Missing dimensions.")
                    return

                text = line.decode(
                    errors="replace"
                ).strip()

                match = re.fullmatch(
                    r"(\d+)\s+(\d+)", text
                )

                if match:
                    rows, cols = map(
                        int, match.groups()
                    )
                    break

            # Read all grid values
            numbers = []

            while len(numbers) < rows * cols:
                line = f.readline()

                if not line:
                    print("Incomplete grid.")
                    return

                numbers.extend(
                    map(
                        int,
                        re.findall(
                            r"\d+",
                            line.decode(
                                errors="replace"
                            )
                        )
                    )
                )

            numbers = numbers[:rows * cols]

            grid = [
                numbers[r * cols:(r + 1) * cols]
                for r in range(rows)
            ]

            answer = minimum_path(grid)

            print(
                f"Grid {test}: minimum sum = {answer}"
            )

            # Submit the calculated answer
            f.write(f"{answer}\n".encode())

        # Read the final server response
        try:
            while True:
                line = f.readline()

                if not line:
                    break

                print(
                    line.decode(errors="replace"),
                    end=""
                )

        except (socket.timeout, OSError):
            pass


if __name__ == "__main__":
    main()
```

## 🚀 How to Run

**1.** Make sure Python 3 is installed.

**2.** Save the program as `dynamic_paths.py`.

**3.** Run it from your terminal:

```bash
python3 dynamic_paths.py
```

The program connects to the challenge server, reads the grids, calculates the minimum path sums, and submits the answers.

## 🛠️ Tools Used

- Python 3
- `socket` — TCP communication with the remote service
- `re` — parsing challenge output
- Dynamic Programming — minimum-cost path calculation
- Terminal — running and testing the script

## 📈 Complexity Analysis

For a grid with \(R\) rows and \(C\) columns:

- **Time complexity:** \(O(R \times C)\)
- **Auxiliary space complexity:** \(O(C)\)

Each cell is processed once, and only one row of DP values is stored.

## 🏁 Final Result

Successfully completed the 100-grid challenge and obtained the flag. 🎉

**Concepts practiced:** Dynamic Programming · Python · TCP Sockets · Algorithm Optimization · CTF Problem Solving

