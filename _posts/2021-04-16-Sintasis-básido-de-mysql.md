---
layout: single
title:  Sintáxis básico en mysql
excerpt: "En este post se muestra los comandos mas simples y mas usados en este gestor de datos"
date: 2021-04-16
classes: wide
header:
    teaser: /assets/images/que-es-un-socket-y-como-funciona/portada.png
    teaser_home_page: true
categories:
    - Explicaciones
tags:
    - Tutoriales
    - Linux
    - Bash
    - Mysql
---

Antes de comenzar primero debemos fijarnos de que el servicio mysql esté activos.
#### Para sistemas Linux basados en debian:
```
> service mysql status | grep "Active"
     Active: active (running) since Fri 2021-04-16 23:08:29 -05; 20min ago
```

A continuacion se presentará los comandos mas simples de este gestor de datos aplicados en consola:

#### Mostrar las bases de datos
```
MariaDB [(none)]> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| flaskcontacts      |
| information_schema |
| mysql              |
| performance_schema |
| rogue_AP           |
| universidad        |
| wordpress          |
+--------------------+
```

#### hacer uso de una DDBB
```
MariaDB [(none)]> USE wordpress;
Reading table information for completion of table and column names
You can turn off this feature to get a quicker startup with -A

```

#### Mostrar las tablas de una DDBB
```
MariaDB [wordpress]> SHOW TABLES;
+-----------------------+
| Tables_in_wordpress   |
+-----------------------+
| wp_commentmeta        |
| wp_comments           |
| wp_links              |
| wp_options            |
| wp_postmeta           |
| wp_posts              |
| wp_users              |
+-----------------------+
```

#### Consultar todos los datos de una tabla (por ejm de la tabla wpa_keys)
```
MariaDB [rogue_AP]> SELECT * FROM wpa_keys;
+-----------+-----------+
| password1 | password2 |
+-----------+-----------+
| admin     | password  |
| noroot    | noroot    |
+-----------+-----------+

```
