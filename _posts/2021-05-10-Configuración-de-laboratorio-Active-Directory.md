---
layout: single
title:  Configuracion de laboratorio Active Directory
excerpt: "En este artículo, aprenderemos cómo montar un laboratorio de Active Directory para pruebas de penetración de manera local. Esto se realizará con el fin de que en proximos articulos se presentaran distintos tipos de ataques que se presentan en entornos empresariales."
date: 2021-05-10
classes: wide
header:
    teaser: /assets/images/adw/portada.png
    teaser_home_page: true
categories:
    - ActiveDirectory
tags:
    - windows
    - AD
---

<p align="center">
<img src="/assets/images/adw/portada.png">
</p>

- [requistos del laboratorio](requisitos-del-laboratorio)

## Requisitos del laboratorio 
- Maquina virtual Parrot OS
- Maquina virtual Windows Server 2016
- Maquina virtual Windows 10 (cliente 1)
- Maquina virtual Windows 10 (cliente 2)


Comenzamos instalando la ISO correspondiente al DC Windows Server 2016
<p align="center">
<img src="/assets/images/adw/1.png">
</p>
  
Aquí personalizará su sistema Windows proporcionándole su nombre de usuario y la contraseña que desea establecer. Luego haga clic en Siguiente para continuar.
<p align="center">
<img src="/assets/images/adw/2.png">
</p>

Nos dirigimos a la configuración de la máquina virtual y haremos clic en **Floppy**, Lo que haremos es eliminar esta caracteristica ya que si no lo hacemos puede generarnos un error en la instalación.
<p align="center">
<img src="/assets/images/adw/3.png">
</p>

Ahora seleccionará el sistema operativo para instalar entre las cuatro opciones que se indican a continuación. Aquí usamos **Standard Evaluation** con la GUI para tener una mejor interfaz de usuario.
<p align="center">
<img src="/assets/images/adw/5.png">
</p>

El sistema operativo debe comenzar a instalarse (esto puede tardarunos minutos).
<p align="center">
<img src="/assets/images/adw/6.png">
</p>

Y, en la opción de configuración personalizada, ingrese la contraseña que desea poner para la cuenta de administrador predeterminada.
En mi caso la contraseña que he puesto es **P@$$w0rd!**
<p align="center">
<img src="/assets/images/adw/7.png">
</p>


