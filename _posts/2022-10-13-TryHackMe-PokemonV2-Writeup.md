---
title: TryHackMe Gotta Catch’em All! Writeup
published: true
---

La máquina Gotta Catch’em All!, es una máquina de TryHackMe en la que mediante la explotación de un servidor web conseguimos responder a las preguntas que nos plantean:

* Encontrar el Pokemon tipo Tierra<br>
* Encontrar el Pokemon tipo Agua<br>
* Encontrar el Pokemon tipo Fuego<br>
* ¿Cuál es el Pokemon favorito de root?<br>

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
# Nmap 7.92 scan initiated Sat Oct  8 13:51:10 2022 as: nmap -p- --open --min-rate 5000 -vvv
       │  -n -oG allPorts - Pn 10.10.50.121
   2   │ # Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
   3   │ Host: 139.162.17.173 () Status: Up
   4   │ Host: 139.162.17.173 () Ports: 22/open/tcp//ssh///, 25/open/tcp//smtp///, 80/open/tcp//http/
       │ //, 443/open/tcp//https///, 587/open/tcp//submission///, 993/open/tcp//imaps///, 995/open/tc
       │ p//pop3s/// Ignored State: filtered (65528)
   5   │ Host: 10.10.50.121 ()   Status: Up
   6   │ Host: 10.10.50.121 ()   Ports: 22/open/tcp//ssh///, 80/open/tcp//http///
   7   │ # Nmap done at Sat Oct  8 13:53:17 2022 -- 2 IP addresses (2 hosts up) scanned in 127.36 sec
       │ onds
```

Nos reporta una serie de puertos abiertos. Debido a que el puerto 80 está abierto, introducimos la $IP-VICTIMA en un buscador y nos reporta el servicio Apache. Sin mucha más información checkeamos la consola para ver si encontramos algo. 

```html
<div class="content_section_text">
          <p>
                Please use the <tt>ubuntu-bug</tt> tool to report bugs in the
                Apache2 package with Ubuntu. However, check <a href="https://bugs.launchpad.net/ubuntu/+source/apache2">existing
                bug reports</a> before reporting a new bug.
          </p>
        </div>
        <REDACTED1>:<REDACTED2>
         <!--(Check console for extra surprise!)-->
      </REDACTED2></REDACTED1>
```

Parece ser que hay un usuario y contraseña entre los dos puntos antes del comentario. Intentaremos entonces usar SSH para conectarnos.

# []($header-1)Acceso al sistema

```bash
❯ ssh ???????@10.10.224.44
The authenticity of host '10.10.224.44 (10.10.224.44)' can't be established.
ED25519 key fingerprint is SHA256:pLr5hKfcRZWD4ZBMz/8vFWnJ2xslYHSX94C4KXwOLVg.
This host key is known by the following other names/addresses:
    ~/.ssh/known_hosts:6: [hashed name]
    ~/.ssh/known_hosts:8: [hashed name]
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.224.44' (ED25519) to the list of known hosts.
pokemon@10.10.224.44's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

84 packages can be updated.
0 updates are security updates.


The programs included with the Ubuntu system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Ubuntu comes with ABSOLUTELY NO WARRANTY, to the extent permitted by
applicable law.

pokemon@root:~$ 
```
Hemos conseguido así una consola interactiva. Desde este punto e investigando un poco hayamos el primer pokemon:

```bash
pokemon@root:~$ ls -R
.:
Desktop  Documents  Downloads  examples.desktop  Music  Pictures  Public  Templates  Videos

./Desktop:
P0kEmOn.zip

./Documents:

./Downloads:

./Music:

./Pictures:

./Public:

./Templates:

./Videos:
Gotta

./Videos/Gotta:
Catch

./Videos/Gotta/Catch:
Them

./Videos/Gotta/Catch/Them:
ALL!

./Videos/Gotta/Catch/Them/ALL!:
Could_this_be_what_Im_looking_for?.cplusplus
```

Nos metemos en el directorio Desktop y descomprimimos el archivo:

```bash
pokemon@root:~/Desktop$ unzip P0kEmOn.zip 
Archive:  P0kEmOn.zip
   creating: P0kEmOn/
  inflating: P0kEmOn/grass-type.txt  
