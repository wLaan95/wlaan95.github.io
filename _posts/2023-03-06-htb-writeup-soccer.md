---
layout: single
title: Soccer - Hack The Box - WriteUp
excerpt: "Resolución de la máquina de dificultad fácil 'Soccer' de Hack the Box"
date: 2023-03-06
classes: wide
header:
  teaser: /assets/images/htb-writeup-soccer/soccer-logo.png
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

![](/assets/images/htb-soccer-writeup/soccer-logo.png)

## Portscan

```
Nmap scan report for soccer.htb (10.10.11.194)                                                                                                                                                                    
Host is up (0.063s latency).                                                                                                                                                                                      
Not shown: 65532 closed tcp ports (reset)                                                                                                                                                                         
PORT     STATE SERVICE         VERSION                                                                                                                                                                            
22/tcp   open  ssh             OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)                                                                                                                       
| ssh-hostkey:                                                                                                                                                                                                    
|   3072 ad:0d:84:a3:fd:cc:98:a4:78:fe:f9:49:15:da:e1:6d (RSA)                                                                                                                                                    
|   256 df:d6:a3:9f:68:26:9d:fc:7c:6a:0c:29:e9:61:f0:0c (ECDSA)                                                                                                                                                   
|_  256 57:97:56:5d:ef:79:3c:2f:cb:db:35:ff:f1:7c:61:5c (ED25519)                                                                                                                                                 
80/tcp   open  http            nginx 1.18.0 (Ubuntu)                                                                                                                                                              
|_http-title: Soccer - Index                                                                                                                                                                                      
|_http-server-header: nginx/1.18.0 (Ubuntu)                                                                                                                                                                       
9091/tcp open  xmltec-xmlmail?                                                                                                                                                                                    
|......
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

## Website

La página principal de la web no tiene información relevante y no hay ningún enlace que nos redirija a otra página. Vamos a usar un buscador de directorios como puede ser **dirsearch** para ver si encontramos algo:

![](/assets/images/htb-soccer-writeup/dirsearch-command.png)

El programa ha encontrado un directorio llamado 'tiny' que contiene una página de login.

## Tiny Exploitation

La apliación web Tiny File Manager es opensource y el código fuente se puede encontrar en GitHub. Gracias a ellos sabemos que por defecto la aplicación viene con dos usuarios, **admin** y **user**. Usando las credenciales encontradas en el código fuente conseguimos acceder al panel del control de la aplicación.

## User shell

Ya tenemos acceso a la aplicación de gestión de ficheros cuya versión es 2.4.3. Si buscamos exploits para TinyFileManager encontramos que esta versión es vulnerable a un 'path transversal' a través de su función de subir ficheros. Dado que tenemos la cuenta de administrador, podemos usar esta [webshell](https://pentestmonkey.net/tools/web-shells/php-reverse-shell) para subir el fichero, abrirlo y obtener una shell reversa:

![](/assets/images/htb-soccer-writeup/user-shell.png)

Tras usar linpeas para encontrar posibles formas de escalar privilegios, encontramos que existe un subodminio dentro de **soccer.htb** llamado **oc-player.soccer.htb**. Tras añadir este nuevo dominio a nuestro fichero /etc/hosts observamos que ahora la página web cuenta con nuevas opciones, como match, login y signup.

Procedemos a crear un usuario y hacer login, y nos encontramos con esta nueva página:

![](/assets/images/htb-soccer-writeup/ticket.png)

Esta página comprueba un número de ticket y nos indica si es válido o no. Revisando el código fuente observamos que a través de Websockets realiza consultas a una base de datos. Siguiendo los pasos de este [artículo](https://rayhan0x01.github.io/ctf/2021/04/02/blind-sqli-over-websocket-automation.html) para realizar **Blind SQL** a WebSockets a través de nodejs, podemos obtener el nombre de la base de datos, sus tablas y la información alojada en ellas. Gracias a esto encontramos el usuario **player** con el que podemos conectar a través de SSH a la máquina.

![](/assets/images/htb-soccer-writeup/db-dump.png)

Navegamos al escritorio y encontramos la primera flag.

## Privilege escalation

Buscamos archivos que tengan permisos de SUID con el siguiente comando:
```
find / -perm /4000 2>/dev/null
```
Observamos que nos sale 'doas' y éste nos permite la ejecución de 'dstat' con permisos de root

![](/assets/images/htb-soccer-writeup/doas.png)

Dstat es vulnerable y nos permite la elevación de privilegios si pudiesemos ejecutarlo con permisos de root, pero nosotros tenemos que ejecutar dstat a través de doas para tener dicho privilegio.

Lo primero es encontrar los directorios sobre los que trabaja dstat

```
find / -type d -name dstat 2>/dev/null

/usr/share/doc/dstat
/usr/share/dstat
/usr/local/share/dstat

```

Usaremos el fichero '/usr/local/share/dstat'. 

Lo segundo que hay que hacer es crear un script en la ruta anterior que se ejecutará con permisos de root, por ejemplo añadir el bit de SUID a una shell:

```
import os

os.system('chmod +s /usr/bin/bash')
```

Ahora ejecutamos el script:

![](/assets/images/htb-soccer-writeup/dstat-priv-esc.png)

Si ahora ejecutamos una bash, la shel obtenida tendrá privilegios de root y así conseguimos la segunda flag.
