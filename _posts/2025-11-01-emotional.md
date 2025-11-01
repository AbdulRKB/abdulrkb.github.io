---
title: "Huntress CTF 2025 — Emotional Write-up"
date: 2025-11-01 16:36:48 +0100
categories: ["Huntress CTF 2025"]
tags: ["Web", "CTF", "Huntress", "SSTI"]
image: https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/emotional-1.png
---

> Don't be shy, show your emotions! Get emotional if you have to! Uncover the flag.
---

As per the source code given, the website was rendering using ejs. I realised the website was vulnerable to SSTI attacks after it kept resetting upon submitting an SSTI polyglot to `/setEmoji`. 


To make sure, I submitted with the payload `<%= 7*7 %>`, and the server replied with 49! I then researched, and found a [GitHub repo](https://github.com/NketiahGodfred/EJS-ssti-exploit/blob/main/exploit.sh) with a payload that allows executing shell code on server. And then I submitted:

```js
<%= global.process.mainModule.constructor._load("child_process").execSync("ls"); %>
```

![emotional-2.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/emotional-2.png)

I then just catt'd out the flag.

![emotional-3.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/emotional-3.png)
