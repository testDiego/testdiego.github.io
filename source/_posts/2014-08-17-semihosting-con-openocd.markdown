---
layout: post
title: "Semihosting con OpenOCD"
date: 2014-08-17 22:43:53 -0700
comments: true
categories: [ARM tools]
keywords: tutorial,arm,linux,microcontrollers,st,stm32f0,cortex-m0,openocd, gdb, semihosting
description: Habilitaremos la opcion de semihosting o como usar la funcion printf con OpenOCD  
---

Si has tenido la oportunidad de programar en C para la computadora, entonces de seguro conocerás la función `printf`. Esta función no es ninguna instrucción de **C** pero si es parte de la **libreria estandar de C**.

Sin entrar en mucho detalles la función `printf` nos permite por lo general mandar información por pantalla. En la computadora típicamente es el monitor.

Cualquier compilador decente de **C** para cualquier plataforma, trae esta librería estándar incluida. El compilador **GNU ARM** no es la excepcion y podremos, en nuestro programa utilizar la función `printf`. ¿Pero como?, si nuestro micro no tiene una pantalla!!.

Bueno nuestro micro no, pero la computadora que usamos para programar y debuggear si. Podemos usar el **debugger** ( _en el caso de la tarjeta Nucleo-f072rb es el ST-Link_ ) para mandar la informacion de la función `printf` de nuestro programa a la pantalla de **OpenOCD**.

