---
layout: project
type: project
image: images/writeups/thm/team/logo.png
title: Team - TryHackMe
permalink: writeups/team
# All dates must be YYYY-MM-DD format!
date: 2021-03-18
labels:
  - TryHackMe
  - boot2root
  - Easy
  - LFI
  - Cronjobs
summary: A beginner friendly boot2root machine.
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

## Enumeration

Starting with rustscan to find all open ports quickly

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/1.png">

Passing the ports to nmap for a full scan

```
┌──(root💀Toyin)-[~/infosec/thm/team]
└─# nmap -A -T4 -p 21,22,80 -oN nmap/scan 10.10.100.77
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-11 02:53 WAT
Nmap scan report for 10.10.100.77
Host is up (0.15s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   2048 79:5f:11:6a:85:c2:08:24:30:6c:d4:88:74:1b:79:4d (RSA)
|   256 af:7e:3f:7e:b4:86:58:83:f1:f6:a2:54:a6:9b:ba:ad (ECDSA)
|_  256 26:25:b0:7b:dc:3f:b2:94:37:12:5d:cd:06:98:c7:9f (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works! If you see this add 'te...
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Aggressive OS guesses: Linux 3.10 - 3.13 (92%), Crestron XPanel control system (90%), ASUS RT-N56U WAP (Linux 3.4) (87%), Linux 3.1 (87%), Linux 3.16 (87%), Linux 3.2 (87%), HP P2000 G3 NAS device (87%), AXIS 210A or 211 Network Camera (Linux 2.6.17) (87%), Linux 5.4 (86%), Linux 2.6.32 (86%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 80/tcp)
HOP RTT       ADDRESS
1   142.29 ms 10.8.0.1
2   142.52 ms 10.10.100.77

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 22.50 seconds
```

Checking the web server on port 80 it lead me to the default apache2 page. I then tried added the hostname "team.thm" to my /etc/hosts file and used curl to check if the pages were different
For good measure i generally curl /robots.txt just incase there is something intersting there

Upon inspecting the default apache page further i find that it does say to include the hostname "team.thm"

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/2.png">

Going to the page team.thm I found a static page and gobuster and nikto scans led nowhere so the next step was to find subdomains.
My preferred tool for scanning is gobuster vhost. I got a hit on "dev.team.thm"

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/3.png">

## Exploitation

I went to the page and got a link that went to "http://dev.team.thm/script.php?page=teamshare.php"
The page parameter seemed to be a likely avenue for an LFI and tested by getting /etc/passwd and thereby seeing the users on the machine

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/4.png">

I tried and failed all LFI to RCE methods and started checking common LFI files that could leak information, /etc/ssh/sshd_config happened to leak dale's ssh key (not weird at all)
I'm putting it because it's probably faster to root this machine than type this out

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/5.png">

After putting the ssh key in a file and changing the permission I was able to log in as dale

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/6.png">

## Privilege Escalation

Running "sudo -l" and "id" it seems that dale is a member of the sudo group, too bad his password is unknown. He is able to run "/home/gyles/admin_checks as the user gyles.
I found that the script can run commands because it executes whatever you pass to $error variable

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/7.png">
<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/8.png">

I use that command execution as gyles to get a reverse shell as that user

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/9.png">

Gyles .bash_history was not cleaned and i found a history of some scripts running as cronjobs. Downloading pspy from my machine i found the scripts that were running

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/10.png">
<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/11.png">
<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/12.png">

Gyles is in the admin group so I can edit /usr/local/bin/main_backup.sh to give a reverse shell as root

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/13.png">
<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/team/14.png">
