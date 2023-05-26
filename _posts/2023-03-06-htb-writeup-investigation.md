---
layout: single
title: Investigation - Hack The Box - WriteUp
excerpt: "Resolución de la máquina de dificultad media 'Investigation' de Hack the Box"
date: 2023-03-13
classes: wide
header:
  teaser: /assets/images/htb-writeup-investigation/investigation-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:  
  - 
---

![investigation-logo](/assets/images/htb-writeup-investigation/investigation-logo.png)

## Portscan

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-24 13:25 CET
Initiating SYN Stealth Scan at 13:25
Scanning 10.10.11.197 [65535 ports]
Discovered open port 80/tcp on 10.10.11.197
Discovered open port 22/tcp on 10.10.11.197
Completed SYN Stealth Scan at 13:25, 13.35s elapsed (65535 total ports)
Nmap scan report for 10.10.11.197
Host is up, received user-set (0.056s latency).
Scanned at 2023-03-24 13:25:17 CET for 14s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.44 seconds
           Raw packets sent: 66133 (2.910MB) | Rcvd: 65536 (2.621MB)
```

## Website

## User shell

## Privilege escalation