Crearemos un nuevo proyecto con **CMSIS** incluido, como en el post [**Primer Programa con CMSIS**](http://testdiego.github.io/blog/2014/08/08/primer-programa-cmsis/). Llamaremos a ese proyecto `test_semihosting`, y en el archivo `main.c` escribimos lo siguiente
```C
#include "stm32f072xb.h"
#include <stdio.h>

extern void initialise_monitor_handles(void);

int main ( void )
{
    int i;

    initialise_monitor_handles();

    printf("Hola semihosting\n\r");
    for(;;)
    {
        /* bucle */
        for(i=0;i<10;i++)
        {
            printf("la variable i = %d\n\r", i);
        }
        printf("fin del bucle\n\n\r");
    }
}
```

<!--more--> 
Observa bien el codigo que acabas de escribir y nota que referenciamos la función `extern void initialise_monitor_handles(void)` esta funcion es la que habilitará al debugger para que despliegue la información por pantalla.

Es necesario mandar llamar esta función en tu programa antes de usar la función `printf`. tal como se aprecia en el codigo de ejemplo.

Aparte de lo anterior necesitaremos compilar el programa con un par de opciones extra. En las opciones del linker debemos colocar `--specs=rdimon.specs -lc -lrdimon`. En el archivo **makefile** escribe en `LCDFLAGS`
```
LDFLAGS = $(CPU) -Wl,--gc-sections --specs=rdimon.specs -lc -lrdimon
```

Compila y conectate a la tarjeta con **OpenOCD**. Al momento de usar **GDB** debemos agregar `mon enable semihosting` a la lista de opciones que normalmente usamos cuando cargamos un programa. Aqui la lista de los comandos.
```
diego@diego-X55A:~/test_semihosting$ arm-none-eabi-gdb Output/test.elf
GNU gdb (GNU Tools for ARM Embedded Processors) 7.6.0.20131129-cvs
Copyright (C) 2013 Free Software Foundation, Inc.
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.  Type "show copying"
and "show warranty" for details.
This GDB was configured as "--host=i686-linux-gnu --target=arm-none-eabi".
For bug reporting instructions, please see:
<http://www.gnu.org/software/gdb/bugs/>...
Reading symbols from /home/diego/test_semihosting/Output/test.elf...done.
(gdb) target remote localhost:3333
Remote debugging using localhost:3333
0x00000000 in ?? ()
(gdb) mon arm semihosting enable
semihosting is enabled
(gdb) load
Loading section .isr_vector, size 0xc0 lma 0x8000000
Loading section .text, size 0x759c lma 0x80000c0
Loading section .rodata, size 0x3b0 lma 0x8007660
Loading section .ARM, size 0x8 lma 0x8007a10
Loading section .init_array, size 0x8 lma 0x8007a18
Loading section .fini_array, size 0x4 lma 0x8007a20
Loading section .data, size 0x8b4 lma 0x8007a24
Start address 0x8002ebc, load size 33492
Transfer rate: 22 KB/sec, 4186 bytes/write.
(gdb) mon reset halt
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0xc1000000 pc: 0x08002ebc msp: 0x20003ffc, semihosting
(gdb)
```

Coloca un breakpoint en la linea `printf("fin del bucle\n\n\r"); ` y corre el programa. Podrás notar en la pantalla de **GDB** que nada aparece, pero si revisas la terminal en la que tienes corriendo **OpenOCD** podras ver la informacion enviada por `printf`.
```
Hola semihosting
la variable i = 0
la variable i = 1
la variable i = 2
la variable i = 3
la variable i = 4
la variable i = 5
la variable i = 6
la variable i = 7
la variable i = 8
la variable i = 9
```

Mola bastante verdad!!!, Pero antes de que te alegres deberías observar el tamaño de tu programa. Cuando usaste `load` GDB te dio esta información.
```
Start address 0x8002ebc, load size 33492
```

**33K de codigo!!!!**. Oh si!.  Usar la librería estándar agrega bastante código extra a tu programa y siendo honestos la libreria estandar de **C** que viene con **GNU ARM** no es la version completa. Se usa una version reducida, especial para embedded llamada **newlib**. Que, ¿creias que era gratis?.


Usando newlib-nano
------------------

Se puede usar una versión reducida de la aún reducida libreria estándar que acompaña a **GNU ARM** la version **newlib-nano**, solo necesitaras pasar al compilador las siguientes opciones `-Os -flto -ffunction-sections -fdata-sections -fno-builtin` y en el linker las opciones `--specs=nano.specs -lc -lnosys`.

Asi que  en tu makefile en GCFLAGS quedaria asi
```
CCFLAGS = $(CPU) $(INCLUDES) -Wall -fno-common -O0 -fomit-frame-pointer -Wstrict-prototypes -fverbose-asm -Os -flto -ffunction-sections -fdata-sections -fno-builtin
```

Y en LDFLAGS quedaría de este modo
```
LDFLAGS = $(CPU) -Wl,--gc-sections --specs=rdimon.specs -lc -lrdimon --specs=nano.specs -lc -lnosys
```

Vuelve a repetir el proceso anterior de compilar, conectar OpenOCD y cargar con GDB tu programa. Fijate cuando usas `load` la cantidad de codigo que en esta ocasion se carga a tu micro.
```
Start address 0x8000b44, load size 5520
```

Wow!. Se redujo a tan solo 5K, una cantidad mas considerable. Pero existe un precio como la imposibilidad de manejar numeros flotantes en la funcion `printf`

Uso de Stack y Heap
-------------------

Bueno no solo memoria de programa se consume por la funcion printf. Tambien se consume memoria RAM, **stack** y **heap** para ser mas especificos. Abre el archivo **STM32F072RB_FLASH.ld** con tu editor de texto y fijate en las líneas 37 y 38
```
_Min_Heap_Size = 0x200;      /* required amount of heap  */
_Min_Stack_Size = 0x400;     /* required amount of stack */
```

En este archivo se le da al stack 2k de memoria y al heap 1k. El uso de stack es muy común pues aquí es donde se almacenan las variables locales, pero a menos que se maneje memoria dinámica el valor de heap sera 0.

En este caso el archivo linker de ejemplo que usamos por default ya trae estos valores pero cuando no los tenga y uses la librería estándar ya sabes que deberás darle un valor adecuado ( _como entre 1k y 2k esta bien_ )

Conclusion
----------

El uso de la función `printf` es muy interesante, en especial en la etapa de desarrollo de tu proyectos, se vuelve tentadora la idea de usarla para depurar tu programa y poblar el software con cientos de `printf`. Pero como ves su uso no es gratuito y solo será útil mientras tengas un debugger conectado a tu micro, una vez que lo retires no servira de nada.

Por cierto la librería estándar no solo es la función printf, hay mucho más. Por que no vas y se lo preguntas a tu buscador favorito.
