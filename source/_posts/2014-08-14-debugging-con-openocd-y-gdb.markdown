---
layout: post
title: "Debugging con OpenOCD y GDB"
date: 2014-08-14 23:12:51 -0700
comments: true
categories: [ARM tools]
keywords: tutorial,arm,linux,microcontrollers,st,stm32f0,cortex-m0,openocd, gdb
description: Como iniciar una sesion de Debugging para nuestra aplicacion usando openocd y gdb 
---

Llegó el momento de sacar mayor provecho al programador de nuestra tarjeta y es que nos solo sirve para programar. **ST-Link** nos sirve para depurar nuestro programa, realizar cosas como ejecuciones paso a paso, colocar breakpoints, revisar el contenido de una variable o registro, etc.

Antes de empezar deberemos tener un proyecto con el cual trabajar, así que tomaremos el proyecto del post [**Primer Programa con CMSIS**](http://testdiego.github.io/blog/2014/08/08/primer-programa-cmsis/). Asegúrate de que compile sin errores ( _puedes probarlo antes en la tarjeta_ ).

Bien, hora de conectarse a nuestra **Nucleo-f072** usando **OpenOCD**
```
$ cd ~/test_f072 #recuerda que debemos estar en la carpeta del proyecto
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg
```
<!--more-->

Wow!! Espera un segundo, mejor en vez de hacerlo de la manera tradicional, escribamos un archivo **.cfg** con las instrucciones para que **OpenOCD** se conecte. Si ya te conectaste solo presiona `Ctrl+C` para desconectarte.

En la misma carpeta de tu proyecto crea el archivo **openocd.cfg**
```
$ touch ~/test_f072_CMSIS/openocd.cfg
```

Abre el archivo recién creado con tu editor de texto favorito y escribe lo siguiente
```
# Indicamos que usaremos stlink como programador
source [find interface/stlink-v2-1.cfg]

# Indicamos que es micro Cortex-M0 de la marca ST
source [find target/stm32f0x_stlink.cfg]

# usamos hardware reset, para conectarnos solo bajo reset
reset_config srst_only srst_nogate
```

Ahora si, nos conectamos usando **OpenOCD** y el archivo que acabamos de crear.
```
$ cd ~/test_f072_CMSIS #recuerda que debemos estar en la carpeta del proyecto
$ sudo openocd
```

Si tienes dudas de lo anterior, solo recuerda que lo explicamos en el anterior post [**Programando con OpenOCD**](http://testdiego.github.io/blog/2014/08/13/programando-con-openocd/).


Usando GDB
----------

GDB es uno de los programas que vienen con tu compilador **GNU ARM** el cual instalaste en el [**primer post**](http://testdiego.github.io/blog/2014/08/05/instalando-el-compilador-gnu-arm/) y basicamente nos servira para ejecutar nuestro programa de manera guiada por asi decirlo. Solo continua con las instrucciones y te daras cuenta a que me refiero.

Abrimos otra terminal y en ella ejecutaremos **gdb** invocando el archivo **.elf** que nos arrojó la compilación de nuestro proyecto ( _Revisa la carpeta Output de tu proyecto_ )
```
$ cd ~/test_f072_CMSIS #recuerda que debemos estar en la carpeta del proyecto
$ arm-none-eabi-gdb Output/test.elf
```

**gdb** nos arrojará la siguiente información por pantalla.
```
GNU gdb (GNU Tools for ARM Embedded Processors) 7.6.0.20131129-cvs
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "--host=i686-linux-gnu --target=arm-none-eabi".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/diego/test_f072_CMSIS/Output/test.elf...done.
(gdb)
```

En la terminal tendremos el prompt correspondiente a **gdb** el cual nos indica que se esta ejecutando y que podremos escribir un comando para poner en funcionamiento nuestro programa.

Lo primero que debemos hacer es conectarnos al puerto **3333** que tiene disponible **OpenOCD**
```
(gdb) target remote localhost:3333
Remote debugging using localhost:3333
0x00000000 in ?? ()
```

Ya conectado **OpenOCD** con **GDB** lo siguiente es cargar a la memoria del micro nuestro programa. Lo hacemos con el comando `load`
```
(gdb) load
Loading section .isr_vector, size 0xc0 lma 0x8000000
Loading section .text, size 0x484 lma 0x80000c0
Loading section .rodata, size 0x8 lma 0x8000544
Loading section .init_array, size 0x8 lma 0x800054c
Loading section .fini_array, size 0x4 lma 0x8000554
Loading section .data, size 0x440 lma 0x8000558
Start address 0x8000378, load size 2456
Transfer rate: 12 KB/sec, 409 bytes/write.
```

Aplicamos un reset para iniciar desde la primera dirección de nuestro programa
```
(gdb) mon reset halt
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0xc1000000 pc: 0x08000378 msp: 0x20003ffc
```

A partir de este punto podemos empezar la ejecución del programa que tiene en memoria el micro. Podemos simplemente poner a correr el programa, pero mejor le diremos que solo lo corra hasta cierto punto.

Insertaremos un **breakpoint** justo en la línea del programa que invierte el estado del led. Revisa tu archivo `main.c` y fijate bien que numero de linea es donde se encuentra la instrucción `GPIOA->ODR ^= (1 << LED_PIN);` . En **gdb** escribe. ( _en nuestro caso es la línea 19_ )
```
(gdb) b main.c:19
Breakpoint 1 at 0x8000154: file main.c, line 19.
```

Ponemos a correr al micro con el comando `c` de _continue_
```
(gdb) c
Continuing.
Note: automatically using hardware breakpoints for read-only addresses.

Breakpoint 1, main () at main.c:19
19          GPIOA->ODR ^= (1 << LED_PIN);
```

Puedes observar en la terminal como **gdb** te indica que el programa se detuvo justo en la línea donde colocaste el breakpoint.


Vuelve a correr el programa con el comando `c` y observaras de nueva cuenta que se detiene en el **breakpoint**, observa tu tarjeta y ve como cada que presionas el comando `c` el led se invierte.

Puedes colocar mas **breakpoints** en diferentes lugares de tu código, solo no te olvides donde los pusiste, y si se te olvide solo escribe `info break`
```
(gdb) info break
Num     Type           Disp Enb Address    What
1       breakpoint     keep y   0x08000154 in main at main.c:19
    breakpoint already hit 2 times
```

Si quieres que el program no se detenga y se ejecute de manera continua tendrás que borrar el **breakpoint**. Lo puedes hacer con `clear`
```
(gdb) clear main.c:19
Deleted breakpoints 1
```

En este caso el breakpoint estaba en el archivo `main.c`, linea **19**

Escribe `c` para que tu program empiece a correr, ya sin ningun breakpoint que lo detenga. Mira como parpadea el led!!.

Para detener la ejecución de tu programa tendrás que ejecutar un `Ctrl+C`. El programa se detendrá justo en la instrucción que estaba ejecutando. En nuestro caso fue el ciclo for
```
^C
Program received signal SIGINT, Interrupt.
0x0800016e in main () at main.c:21
21          for(i=0;i<300000;i++);
```

Para salir de **gdb** solo escribe `q`, nos pedira confirmacion asi que tecleamos `y`
```
(gdb) q
A debugging session is active.

    Inferior 1 [Remote target] will be detached.

Quit anyway? (y or n) y
Detaching from program: /home/diego/test_f072_CMSIS/Output/test.elf, Remote target
Ending remote debugging.
```

Que tal, a que no sabias que se podía hacer. Nah, probablemente si. Para terminar **GDB** es una herramienta muy buena para depurar y encontrar errores en tu programa cuando estas en la fase de desarrollo. Hay muchas cosas mas interesantes que se pueden hacer con gdb, pero eso lo veremos mas adelante.

