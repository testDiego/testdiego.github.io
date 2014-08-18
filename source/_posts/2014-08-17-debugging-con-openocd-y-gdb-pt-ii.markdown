---
layout: post
title: "Debugging con OpenOCD y GDB (pt II)"
date: 2014-08-17 13:37:42 -0700
comments: true
categories: [ARM tools]
keywords: tutorial,arm,linux,microcontrollers,st,stm32f0,cortex-m0,openocd, gdb
description: valores de registros y variables en gdb con el comando print 
---

Continuamos con **GDB** y en esta ocasion veremos como observar el contenido de las variables y de los registros. Seguiremos retomando el programa que usamos en la [**primera parte**](http://localhost:4000/blog/2014/08/17/debugging-con-openocd-y-gdb/).

Lo primero que haremos sera modificar un poco nuestro prgrama, agregaremos unas variables que seran las que observemos. Abre el archivo `main.c` con tu editor favorito y modificalo de la siguiete manera
{% codeblock main.c %}
#include "stm32f072xb.h"

#define LED_PIN 5

unsigned long uno = 0; /*variable iniciada a cero*/
unsigned long dos = 2; /*variable iniciada a dos*/
 
int main ( void )
{
    volatile unsigned long i;
    unsigned long tres;  /*variable con valor incial indeterminado*/


    /* Enable GPIOA clock */
    RCC->AHBENR |= RCC_AHBENR_GPIOAEN;
    /* Configure GPIOA pin 5 as output */
    GPIOA->MODER |= (1 << (LED_PIN << 1));
    /* Configure GPIOA pin 5 in max speed */
    GPIOA->OSPEEDR |= (3 << (LED_PIN << 1));

    for(;;)
    {
        /* Toggle pin 5 from port A */
        GPIOA->ODR ^= (1 << LED_PIN);
        /*incrementamos variables de prueba*/
        uno++;     /*incremento en uno*/
        dos += 2;  /*incremento en dos*/
        tres++;    /*incremnteo en uno*/
        /* simple and practical delay */
        for(i=0;i<300000;i++);
    }
}
{% endcodeblock %}

<!--more--> 
Compila el programa y conecta **OpenOCD**.
```
$ cd ~/test_f072_CMSIS #recuerda que debemos estar en la carpeta del proyecto
$ make
$ sudo openocd
```

Carga el programa con **GDB**, recuerda que todo en esto es en otra terminal
```
$ cd ~/test_f072_CMSIS #recuerda que debemos estar en la carpeta del proyecto
$ arm-none-eabi-gdb Output/test.elf
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
(gdb) target remote localhost:3333
Remote debugging using localhost:3333
0x00000000 in ?? ()
(gdb) load
Loading section .isr_vector, size 0xc0 lma 0x8000000
Loading section .text, size 0x484 lma 0x80000c0
Loading section .rodata, size 0x8 lma 0x8000544
Loading section .init_array, size 0x8 lma 0x800054c
Loading section .fini_array, size 0x4 lma 0x8000554
Loading section .data, size 0x440 lma 0x8000558
Start address 0x8000378, load size 2456
Transfer rate: 12 KB/sec, 409 bytes/write.
(gdb) mon reset halt
target state: halted
target halted due to debug-request, current mode: Thread
xPSR: 0xc1000000 pc: 0x08000378 msp: 0x20003ffc
```

Vamos a colocar un **breakpoint** justo en  el mismo lugar que el anterior `GPIOA->ODR ^= (1 << LED_PIN);`. Pero como modifcamos el programa, ahora se encuntra en un numero de linea diferente.
```
(gdb) b main.c:25
Breakpoint 1 at 0x8000154: file main.c, line 25.
```

Corre el programa con `c` para que se dentenga justo en el breakpoint.
```
(gdb) c
Continuing.
Breakpoint 1, main () at main.c:25
25          GPIOA->ODR ^= (1 << LED_PIN);
```

Mostrando valores de variables
------------------------------

Bien ahora que el programa esta detenido podemos preguntar que valor poseen las variables justo en este punto del programa, y para ello usamos el comando `print` mejor abreviado como `p`. Preguntaremos por la varibale `uno`
```
(gdb) p uno
$1 = 0
```

Como puedes ver nos arroja el valor en decimal que posee la variable en ese momento. Ahora te toca a ti. Pregunta por el valor de las otras dos variables, `dos` y `tres`.

Corre de nueva cuenta el programa con `c`. Segun nuestro codigo las variables se incrementaran en uno y dos valores. Asi que de nueva cuenta interroga por el valor de las variables y observa como su valor efectivamente se incrementa. Yo observare cuanto vale `dos`
```
(gdb) p dos
$2 = 4
```

El comando `print` o `p` pude imprimir el valor de una variable en mas tipos de formatos a parte del decimal (default) solo deberemos agregar `\x` donde x es la letra correspodiente al formato. Ejemplo

**hexadecimal**
```
(gdb) p/x dos
$3 = 0x4
```
**binario**
```
(gdb) p/t dos
$3 = 100
```

Existen mas formatos como octal, sin signo o caracter. Puedes revisar [**Aqui**](http://davis.lbl.gov/Manuals/GDB/gdb_9.html#SEC58)


Mostrando valores de registros
------------------------------

No solo de las variables podemos observar sus valores actuales, que tal si queremos saber el contenido actual del registro `ODR`. Y saber si efectivamente el **bit 5** vale uno o dos.
```
(gdb) p/x GPIOA->ODR
No symbol "GPIOA" in current context.
```

Oh sorpresa!. No se puede, asi es no se puede porque `GPIOA` no es ninguna variable es una definicion, de un puntero a una estructura para ser mas exactos. Pero lo que si se puede es indicar que nos muestre la informacion de una direccion de memoria y resulta que el registro `GPIOA->ODR` se encuntra en la direccion `0x48000014`.
Lo expresamos en hexadecimal y realizamos un pequeño cast a la direccion  
```
(gdb) p/x *(int*)0x48000014
$6 = 0x20
```

Otra manera alterna es usar `x`. El cual es el comando para indicar contenidos de direcciones de memoria.
```
(gdb) x 0x48000014
0x48000014: 0x00000020
```

Dale continue con `c` y vuelve a imprimir el valor de la direccion para que observes como el bit 5 cambia de valor ( _y ve como concuerda con el led_ ).

Tengo que admitirlo, tener que saber la direccion de cada registro de cada periferico del micro, es algo engorroso y te obliga a tener siempre a la mano la hoja de datos del micro ( _o el archivo stm32f072xb.h_ ). Que se le va haser.


La opcion -g3 de GCC
---------------------

No todo esta perdido, memorizar las direcciones de los registros no es la unica opcion. Simplemente debemos compilar nuestro prgrama con la opcion -g3 ( _actualmente lo hacemos solo con al opcion -g_ ).

Hasique abre el archivo makefile de tu proyecto con tu editor de texto favorito y realiza el siguiente cambio
modifica la linea siguiente. De esto
```
    $(CC) $(CCFLAGS) -c -g -o $@ $^
```
a esto
```
    $(CC) $(CCFLAGS) -c -g3 -o $@ $^
```  

Compila de nuevo y carga de nuevo el programa a tu tarjeta usando OpenOCD y GDB, coloca el breakpoint corre el program y cuando se detenga pregunta por el valor de `GPIOA->ODR`
```
(gdb) p/x GPIOA->ODR
$7 = 0x20
```

Tu programa se puede compilar para generar informacion que ayudara al debugger a rastrear tu programa, tal como lo estamos haciendo. Existen tres niveles de informacion con la que podemos compilar. `-g0, -g1, -g2` y `-g3`. siendo `-g0` ninguna infromacion para debuggear y `-g3` el maximo nivel de informacion que podemos tener. El nivel por default es `-g2` o `-g`. 

 
Para informacion mas a detalle de GCC y GDB revisa

- [**GCC Debugging options**](https://gcc.gnu.org/onlinedocs/gcc-4.4.0/gcc/Debugging-Options.html#Debugging-Options)

- [**GDB print option**](http://davis.lbl.gov/Manuals/GDB/gdb_9.html#SEC58)
