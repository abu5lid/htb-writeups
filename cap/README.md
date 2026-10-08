# Cap: Hack The Box Write-up

## Overview
I assessed the Cap machine in an authorized Hack The Box lab. A packet capture exposed FTP credentials, which gave me access to FTP and SSH. I then found a file capability on Python that let me obtain a root shell.

## Enumeration
I started with an Nmap scan:

```bash
nmap -A [ip redacted] --script=vuln -oN scan.txt
```

The scan revealed port 21 running FTP (vsftpd), port 22 running SSH, and port 80 running HTTP (Gunicorn).

## Web Investigation
After visiting the site, I explored it until I found a Security Snapshot link, which redirected to `[ip redacted]/data/1`.

I fuzzed the numeric part of this URL:

```bash
wfuzz -c -z range,0-100 -u http://[ip redacted]/data/FUZZ
```

This revealed that there was a page at `/data/0`. Clicking the download button on `/data/1` led to `/download/1` and downloaded a `.pcap` file, so I navigated to `/download/0` and retrieved the capture there.

## Packet Capture Analysis
Opening the file in Wireshark revealed credentials in unencrypted FTP traffic.

## Initial Access
I used the discovered credentials to log into FTP and download the user flag in `user.txt`.

## Privilege Escalation
I then logged into SSH using the same credentials. I enumerated file capabilities with:

```bash
getcap -r / 2>/dev/null
```

This showed that `/usr/bin/python3.8` had the `CAP_SETUID` file capability. That capability allows a process to change its user ID ([Linux setuid manual](https://www.man7.org/linux/man-pages/man2/setuid.2.html)). Running the following command gave me a root shell:

```bash
/usr/bin/python3.8 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

I retrieved the root flag from `/root/root.txt`.

## Findings and Remediation

- **Exposed packet captures:** The numbered capture endpoints exposed a PCAP containing credentials. Restrict each capture and download to authorized users with server-side access checks; changing an ID must not bypass authorization.
- 
- **Plaintext credentials and credential reuse:** FTP exposed credentials in clear text, and the same credentials worked for SSH. Replace plaintext FTP with an encrypted file-transfer service.
- 
- **Excessive Python capability:** Python had `CAP_SETUID`, allowing privilege escalation. Remove that capability unless there is a specific need for it, and review other file capabilities.

## Lessons Learned
Small weaknesses can form an attack chain: access to a packet capture exposed credentials, those credentials worked on more than one service, and an excessive file capability allowed escalation to root. 
