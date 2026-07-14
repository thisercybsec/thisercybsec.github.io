---
title: "HTB: Forge — SSRF to Root"
date: 2026-07-13 10:00:00 -0400
categories: [CTF, HackTheBox]
tags: [ssrf, web, linux, privesc]
img_path: /assets/img/posts/htb-forge
---

> Replace this file's name, front matter, and content for every new writeup.
> Keep the section headers consistent — that consistency is what makes your
> growth over time easy for someone to skim and follow.
{: .prompt-tip }

## Overview

2-3 sentence summary: box difficulty, OS, and the core vulnerability chain.
E.g., "Forge is a Linux box that chains an SSRF vulnerability in a file-upload
feature into local file read, followed by a sudo misconfiguration for root."

## Recon

Initial nmap scan to identify open ports and services.

```bash
nmap -sC -sV -oN nmap-initial.txt 10.10.10.x
```

![nmap scan results](01-nmap-scan.png)
_Initial port scan showing HTTP and SSH open_

## Enumeration

What you found poking at each service — web directories, subdomains, etc.

```bash
gobuster dir -u http://10.10.10.x -w /path/to/wordlist.txt
```

![Web enumeration](02-web-enum.png)
_Directory brute force revealing an upload endpoint_

## Exploitation

Walk through the exploit step by step. Show the command, the result, and
*why* it worked — this is the part that shows your thought process.

```bash
curl -X POST http://10.10.10.x/upload -d "url=http://169.254.169.254/latest/meta-data/"
```

![SSRF proof of concept](03-ssrf-poc.png)
_Confirming SSRF against the internal metadata endpoint_

## Gaining a Shell

```bash
nc -lvnp 4444
```

![Reverse shell established](04-shell.png)
_Landed a shell as www-data_

## Privilege Escalation

```bash
sudo -l
```

![Sudo misconfiguration](05-privesc.png)
_Identifying the sudo rule that allows privilege escalation_

## Root

![Root flag](06-root-flag.png)
_Final root shell and flag_

## Lessons Learned

- What the vulnerability class was and why it matters
- What you'd try differently next time
- Tools or techniques you want to get more comfortable with

## References

- [Link to relevant CVE, blog post, or technique writeup]
