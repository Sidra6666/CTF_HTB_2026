# Dynastic — Trithemius Cipher

## 📌 CTF Overview
- **Category:** Crypto
- **Vulnerability:** Classical Cipher
- **Tool Used:** Python 3

## 🎯 Objective
Decrypt the encrypted message to recover the passcode and retrieve the HTB flag.

## 💻 Solution — Using Terminal

### Step 1: Extract the Challenge Files
```bash
unzip crypto_dynastic.zip
cd crypto_dynastic
ls
```

### Step 2: View the Encrypted Text
```bash
cat output.txt
```

### Step 3: Decrypt the Message
```bash
python3 -c 's=open("output.txt").read().splitlines()[1]; print("".join(chr((ord(c)-65-i)%26+65) if c.isalpha() else c for i,c in enumerate(s)))'
```

### 🏁 Flag
```text
HTB{DID_YOU_KNOW_ABOUT_THE_TRITHEMIUS_CIPHER?!_IT_IS_SIMILAR_TO_CAESAR_CIPHER}
```

## ✅ Conclusion
The Dynastic challenge demonstrates the Trithemius cipher, which shifts letters according to their positions in the message. Reversing these shifts reveals the plaintext.
