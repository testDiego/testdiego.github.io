---
layout: post
title: "Programando con OpenOCD"
date: 2014-08-13 20:39:06 -0700
comments: true
categories: [ARM tools]
keywords: tutorial,arm,linux,microcontrollers,st,stm32f0,cortex-m0,openocd, flash
description: Diferentes formas de cargar un programa a la memoria flash del micro usando openocd 
---

Existen diferentes maneras en que podemos programar nuestro microcontrolador usando **OpenOCD**, ( _una muestra de su versatilidad_ ), y en este nuevo post examinaremos algunas de ellas, quedara en ti escoger la más adecuada para tu sistema de trabajo.

Antes que nada y para simplificar un poco utilizaremos el programa que creamos en el post [**Primer Programa Bare Board**](http://testdiego.github.io/blog/2014/08/06/primer-programa-bare-board) y daremos por echo que ya esta correctamente compilado y tenemos los archivos tets.elf y test.hex generados en el subfolder Output.


Usando Telnet
-------------

Esta opción ya la hemos manejado en los anteriores post, pero no está de más incluirla aquí. Primero que nada tendrás que conectarte a tu tarjeta con **OpenOCD**
```
$ cd ~/test_f072 #recuerda que debemos estar en alcarpeta del proyecto
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg
```
<!--more-->
Recuerda que debes indicar la interfaz que manejas ( _en este caso stlink_ ) y el micro a programar ( _en este caso stm32f0_ ). Y eso es lo que se hace en el código de arriba.

Por default **OpenOCD** estará escuchando en el **puerto 4444** para aceptar conexiones por **telnet**, así que lo siguiente es usar el programa **telnet** para conectarnos a ese mismo puerto. En otra terminal escribimos
```
$ cd ~/test_f072 #recuerda que debemos estar en la carpeta del proyecto
$ telnet localhost 4444
```  

Notamos como **OpenOCD** nos acepta la conexión, con lo cual estamos listos para mandarle comandos y que este los ejecute. Puedes investigar cuales son los comandos que acepta **OpenOCD** leyendo su manual de usuario.

Para programar la memoria del micro se usarán los siguientes
```
reset halt
flash write_image erase Output/test.hex
reset run
exit
```

El primer comando resetea y detiene al micro, el segundo manda el programa y lo escribe en la memoria y el último lo resetea y pone a correr el programa, asi que ya podras ver un feliz led parpadeando.


Usando solo OpenOCD
--------------------

Se puede usar únicamente **OpenOCD** sin necesitar de un programa externo como **telnet** para programar el micro. Simplemente le debes pasar algunos comandos a **OpenOCD** al momento de invocarlo
```
$ cd ~/test_f072 #recuerda que debemos estar en la carpeta del proyecto
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg -c "program Output/test.hex verify reset"
```

Si observas bien puedes darte cuenta que con la bandera `-c` puedes pasar comandos a **OpenOCD**. En este caso le estamos indicando que programe el micro con el archivo **.hex** `Output/test.hex` y que después de hacer la programación realice una verificación y un reset.

Esta nueva forma hace que **OpenOCD** se conecte a nuestro micro, cargue el programa, lo verifique y ponga correr el programa para después desconectarse de la tarjeta.


Usando Archivos .cfg con OpenOCD
--------------------------------

El anterior método resultó ser mucho más corto y sencillo que el anterior con **telnet**, pero en contra está la kilométrica línea con la que debemos invocar **OpenOCD** desde la terminal. Lo cual lo hace bastante engorroso y fácil de cometer equivocaciones.

Para evitar lo anterior escribiremos los argumentos con los que invocamos **OpenOCD** en un archivo `.cfg` ( _que son los que entiende OpenOCD_ ) así que en la misma carpeta de tu proyecto crea el archivo **openocd.cfg**
```
$ touch ~/test_f072/openocd.cfg
```

Abre el archivo recién creado con tu editor de texto favorito y escribe lo siguiente
```
# Indicamos que usaremos stlink como programador
source [find interface/stlink-v2-1.cfg]

# Indicamos que es micro Cortex-M0 de la marca ST
source [find target/stm32f0x_stlink.cfg]

# Programamos el archivo "test.hex" que se encuentra en la carpeta
# Output de nuestro proyecto y despues verificamos y damos un reset
program Output/test.hex verify reset

```

Ahora tenemos los anteriores parámetros escritos en un archivo en la carpeta de nuestro proyecto, con lo cual invocar **OpenOCD** se vuelve mas sencillo. Solo escribimos
```
$ cd ~/test_f072 #recuerda que debemos estar en la carpeta del proyecto
$ sudo openocd -f openocd.cfg
```

Como puedes ver a **OpenOCD** lo invocamos pasando el nombre del archivo que contiene los comandos que se deben ejecutar al conectarse a nuestra tarjeta. Un metodo simple y rápido, que nos da pie a crear formas más flexibles de conectarnos y programar nuestros dispositivos.


Conclusion
----------

Como puedes ver unas opciones parecen ser mejores que otras. Bueno eso dependerá de la forma que uses **OpenOCD**. Con telnet tienes una opción poderosa para realizar conexiones remotas, que tal si tu boards se encuentra conectado a una máquina diferente.

Y por otra parte los archivos `.cfg` son una herramienta poderosa al momento de crear diferentes configuraciones para acceder a tu tarjeta.

Así que escoge sabiamente ;) y si quieres mas información consulta el **Manual de usuario de OpenOCD**

