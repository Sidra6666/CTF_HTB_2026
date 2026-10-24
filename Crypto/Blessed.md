# 🔐 Blessed — Hack The Box



### Description

The challenge involves a SuperComputer that manages multiple robots using Boneh–Lynn–Shacham (BLS) digital signatures.

The system claims that sensitive operations require approval from all verified robots.

Our objective was to bypass this security mechanism and retrieve the hidden flag.

## 🔍 Step 1: Connect to the Server

Initially, we connected to the challenge using Netcat.

```bash
nc 154.57.164.74 30938
```

The server displayed a menu containing several commands:

```json
{
  "help": "Show this panel",
  "create": "Generate a new robot, already verified",
  "join": "Add a new robot, given a public key and a signature",
  "verify": "Start interactive process to verify a robot given an ID",
  "list": "Return a list of all existing robots",
  "unveil_secrets": "Show the secrets given an aggregated signature of all registered robots",
  "exit": "Shutdown the SuperComputer"
}
```

The most important command was:

```json
{"cmd":"unveil_secrets"}
```

However, this operation required a valid aggregated BLS signature from the verified robots.

---

## 🤖 Step 2: Create a Robot

We generated a new robot using:

```json
{"cmd":"create"}
```

The server returned:

```json
{
  "msg": "Do not lose your secret key!",
  "sk": "<private_key>",
  "robot_id": "<robot_id>",
  "pk": "<public_key>"
}
```

Each robot contained:

- `sk`: Private key used to generate signatures.
- `pk`: Public key used to verify signatures.
- `robot_id`: Unique identifier assigned to the robot.

Our newly created robot was already verified.

---

## 🔑 Step 3: Generate a BLS Signature

We used Python's `py-ecc` library to generate a valid signature.

```python
from py_ecc.bls import G2ProofOfPossession as bls

sk = int("<private_key>", 16)

signature = bls.Sign(sk, b"list")

print(signature.hex())
```

### Explanation

`bls.Sign()` generates a digital signature using the robot's private key.

The message `b"list"` represents the operation being authorized.

The signature was then submitted to the server:

```json
{
  "cmd": "list",
  "robot_id": "<robot_id>",
  "sig": "<generated_signature>"
}
```

The server accepted our signature and returned the registered robots' public keys.

---

## 🧠 Step 4: Analyze the Vulnerability

The SuperComputer uses BLS aggregate signatures.

BLS signatures allow multiple signatures on the same message to be combined into one signature.

The challenge's security depends on every verified robot participating in the approval process.

However, the implementation contains weaknesses involving:

1. Predictable elliptic-curve random-number generation.
2. A zero-knowledge verification protocol that depends on predictable challenge bits.
3. A rogue public-key attack against BLS signature aggregation.

### Elliptic-Curve Random-Number Generator

The server generates robot IDs using an elliptic-curve-based pseudorandom number generator.

Its outputs expose truncated coordinates of consecutive elliptic-curve points.

By recovering the generator's internal state, an attacker can predict future outputs.

This is particularly dangerous because the same generator is used during robot verification.

### Zero-Knowledge Verification

When an unverified robot attempts to become verified, the server performs 64 verification rounds.

In each round, the server randomly chooses one of two questions.

If the random choices are predictable, an attacker can prepare the appropriate responses in advance.

### Rogue Public-Key Attack

The final weakness involves BLS public-key aggregation.

By constructing a malicious public key that cancels the other robots' public keys, an attacker can make the aggregate public key correspond to a private key they control.

After getting the malicious robot verified, the attacker can generate an aggregate signature accepted by the SuperComputer.

---

## 💻 Step 5: Install SageMath

We used SageMath to perform the elliptic-curve calculations required by the exploit.

First, we installed Homebrew.

Then we installed SageMath:

```bash
brew install --cask sage
```

We verified the installation:

```bash
sage --version
```

Output:

```text
SageMath version 10.9
```

SageMath provides advanced mathematical functionality, including elliptic-curve arithmetic and lattice calculations.

---

## 📦 Step 6: Install Python Dependencies

We installed the required cryptographic library:

```bash
sage -pip install py-ecc
```

We also installed Pwntools:

```bash
sage -pip install --no-deps pwntools
```

Additional dependencies were installed to resolve compatibility issues.

These libraries enabled the exploit to communicate with the server and generate BLS signatures.

---

## 📥 Step 7: Obtain the Exploit

We used an existing public exploit implementation by **7Rocky**.

```bash
curl -fL https://raw.githubusercontent.com/7Rocky/HackTheBox-scripts/main/Challenges/Crypto/Blessed/solve.py -o solve.py
```

The script automates the cryptographic attack.

Its main stages include:

1. Connecting to the SuperComputer.
2. Retrieving robot information.
3. Recovering the elliptic-curve generator state.
4. Predicting the verification challenges.
5. Registering and verifying a rogue public key.
6. Constructing a valid aggregate signature.
7. Calling `unveil_secrets` to retrieve the flag.

**Exploit credit:** 7Rocky — HackTheBox-scripts, Crypto/Blessed.

---

## 🚀 Step 8: Execute the Exploit

Our original challenge instance became unavailable.

The connection test returned:

```text
Connection refused
```

We restarted the challenge and received a new IP address and port.

We then executed:

```bash
sage -python solve.py 154.57.164.80:37214
```

### Command Explanation

- `sage`: Runs SageMath.
- `-python`: Executes a Python script using SageMath's Python environment.
- `solve.py`: The exploit script.
- `154.57.164.80`: Remote challenge IP address.
- `37214`: Remote challenge port.

The exploit connected to the challenge and successfully retrieved the flag.

### Result

```text
[+] Challenge solved successfully!
[+] Flag retrieved: HTB{REDACTED}
```

*The output above is illustrative; the actual flag is intentionally omitted.*

**Status: SOLVED ✅**

---

## 🛡️ Security Lessons Learned

This challenge demonstrates several important cryptographic security concepts.

### 1. Random-Number Generation Matters

Cryptographic protocols require unpredictable random values.

Predictable randomness can compromise otherwise secure mathematical constructions.

### 2. Zero-Knowledge Protocols Depend on Unpredictable Challenges

A verification protocol can become insecure when attackers can predict the verifier's questions.

### 3. BLS Aggregation Requires Protection Against Rogue Keys

BLS aggregate signatures must be implemented with appropriate protections against malicious public-key registration.

Proof-of-possession mechanisms or other secure aggregation designs can help prevent rogue-key attacks.

### 4. Secure Cryptography Requires Secure Implementation

Even strong cryptographic algorithms can fail when combined with insecure random-number generators or flawed verification protocols.

## 🏁 Conclusion

The Blessed challenge demonstrated how weaknesses in random-number generation and public-key verification can undermine a cryptographic authentication system.

By combining elliptic-curve analysis, predictable verification challenges, and a rogue public-key attack, the SuperComputer's aggregate signature requirement was bypassed.

This challenge provided practical experience with cryptographic exploitation, Python automation, and SageMath.

**Challenge completed successfully! 🎉**