```

Abrimos el archivo y nos reporta la solución en hexadecimal por lo que con el comnado xxd terminamos de resolverlo:

```bash
pokemon@root:~/Desktop/P0kEmOn$ cat grass-type.txt 
50 6f 4b 65 4d 6f 4e 7b 42 75 6c 62 61 73 61 75 72 7d
pokemon@root:~/Desktop/P0kEmOn$ cat grass-type.txt | xxd -r -p
???????{????????}
```

Cuando hemos hecho ls -R, nos hemos dado cuenta de que existe un directorio (./Videos/Gotta/Catch/Them/All!:) con un archivo que puede que contenga información importante. Nos dirigimos a el y lo abrimos:

```bash
pokemon@root:~$ cd Videos/Gotta/Catch/Them/ALL\!/
pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ ls
Could_this_be_what_Im_looking_for?.cplusplus
pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ cat Could_this_be_what_Im_looking_for\?.cplusplus 
# include <iostream>

int main() {
        std::cout << "??? : ????????"
        return 0;
}
```

# []($header-1)Escalada de privilegios

Parece ser que tenemos otro usuario y contraseña por lo que inetntamos conectarnos a el por SHH con:

```bash
pokemon@root:~/Videos/Gotta/Catch/Them/ALL!$ ssh ash@10.10.224.44
The authenticity of host '10.10.224.44 (10.10.224.44)' can't be established.
ECDSA key fingerprint is SHA256:mXXTCQORSu35gV+cSi+nCjY/W0oabQFNjxuXUDrsUHI.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '10.10.224.44' (ECDSA) to the list of known hosts.
ash@10.10.224.44's password: 
Welcome to Ubuntu 16.04.6 LTS (GNU/Linux 4.15.0-112-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/advantage

84 packages can be updated.
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

Could not chdir to home directory /home/ash: Permission denied
$
```

Nos reporta una consola. En este punto podemos hacer sudo -l para ver los permisos que tiene el usuario: 

```bash
$ sudo -l
[sudo] password for ash: 
Matching Defaults entries for ash on root:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ash may run the following commands on root:
    (ALL : ALL) ALL
```

Vemos que puede usar todos los comandos como sudo. Nos dirigimos entonces a /home para ver su contenido. En el aparece el usuario ash y el archivo roots-pokemon.txt

```bash
$ cd /home
$ ls
ash  pokemon  roots-pokemon.txt
```

Como somos el usuario ash y este puede usar todos los comandos como root intentamos abrir el archivo roots-pokemon.txt

```bash
$ sudo cat roots-pokemon.txt
???????!
```

Obtenemos el pokemon favorito de root y solo nos quedaría el tipo agua y fuego. Para ello, podemos buscar archivos que se llamen ???water???.txt y ???fire???.txt

Lo hacemos de esta forma:

```bash
$ find / -type f -name *water*.txt 2>/dev/null
/var/www/html/water-type.txt
$ sudo cat /var/www/html/water-type.txt
Ecgudfxq_EcGmP{Ecgudfxq}
```

Parece ser que se ha aplicado una rotación en el texto. Para solucionar esto, podemos dirigirnos a <a href="https://rot13.com">rot13</a> y probar con distintas rotaciones. Damos así con la respuesta al tipo agua. Hacemos lo mismo con el tipo fuego:

```bash
$ find / -type f -name *fire*.txt 2>/dev/null
/etc/why_am_i_here?/fire-type.txt
$ sudo cat /etc/why_am_i_here?/fire-type.txt
UDBrM20wbntDaGFybWFuZGVyfQ==
```

Ocurre lo mismo que la flag anterior. No nos dan el texto claro. Observando un poco el texto, podemos deducir que está en base64. Podemos descifrarlo entonces desde la consola con:

```bash
$ sudo cat /etc/why_am_i_here?/fire-type.txt | base64 -d
???????{??????????}
```

Obtenemos así todas las flags!

Muchas gracias por leer este writeup, espero que te haya gustado!
