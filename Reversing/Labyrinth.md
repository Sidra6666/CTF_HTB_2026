# 🔐 Labyrinth — Pwn CTF Write-up


## 📝 Challenge Description

> You find yourself trapped in a mysterious labyrinth, with only one chance to escape. Choose the correct door wisely, for the wrong choice could have deadly consequences.

The challenge presents a labyrinth containing 100 doors. The objective is to escape the labyrinth and retrieve the hidden flag.

Instead of solving the maze through normal gameplay, we analyze the executable and exploit a stack-based buffer overflow to redirect program execution to a hidden function.

---

## 🎯 Objective

Identify the vulnerability in the binary and exploit it to execute the hidden `escape_plan()` function, which reveals the flag.

## 🔍 Step 1: Connecting to the Server

We started by connecting to the challenge server using Netcat.

```bash
nc 154.57.164.70 37177
```

The server displayed a list of 100 doors:

```text
Select door:

Door: 001
Door: 002
Door: 003
...
Door: 069
...
Door: 100

>>
```

After selecting a door, the program displayed:

```text
You are heading to open the door but you suddenly see something on the wall:

"Fly like a bird and be free!"

Would you like to change the door you chose?

>>
```

This second input prompt became important during exploitation.

---

## 🔎 Step 2: Analy
