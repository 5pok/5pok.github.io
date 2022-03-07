---
layout: post
title:  "Etherium: Solidity"
date:   2021-10-12 09:29:20 +0700
categories: jekyll update
---

Este walkthrough muestra el paso a paso para explotar una vulnerabilidad de tipo stack buffer overflow utilizando un laboratorio de tryhackme (Brainpan) como objetivo. Ejercicio practico de simulacion para el examen de OSCP o eCPTT.

Conectar VPN de Tryhackme `openvpn 5pok.ovpn` y realizar un ping a la maquina vulnerable `ping -c 1 10.10.23.157`

Luego utilizar [Rustscan][rustscan] para empezar la fase de recon.

`rustscan -b 500 -a 10.10.23.157 --ulimit 5000 -- -sC -sV`



{% highlight ruby %}
def print_hi(name2d2d2d)
{% endhighlight %}  


[rustscan]: https://github.com/RustScan/RustScan
