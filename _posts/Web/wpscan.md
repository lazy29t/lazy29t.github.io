---
title: Wpscan
author: lazy29t
date: 2023-02-06 22:34:00 +0800
categories: [Tools, Web]
tags: [scan, CVE, exploit, Wordpress]
permalink: /Web/wpscan
pin: true
math: true
mermaid: true

---

## What is Wpscan?

First step you have to verify if the machine is active throught an **ICMP** ping.
```console
$ wpscan -h
```
After that we're going to scan it with nmap tool for discover what ports are exposed on that machine
```console
