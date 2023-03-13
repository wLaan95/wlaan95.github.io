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
  - nginx
  - ssrf
  - sudo
  - nodejs

---

![stocker-logo](/assets/images/htb-writeup-stocker/stocker-logo.png)

## Portscan

```nmap
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

La web no presenta información útil, y usando dirsearch no encontramos ningún path interesante. Probamos con gobuster para buscar posibles subdominios y obtenemos el siguiente resultado:

```gobuster
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

Existe un subdominio llamado **dev** que contiene un panel de login. Después de varios intentos, y de búsqueda, vemos que no se puede realizar un SQLi normal. Para este caso tenemos que usar una inyección [noSQL](https://github.com/swisskyrepo/PayloadsAllTheThings/tree/master/NoSQL%20Injection)

Capturamos la petición realizada en el login con BurpSuite, modificamos los valores de username y password, y le damos un formato JSON

![nosqli-json](/assets/images/htb-writeup-stocker/nosqli-json.png)

Con esto conseguimos acceder a la página de "stock".

Probamos a añadir un item a la cesta y a realizar la compra, la web nos devuelve un link que nos muestra una factura del pedido realizado. Parece tratarse de un pdf generadod a través de un json que contenía la información del producto de la cesta.

![json-cesta](/assets/images/htb-writeup-stocker/json-cesta.png)

Revisando los metadatos del pdf vemos que se ha generado con la herramiento **Skia/pdf**, herramienta que se usa en la opción de Google chrome "Save as PDF". Revisando posibles exploit observamos que se puede reaizar un **ssrf** llegando a poder leer archivos del servidor.

![ssrf-request](/assets/images/htb-writeup-stocker/ssrf-request.png)

![ssrf-response](/assets/images/htb-writeup-stocker/ssrf-response.png)

## User shell

Ahora que podemos leer ficheros (y sabemos la existencia de un usuario llamado "angoose"), vamos a intentar encontrar información relevante en ficheros de configuración. Usando "Wappalyzer" sabemos que el servidor usa Nginx que tiene sus ficheros de configuración en tres posibles rutas. Encontramos el archivo "nginx.conf" en "/etc/nginx".

Vemos que **dev.stocker.htb** está alojado en la ruta "/var/www/dev" y además usa NodeJS, por lo que index.js puede contener información importante.

![index-js](/assets/images/htb-writeup-stocker/index-js.png)

Obtenemos una password pero el usuario "dev" sabemos que no existe en el servidor, así que la probamos con el otro usuario a través de ssh y conseguimos acceso a una consola interactiva.

![user-shell](/assets/images/htb-writeup-stocker/user-shell.png)

Procedemos a leer la flag **user.txt**.

## Privilege escalation

Nuestro usuario tiene permisos para ejecutar "/usr/bin/node" pero solo a los siguientes ficheros "/usr/local/scripts/\*js". Como vemos, estamos limitados a la carpeta scripts en dicha ruta, pero el carácter comodín '*' nos da rienda suelta para utilizar **path traversal** y ejecutar el script que nosotros queramos (siempre y cuando sea js).

Utilizando la web de [GTFOBins](https://gtfobins.github.io/gtfobins/node/) creamos un script en la carpeta /tmp que contenga el código a ejecutar con "node".

![node-script](/assets/images/htb-writeup-stocker/node-script.png)

Ahora ejecutamos este script con permisos de root y conseguimos elevar privilegios.

![exploit-root](/assets/images/htb-writeup-stocker/exploit-root.png)
