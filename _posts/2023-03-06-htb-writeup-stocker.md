---
layout: single
title: Stocker - Hack The Box - WriteUp
excerpt: "Resolución de la máquina de dificultad fácil 'Stocker' de Hack the Box"
date: 2023-03-09
classes: wide
header:
  teaser: /assets/images/htb-writeup-stocker/stocker-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:  
  - bootstrap

---

![](/assets/images/htb-writeup-stocker/stocker-logo.png)

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

La web no presenta información útil (más allá del nombre de un empleado que podría ser útil más adelante si necesitamos algún tipo de nombre de usuario, "Angoose Garden"), y usando dirsearch no encontramos ningún path interesante. Probamos con gobuster para buscar posibles subdominios y obtenemos el siguiente resultado:

```
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://stocker.htb
[+] Method:       GET
[+] Threads:      50
[+] Wordlist:     /usr/share/wordlists/seclists/Discovery/DNS/subdomains-top1million-5000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2023/03/10 18:16:39 Starting gobuster in VHOST enumeration mode
===============================================================
Found: dev.stocker.htb (Status: 302) [Size: 28]
                                               
===============================================================
2023/03/10 18:16:45 Finished
===============================================================
```
Existe un subdominio llamado **dev** que contiene un panel de login. Después de varios intentos, y de búsqueda, vemos que no se puede realizar un SQLi normal. Para este caso tenemos que usar una inyección [noSQL](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection). 

Capturamos la petición realizada en el login con BurpSuite, modificamos los valores de username y password, y le damos un formato JSON

![](/assets/images/htb-writeup-stocker/nosqi-json.png)

Con esto conseguimos acceder a la página de "stock".

## User shell

## Privilege escalation

