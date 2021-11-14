---
layout: writeup
type: writeup
image: images/writeups/thm/h4cked/logo.png
title: H4cked - TryHackMe
permalink: writeups/h4cked
# All dates must be YYYY-MM-DD format!
date: 2021-04-20
labels:
  - TryHackMe
  - Easy
  - Wireshark
  - Forensics
summary: find out what happened by analysing a .pcap file and hack your way back into the machine.
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = H4cked
TARGET = VARIABLE
OS = LINUX
DIFFICULTY = EASY
```

## Forensics

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/1.png">

Analyzing the pcap file to find the attacker exploited the ftp by bruteforcing user jenny's password and uploaded a php reverse shell

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/2.png">

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/3.png">

Using the same ftp credentials to switch to the user jenny and then found the user to have full privileges and got root. Then setting up a rootkit to maintain access

## Hack Back

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/4.png">

Rrustscan to quickly check for the open ports

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/5.png">

Using nmap to do a full scan

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/6.png">

Using hydra on ftp for the user jenny

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/7.png">

Loging in i see that the web sever is being hosted in that directory
/var/www/html

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/8.png">

I then uploaded a php reverse shell, started a netcat listener and got a reverse shell after going to the page

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/9.png">

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/10.png">

I used the same credentials i got for the user jenny previously to try to log into their account
I then used the privileges to escalate to root

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/h4cked/11.png">