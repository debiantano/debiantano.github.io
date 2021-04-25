---
layout: single
title:  Hacking WiFi con la suite de aircrack-ng
excerpt: "Hackeando una red wifi con el método convencional haciendo uso de las herramientas que posee la suite de aircrack"
date: 2021-04-23
classes: wide
header:
    teaser: /assets/images/wpa/portada.png
    teaser_home_page: true
categories:
    - wifi
tags:
    - wifi
---

<p align="center">
<img src="/assets/images/wpa/portada.png" width="80%">
</p>

- [Interfaz en modo monitor](#interfaz-en-modo-monitor)
- [Captura del tráfico con airodump-ng](#captura-del-tráfico-con-airodump-ng)
- [Filtro con airodump-ng](#filtro-con-airodump-ng)
- [Desautenticacion a un cliente conectado en la red](#desautenticación-a-un-cliente-conectado-en-la-red)
- [ cracking del handshake](#cracking-del-handshake)

## Interfaz en modo monitor
El modo monitor resulta muy útil para ver qué paquetes se encuentran en nuestro entorno. Su utilidad se basa en que todos los paquetes que pasan tienen la información de a qué protocolo pertenece y las opciones de reensamblado. Incluso, si están cifrados, tienen la información en claro, es decir, que es posible saber qué contiene el paquete.  

Para activar este modo en nuestra tarjeta se hará so de la utilidad **airmon-ng**.

```
> airmon-ng start wlan0
```
<p align="center">
<img src="/assets/images/wpa/1.png" width="90%">
</p>

## Captura del tráfico con airodump-ng
Para la captura de paquetes wireless 802.11 se hará uso de **airodump-ng** y es necesario para ir acumulando vectores de inicialización IVs (en el caso del protocolo WEP).
Para este caso lo que haremos es captar toda la informacion de los Puntos de Acceso que se encuentren a nuestro alcance.

```
> airodump-ng <network>
```
<p align="center">
<img src="/assets/images/wpa/2.png">
</p>
  
<p align="center">
<img src="/assets/images/wpa/5.png width="75%">
</p>

---

## Filtro con airodump-ng
```
> airodump-ng -c 10 --bssid 70:4F:57:51:06:40 -w wpa2 wlan0
```
<p align="center">
<img src="/assets/images/wpa/10.png" width="90%">
</p>

## Desautenticaión a un cliente conectado en la red
Este ataque envia paquetes de disasociación a uno o más clientes que están actualmente asociados a un punto de acceso. 
El objetivo es conseguir que un cliente se conecte nuevamente a la red para que de esta manera podamos ser capaces de capturar el **handshake**.
<p align="center">
<img src="/assets/images/wpa/6.png" width="90%">
</p>

## Cracking del handshake
Para la fase de crackeo lo más aconsejable en entornos reales es crear nuestros propios diccionarios en base a la información del Punto de Acceso.  
OJO, este ataque siempre tiene que ser con consentimiento del objetivo o cliente, ya que si no estaríamos incurriendo en un delito.
Herramientaas como cupp, crunch, johnTheRipper son las mas usadas para la cración de diccionarios.
<p align="center">
<img src="/assets/images/wpa/8.png" width="90%">
</p>

