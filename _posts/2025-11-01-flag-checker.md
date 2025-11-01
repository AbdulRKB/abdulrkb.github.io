---
title: "Huntress CTF 2025 Writeup: Flag Checker"
date: 2025-11-01 16:36:48 +0100
categories: ["Huntress CTF 2025"]
tags: ["Web", "CTF", "Huntress", "Timing Attack"]
---

> We've decided to make this challenge really straight forward. All you have to do is find out the flag!

> Juuuust make sure not to trip any of the security controls implemented to stop brute force attacks...
---
![flag-checker-1.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/flag-checker-1.png)

I used the alert "Brute forcing won't work" as a hint that it was a brute forcing challenge, but what kind of brute force?

After looking at the website more and submitting data, I realised that inputting correct substring (from index 0) caused the website to have a delay which matched the length of the correct substring, e.g. "flag" led to a response time of 0.4s, and "flag{" led to a response time of 0.5s.

![flag-checker-2.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/flag-checker-2.png)

This showed that the web was vulnerable to character by character time-analysis brute forcing. So I created a python script to automate the process of testing every character from `a-f0-9`to see if a change in response time was detected, and upon detection, I would attach that character to the prefix string manually and rerun the string to test characters for next position.  

After submitting about 10 inputs, I was met with the following error:

![flag-checker-3.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/flag-checker-3.png)

I tried to bypass this by adding `X-Forwarded-For`, however It didn't work. So I decided to reset the machine every time the server blocked my IP. 

```python
import requests


IP = "10.1.51.122"
URL = f"http://"+IP+"/submit?flag=flag{77ba0346d9565e77344b9fe40ecf1369"
DATA = "abcdef0123456789"
last_char = '2'
DATA = last_char+DATA.split(last_char)[1]
try:

    for d in DATA:
        resp = requests.get(URL+d)
        resp_data = resp.content.decode()
        print(f"[+] Trying {d}, time: {resp.headers['X-Response-Time']}")
        if "Stop Hacking" in resp_data:
            print(f"[-] Rate Limit, last character: {d} ")
            break
except KeyboardInterrupt:
    print("[x] Exiting!")
except KeyError:
    print("[-] Server Still Starting...")
```
