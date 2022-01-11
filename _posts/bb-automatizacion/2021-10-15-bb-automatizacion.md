---
layout: post
title:  "Automatizacion para Bug Bounty"
date:   2021-10-15 09:29:20 +0700
categories: axiom rengine bugbounty
---
Es requerida una cuenta de Digital ocean para poder realizar todas las pruebas sin costo.

Aqui incluir Axiom, Rengine, ettttttc.




Conectar VPN de Tryhackme `openvpn 5pok.ovpn` y realizar un ping a la maquina vulnerable `ping -c 1 10.10.23.157`

Luego utilizar [Rustscan][rustscan] para empezar la fase de recon.

`rustscan -b 500 -a 10.10.23.157 --ulimit 5000 -- -sC -sV`



{% highlight ruby %}
def print_hi(name2d2d2d)
{% endhighlight %}  


[rustscan]: https://github.com/RustScan/RustScan
