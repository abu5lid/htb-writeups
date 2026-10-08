# Cap: Hack The Box Write-up

## Overview

## Enumeration
I started with an nmap scan using nmap -A [ip redacted] --script=vuln -oN scan.txt.
The scan revealed port 21 open running vsftpd 3.03 on FTP, 22 runninng SSH, and 80 running gunicorn on HTTP.
## Web Investigation
After visiting the url I crawled the website until I found a security snapshot link which redirected to [ip redacted]/data/1
I fuzzed this url using 'wfuzz -c -z range,0-100 -u http://[ip redacted]/data/FUZZ'
This revealed that there was another file at data/0 which was a .pcap file.
## Packet Capture Analysis
Opening the file in WireShark revealed unencrypted credentials in an FTP layer communication.
## Initial Access
I used the discovered credentials to log into FTP and download the user flag in user.txt.
## Privilege Escalation
I then logged into SSH using the same credentials.
Capability enumeration using 'getcap -r / 2>dev/null' found setuid capabilities under /usr/bin/python3.8
Running '/usr/bin.python3.8 -c 'import os;os.setuid(0); os.system("/bin/bash")' provided me with a root shell.
I retrieved the root flag from root/root.txt
## Findings and Remediation

## Lessons Learned
