---
layout: single
title: Stocker - Hack The Box - WriteUp
excerpt: "Resolución de la máquina de dificultad fácil 'Stocker' de Hack the Box"
date: 2023-03-06
classes: wide
header:
  teaser: /assets/images/htb-writeup-stocker/stocker-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:  
  - websocket
  - blind sqli
  - doas
  - dstat
  - path transversal
---

![](/assets/images/htb-soccer-writeup/stocker-logo.png)

## Portscan

```
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-09 19:47 CET
Initiating SYN Stealth Scan at 19:47
Scanning 10.10.11.196 [65535 ports]
Discovered open port 80/tcp on 10.10.11.196
Increasing send delay for 10.10.11.196 from 0 to 5 due to 169 out of 562 dropped probes since last increase.
Discovered open port 22/tcp on 10.10.11.196
Completed SYN Stealth Scan at 19:47, 13.67s elapsed (65535 total ports)
Nmap scan report for 10.10.11.196
Host is up, received user-set (0.054s latency).
Scanned at 2023-03-09 19:47:29 CET for 14s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.77 seconds
           Raw packets sent: 67686 (2.978MB) | Rcvd: 66512 (2.814MB)
```

## Website

## User shell

## Privilege escalation

