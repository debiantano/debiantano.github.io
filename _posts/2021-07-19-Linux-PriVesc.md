---
layout: single
title:  Escalada de Privilegios en Linux
excerpt: "En este post se presentará distintas maneras de poder conseguir el usuario administrador(root) del sistema partiendo de un usuario con pocos privilegios"
date: 2021-07-19
classes: wide
header:
    teaser: /assets/images/que-es-un-socket-y-como-funciona/portada.png
    teaser_home_page: true
categories:
    - Explicaciones
tags:
    - Linux
    - OSCP
---

El objetivo final con la escalada de privilegios en Linux es obtener un shell ejecutándose como usuario root.  
La escalada de privilegios puede ser simple (por ejemplo, un exploit de kernel) o requerir mucho reconocimiento sobre el sistema comprometido.  
En muchos casos, la escalada de privilegios puede no depender simplemente de una mala configuración, pero puede requerir que piense y combine múltiples configuraciones erróneas.   

### Usuarios
Las cuentas de usuario se configuran en el archivo /etc/passwd.  
Los hash de contraseña de usuario se almacenan en el archivo /etc/shadow.  
Los usuarios se identifican mediante un ID de usuario entero (UID).  
La cuenta de usuario "root" es un tipo especial de cuenta en Linux, tiene un UID de 0 y el sistema otorga a este usuario acceso a cada archivo.  

### Grupos
Los grupos se configuran en el archivo /etc/group.    
Los usuarios tienen un grupo principal y pueden tener varios grupos secundarios (o complementarios).  
De forma predeterminada, el grupo principal de un usuario tiene el mismo nombre como su cuenta de usuario.  

### Archivos y directorios
Todos los archivos y directorios tienen un solo propietario y un grupo.    
Los permisos se definen en términos de lectura, escritura y ejecución.  
Hay tres conjuntos de permisos, uno para el propietario, otro para el grupo, y uno para todos los "otros" usuarios.  
Solo el propietario puede cambiar los permisos.

### Permisos de archivos
• Leer: se puede leer el contenido del archivo.  
• Escritura: el contenido del archivo se puede modificar.  
• Ejecutar: el archivo se puede ejecutar (es decir, ejecutar como algún tipo de proceso).  


