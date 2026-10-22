# Flag Command — Web

## 📌 CTF Overview

- **Platform:** Hack The Box
- **Category:** Web
- **Target:** `http://154.57.164.71:35186/`
- **Tools Used:** Terminal, cURL, Python

## 🎯 Objective

Investigate the Flag Command web application, discover the command required to progress through the challenge, and retrieve the CTF flag.

## 🌐 Website Preview

The challenge takes place in a mysterious forest maze filled with whimsical characters and hidden commands. The objective is to explore the application, identify the correct command, and uncover the flag.

![Flag Command Website](screenshots/flag-command-website.png)

*Figure 1: Flag Command challenge homepage.*

## 🔍 Vulnerability Explanation

The challenge involves investigating how the web application processes commands and communicates with its backend. Inspecting the page source, JavaScript files, and API responses can reveal useful clues about the expected input.

## 💻 Solution — Using Terminal Commands

### Step 1: Set the Target URL

```bash
export TARGET='http://154.57.164.71:35186'
```

### Step 2: Inspect the Website

```bash
curl -i "$TARGET/"
```

### Step 3: Save the Page Source

```bash
curl -s "$TARGET/" -o /tmp/htb_page.html
cat /tmp/htb_page.html
```

### Step 4: Inspect the API Options

```bash
curl -s "$TARGET/api/options" | python3 -m json.tool
```

### Step 5: Test the Command Endpoint

```bash
curl -i -X POST "$TARGET/api/monitor" \
  -H 'Content-Type: application/json' \
  -d '{"command":"test"}'
```

### Step 6: Retrieve the Flag

Submit the correct command identified during the investigation and inspect the response for the flag.

```text
HTB{...}
```

## 📸 Evidence

Add screenshots showing the website, relevant terminal commands, and the successful flag retrieval.

![Terminal Evidence](screenshots/flag-command-terminal.png)

*Figure 2: Terminal investigation and challenge results.*

## ✅ Conclusion

This challenge provided practical experience with web application inspection, HTTP requests, API analysis, and command-based challenge logic using cURL and Python.

**Flag:** `HTB{...}` — replace with the verified flag.
