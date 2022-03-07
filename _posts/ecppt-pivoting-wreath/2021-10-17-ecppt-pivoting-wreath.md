---
layout: post
title:  "[eCPPT Style] - Pivoting Wreath"
date:   2021-10-17 09:29:20 +0700
categories: jekyll update
---
<figure>
<img src="/assets/img/wreath.png" alt="wreath">
</figure>

Este walkthrough muestra el paso a paso para explotar un laboratorio de tryhackme (Wreath), utilizado como entrenamiento para preparar el examen de OSCP o eCPTT.

Conectar VPN de Tryhackme `sudo openvpn 5pok.ovpn` y realizar un ping a la maquina vulnerable `ping -c 1 10.10.8.78`a


A.- Recon: Utilizar [Rustscan][rustscan] para empezar la fase de recon.

[rustscan]: https://github.com/RustScan/RustScan
