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
<img src="/assets/images/adw/1.PNG">
</p>
  
Aquí personalizará su sistema Windows proporcionándole su nombre de usuario y la contraseña que desea establecer. Luego haga clic en Siguiente para continuar.
<p align="center">
<img src="/assets/images/adw/2.PNG">
</p>

Nos dirigimos a la configuración de la máquina virtual y haremos clic en **Floppy**, Lo que haremos es eliminar esta caracteristica ya que si no lo hacemos puede generarnos un error en la instalación.
<p align="center">
<img src="/assets/images/adw/3.PNG">
</p>

Ahora seleccionará el sistema operativo para instalar entre las cuatro opciones que se indican a continuación. Aquí usamos **Standard Evaluation** con la GUI para tener una mejor interfaz de usuario.
<p align="center">
<img src="/assets/images/adw/5.PNG">
</p>

El sistema operativo debe comenzar a instalarse (esto puede tardarunos minutos).
<p align="center">
<img src="/assets/images/adw/6.PNG">
</p>

Y, en la opción de configuración personalizada, ingrese la contraseña que desea poner para la cuenta de administrador predeterminada.
En mi caso la contraseña que he puesto es **P@$$w0rd!**
<p align="center">
<img src="/assets/images/adw/7.PNG">
</p>

Instalamos la VMWare Tools para que la proporción de la pantalla se vea completa.
<p align="center">
<img src="/assets/images/adw/8.PNG">
</p>


Tras pinchar, nos saldrá esto en el escritorio:
Haremos la instalación básica con todo por defecto, y una vez terminado, el Windows Server 2016 nos debería ocupar toda la pantalla.
Posteriormente tendremos que reiniciar para que se apliquen todos los cambios.
<p align="center">
<img src="/assets/images/adw/9.PNG">
</p>

----

Cambiaremos el nombre del equipo. Para ello, se hace click en **Cambiar configuración**:
<p align="center">
<img src="/assets/images/adw/10.PNG">
</p>

Y posteriormente, **Cambiar**:
<p align="center">
<img src="/assets/images/adw/11.PNG">
</p>

A modo de ejemplo, configuraremos el siguiente nombre de equipo:
Será necesario reiniciar el equipo para que los cambios tengan efecto.
<p align="center">
<img src="/assets/images/adw/12.PNG">
</p>

----


Situados en el centro de administrador del servidor:
<p align="center">
<img src="/assets/images/adw/13.PNG">
</p>

Presionaremos en **Administrar** y posteriormente en **Agregar roles y características**. Haremos un Skip de la primera parte:
<p align="center">
<img src="/assets/images/adw/14.PNG">
</p>


Posteriormente, **Instalación basada en características o en roles**
<p align="center">
<img src="/assets/images/adw/15.PNG">
</p>

Como rol de servidor, seleccionaremos la opción **Servicios de dominio de Active Directory**, y agregaremos sus características:
<p align="center">
<img src="/assets/images/adw/16.PNG">
</p>

En la siguiente pestaña, no haremos nada, simplemente le daremos a **Next**:
<p align="center">
<img src="/assets/images/adw/17.PNG">
</p>

Una vez llegados al último punto, le damos a Instalar:
<p align="center">
<img src="/assets/images/adw/18.PNG">
</p>

Finalizado la instalación cerramos.
<p align="center">
<img src="/assets/images/adw/19.PNG">
</p>

Aparecerá un nuevo aviso:
<p align="center">
<img src="/assets/images/adw/20.PNG">
</p>



