---
layout: blog
type: blog
title: AWS VPS Setup
permalink: blog/aws-vps-setup
# All dates must be YYYY-MM-DD format!
date: 2021-11-16
labels:
  - aws
  - vps
  - bash
  - pentesting vps
---

# What is a VPS?

A **virtual private server** (**VPS**) is a [virtual machine](https://en.wikipedia.org/wiki/Virtual_machine "Virtual machine") sold as a service by an Internet hosting service.

A virtual private server runs its own copy of an operating system(OS), and customers may have superuser-level access to that operating system instance, so they can install almost any software that runs on that OS. For many purposes it is functionally equivalent to a dedicated physical server and, being software-defined, can be created and configured much more easily. A virtual server costs much less than an equivalent physical server. However, as virtual servers share the underlying physical hardware with other VPSes, performance may be lower, depending on the workload of any other executing virtual machines.

#wikipedia

## Why use a VPS for pen testing?

When performing pen tests, labs, bug bounty tests or even CTFs, you would generally be using a virtual machine for most of the tasks however there are a couple of reasons to use a VPS over a local virtual machine.

The biggest benefit would be that using a VPS takes the load off of your local machine, this includes CPU, RAM and even disk usage. Being able to SSH into a remote machine that is much more stable would make things run much smoother in general. Of course, internet connectivity does matter more when performing some tasks, such as remote desktop, but in general, it should not be an issue.

Another benefit would be persistence in your session. Being able to connect from any device to your VPS will save you some heartache when something goes wrong (shit happens).

There are other reasons, such as having a public IP without the need for tunnels, the freedom of configuration and the price.

# Building a VPS

To create a suitable VPS for your tasks, the first step is to determine the hosting service to use and there are quite a few of them, some of which include, Microsoft Azure, Amazon Web Services and Digital Ocean. I would be using the Elastic Compute Cloud service of Amazon Web Services for this.

Upon creating an AWS account you get 12 months free to use some of their services, one of which includes a limited compute EC2 service. Heading to the EC2 Dashboard and selecting `Launch Instance` will take you to the guide for creating your instance.

- Step 1: Choose an Amazon Machine Image (AMI) 
	Select `Ubuntu Server 20.04 LTS` as it is included in the free tier and make sure the x86 option is selected and not the Arm option.
- Step 2: Choose Instance Type
	Use the default selection if you wish to continue with the free tier but if more resources are required for your application then choose a configuration that best suits you.
- Step 3: Configure Instance Details
	Enable `Auto-assign Public IP` and leave the rest in their default state.
- Step 4: Add Storage
	Set the `size` to 30GB as that is the max available for the free tier and deselect the `delete on termination` option.
- Step 5: Add tags
	Leave as default
- Step 6: Configure Security Group
	Set `Type` to `All traffic` and `Source` to `Anywhere` to enable access to the VPS from any system on all ports.
	Note: typically this is not how a VPS should be set but it is needed for this particular use case.
- Step 7: Review
	Click on `Launch` and setup an SSH key pair that will be used to connect to ther server and keep the downloaded file carefully.
	
The state of your EC2 Instance can be checked in the Instances tab of the EC2 Service page. Once the VPS is up and running, you can SSH in or connect using the built-in connect tool.

More could be done, such as setting up a domain name for your Instance, installing tmux for session persistence (will be done below) or setting up a VPN or ssh tunnel.
	
# Installing Tools in VPS

For this, I have created a [GitHub Project](https://github.com/NullCell8822/Tools-Install) that contains a script that I am continuously updating with different tools. To execute the script, run the command below.

```shell
curl -s -H 'Cache-Control: no-cache' https://raw.githubusercontent.com/NullCell8822/Tools-Install/main/install.sh | bash
```

## Tools Installed

### Basics

```txt
git swig whatweb wireshark file nmap fping locate ncdu net-tools git openvpn tmux curl nmap john wfuzz nikto gobuster masscan wireguard nfs-common hydra rename nano vim openssl awscli chromium docker

cmake gcc g++ build-essential lsb-release libpq5 dnsutils lua5.1 alsa-utils python3-pip p7zip-full ca-certificates gnupg-agent software-properties-common net-tools cewl mlocate libcurl4-openssl-dev libssl-dev jq libxml2 libxml2-dev libxslt1-dev ruby-dev build-essential libgmp-dev zlib1g-dev build-essential libssl-dev libffi-dev python3-dev python3-setuptools libldns-dev ruby ruby-dev python3-pip python3-dnspython ruby-full ruby-railties php binutils gdb strace perl libnet-ssleay-perl libauthen-pam-perl libio-pty-perl libncurses5-dev build-essential zlib1g libpq-dev libpcap-dev libsqlite3-dev golang
```

### Go Tools

```txt
subfinder
subjack
httprobe
assetfinder
meg
tojson
unfurl
waybackurls
gf
anew
qsreplace
ffuf
gobuster
amass
getjs
getallurls (gau)
shuffledns
dnsprobe
nuclei
cf-check
crobat
slackcat
```

### Python Pip

```txt
altdns
droopescan
raccoon
```

### Ruby Gems

```txt
wpscan
evil-winrm
```

### GitHub Clones

```txt
massdns
masscan
corsy
dirsearch
arjun
dnsgen
sublert
linkfinder
bass
interlace
sqlmap
graphqlmap
ssrfmap
xsstrike
xsrfprobe
red_hawk
gittools
403bypasser
jwt_tool
sherlock
linenum
linpeas
linux-exploit-suggester
windows-exploit-suggester
seclists
fuzzdb
```

### Others

```txt
findomain
nmap vulners script
aquatone
metasploit framework
```
