---
layout: single
title: Agile - Hack The Box - WriteUp
excerpt: "Resolución de la máquina de dificultad fácil 'Agile' de Hack the Box"
date: 2023-03-13
classes: wide
header:
  teaser: /assets/images/htb-writeup-agile/agile-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:  
  - suid
  - lfi
  - burpsuite

---

![agile-logo](/assets/images/htb-writeup-agile/agile-logo.png)

## Portscan

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-13 17:41 CET
Initiating SYN Stealth Scan at 17:41
Scanning 10.10.11.203 [65535 ports]
Discovered open port 80/tcp on 10.10.11.203
Discovered open port 22/tcp on 10.10.11.203
Completed SYN Stealth Scan at 17:41, 13.69s elapsed (65535 total ports)
Nmap scan report for 10.10.11.203
Host is up, received user-set (0.063s latency).
Scanned at 2023-03-13 17:41:43 CET for 14s
Not shown: 65533 closed tcp ports (reset)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack ttl 63
80/tcp open  http    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.85 seconds
           Raw packets sent: 68650 (3.021MB) | Rcvd: 67486 (2.699MB)
```

## Website

Al entrar a la web nos encontramos con una aplicación encargada de generar y guardar contraseñas para las webs que el usuario decida. Se nos da la opcion de "logear" o crear una cuenta. Creamos una cuenta auxiliar y pasamos a un panel que nos permite añadir, editar y borrar las contraseñas que queramos.

A la vez vamos a usar dirsearch para intentar localizar paths ocultos en la aplicación web.

![dirsearch](/assets/images/htb-writeup-agile/dirsearch.png)

Como se puede ver, el script nos ha devuelto tres paths nuevos, de los cuales uno es al que nos redirige la aplicación al hacer login, otro no muestra nada y el tercero, **download** nos muestra en pantalla un error.

![download-error](/assets/images/htb-writeup-agile/download-error.png)

Este error indica que la función **download** no es capaz de acceder al archivo "/tmp/none". Si analizamos la función veremos que recibe un argumento el cual indica el archivo a descargar, y éste argumento lo envía la aplicación a la hora de exportar un "vault".

Usando **Burpsuite** capturamos dicha petición y le modificamos el argumento para ver si podemos leer fichero del sistema.

![lfi-burp](/assets/images/htb-writeup-agile/lfi-burp.png)
![etc-passwd](/assets/images/htb-writeup-agile/etc-passwd.png)

Hemos conseguido leer el fichero "/etc/passwd" del sistema a través de la petición a la función download.

## User shell

Dado que tenemos una vulnerabilidad de tipo LFI, podemos intentar leer ficheros de configuración o de código que puedan contener credenciales válidas. Dado que hemos usado el error de la función download, vamos a leer el fichero en el cual se encuentra.

![vault-view-request](/assets/images/htb-writeup-agile/vault-view-request.png)
![vault-view-response](/assets/images/htb-writeup-agile/vault-view-response.png)

Este fichero contiene toda la funcionalidad relacionada con el uso del "vault", y podemos intentar ver si tenemos acceso a las contraseñas de otros usuarios. Usando la funcion de "/vault/row/<id>" vamos probando ids y en el número 8 encontramos la contraseña del usuario "corum" para el servidor de agile.

![corum-password](/assets/images/htb-writeup-agile/corum-password.png)

Con estas credenciales probamos a logear al servidor a través de ssh y conseguimos acceso a la máquina.

![user-shell](/assets/images/htb-writeup-agile/user-shell.png)

Leemos la flag "user.txt" y la introducimos en HTB.

## Privilege escalation

Para la escadala de privilegios, el usuario corum no tiene privilegios de root para ejecutar ningún comando pero sí que vemos que python3.10 tiene el bit de suid activado.

![suid](/assets/images/htb-writeup-agile/suid.png)

Sabiendo esto y usando la web de [GTFOBins](https://gtfobins.github.io/gtfobins/python/#suid) conseguimos elevar privilegios y obtener una shell con permisos de root.

![root-shell](/assets/images/htb-writeup-agile/root-shell.png)