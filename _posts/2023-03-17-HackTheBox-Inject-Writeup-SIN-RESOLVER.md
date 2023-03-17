---
title: HackTheBox Precious Writeup
published: false
---

La máquina Precious, es una máquina de HackTheBox en la que mediante la explotación de esta conseguiremos la flag del usuario y la de root.


# [](#header-1)Enumeración

La enumeración se trata de una de las fases principales, en las que recolectamos la máxima información posible de la máquina víctima.

Usaremos nmap para el siguiente escaneo:

```bash
nmap -p- --open --min-rate 5000 -vvv -n $IP-VICTIMA -oG allPorts
```

*  Escaneo todos los puertos abiertos: -p- --open

*  Mediante --min-rate 5000 indicamos que queremos emitir paquetes no más lentos que 5000 paquetes por segundo.

*  Triple verbose (-vvv) para que vaya reportando por consola los puertos abiertos y no tener que esperar al final del escaneo.

*  Para que el escaneo no aplique resolución DNS usamos -n

*  En mi caso exporto los puertos abiertos en formato grepeable (-oG) al archivo allPorts

```bash
❯ nmap -p- --open --min-rate 5000 -T5 -n -vvv -Pn 10.10.11.204 -oN allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-17 13:12 EDT
Initiating Connect Scan at 13:12
Scanning 10.10.11.204 [65535 ports]
Discovered open port 8080/tcp on 10.10.11.204
Discovered open port 22/tcp on 10.10.11.204
Completed Connect Scan at 13:13, 37.75s elapsed (65535 total ports)
Nmap scan report for 10.10.11.204
Host is up, received user-set (0.059s latency).
Scanned at 2023-03-17 13:12:24 EDT for 37s
Not shown: 58529 filtered tcp ports (no-response), 7004 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack
8080/tcp open  http-proxy syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 37.79 seconds
```

Nos reporta el puerto 22 y el 8080. Analizaremos la versión y servicio de estos mediante:

```bash
❯ nmap -p22,8080 -sCV 10.10.11.204
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-17 13:14 EDT
Nmap scan report for inject.htb (10.10.11.204)
Host is up (0.065s latency).

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 caf10c515a596277f0a80c5c7c8ddaf8 (RSA)
|   256 d51c81c97b076b1cc1b429254b52219f (ECDSA)
|_  256 db1d8ceb9472b0d3ed44b96c93a7f91d (ED25519)
8080/tcp open  nagios-nsca Nagios NSCA
|_http-title: Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.21 seconds
```

# [](#header-1)Acceso al sistema

Puesto que está el puerto 8080 abierto, nos dirigimos a la página (http://inject.htb:8080) y observamos una home page de Zodd Cloud la que en la esquina superior derecha podemos subir archivos desde nuestro equipo. Parece que or ahí pueden ir los tiros. Usaré gobuster para localizar algún directorio oculto, por si acaso.


```bash
❯ gobuster dir -u http://10.10.11.204:8080 -w /usr/share/wordlists/dirb/common.txt -t 25
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.11.204:8080
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2023/03/17 13:22:25 Starting gobuster in directory enumeration mode
===============================================================
/blogs                (Status: 200) [Size: 5371]
/error                (Status: 500) [Size: 106] 
/environment          (Status: 500) [Size: 712] 
/register             (Status: 200) [Size: 5654]
/upload               (Status: 200) [Size: 1857]
                                                
===============================================================
2023/03/17 13:24:39 Finished
===============================================================
```

La página de registro no funciona y lo único interesante es /upload, que nos remite a la subida de archivo que habíamos visto.

Intento subir un archivo cualquiera y nos reporta: Only Image Files are Accepted.

Intento subir un archivo .jpg pero que no es una imagen y no nos deja. Tampoco acepta .php.jpg pero al subir una imagen de verdad con extensión .jpg nos reporta un link en el que podemos ver la propia imagen.



Muchas gracias por leer este writeup, espero que te haya gustado!
