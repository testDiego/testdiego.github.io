---
layout: post
title: "Primer Programa CMSIS (pt II)"
date: 2014-08-08 13:07:34 -0700
comments: true
categories: [stm32f0]
keywords: tutorial,arm,linux,microcontrollers,st,stm32f0,cortex-m0,cmsis,stm32cubef0
description: Creacion de un primer proyecto para un micro stm32f0 utilizadno la tarjeta Nucleo-f072rb, el compilador gnu arm, openocd y el estandar CMSIS
---

En el post [**Primer Programa Bare Board**](http://testdiego.github.io/blog/2014/08/06/primer-programa-bare-board/) aprendimos como correr nuestro primer programa sin la ayuda de libreria y función alguna ( _excepto el startup y el linker file_ ). En esta ocasión usaremos la definición de registros acorde al estándar **CMSIS**.

**CMSIS** nos ayudará a acceder a los registros del micro de una forma más cómoda y organizada, además es un estándar difundido entre los fabricantes de micros con CPUs **ARM**,

Creamos una nueva carpeta para nuestro proyecto con **CMSIS**
```
$ mkdir ~/test_f072_CMSIS
```

De nueva cuenta deberás copiar a la carpeta de tu proyecto el archivo linker. Recuerda lo obtienes de la librería de ST en la siguiente ruta
```
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/TrueSTUDIO/STM32F072RB-Nucleo/STM32F072RB_FLASH.ld
```
<!--more-->
Crea una nueva carpeta en tu proyecto llamada **system** en la cual colocaremos los archivos CMSIS
```
$ mkdir ~/test_f072_CMSIS/system
```

Copia en esta nueva carpeta los siguientes archivos, los cuales los encontrarás en la librería de ST en la siguientes rutas
```
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Source/Templates/gcc/startup_stm32f072xb.s
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Source/Templates/system_stm32f0xx.c
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Include/stm32f072xb.h
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Include/system_stm32f0xx.h
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Include/arm_math.h
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Include/core_cm0.h
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Include/core_cmFunc.h
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Include/core_cmInstr.h
```

Antes de seguir avanzando, aclararemos que al archivo **startup** no le haremos ninguna modificación y que los archivos **system** nos permiten inicializar algunas pequeñas cosas del micro, como los relojes de las memorias.

Al que si modificaremos es al archivo **system_stm32f0xx.c**, Lo abrimos con nuestro editor de texto y escribimos lo siguiente apartir de la **linea 110**.
{% codeblock system_stm32f0xx.c %}
#if !defined  (HSI48_VALUE)
    #define HSI48_VALUE ((uint32_t)48000000)/*!< Default value of the Internal USB oscillator in Hz.
                                             This value can be provided and adapted by the user application. */
#endif
{% endcodeblock %}

y modificamos la **linea 84** de la siguiente manera
{% codeblock system_stm32f0xx.c %}
#include "stm32f072xb.h"
{% endcodeblock %}

Creamos nuestra carpeta **Output** donde guardaremos los archivos que nos arroja la compilación
```
$ mkdir test_f072_CMSIS/Output
```

Creamos el archivo que contendrá nuestro código magico =).
```
$ touch ~/test_f072_CMSIS/main.c
```

Abrimos con nuestro editor de texto favorito el archivo **main.c** y escribimos el siguiente código
{% codeblock Hola Mundo CMSIS - main.c %}
#include "stm32f072xb.h"

#define LED_PIN 5

int main ( void )
{
    volatile unsigned long i;
    
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
        /* simple and practical delay */
        for(i=0;i<300000;i++);
    }
}
{% endcodeblock %}

El anterior programa hará parpadear el led de la tarjeta conectado al **puerto A pin 5**. A diferencia del anterior post en este código no usamos las direcciones de los registros directamente, sino que usamos la definición de registros, lo cual es mucho más práctico.

Estos registros los encuentras definidos en el archivo **stm32f072xb.h**. Las definiciones son simples punteros a estructuras y esto es así porque lo dicta el estándar CMSIS `[Periferico]->[registro]`.

Solo nos falta crear el archivo makefile para que make realice la compilación.
```
$ touch test_f072_CMSIS/makefile
```

Abre el archivo en tu editor de texto y escribe el siguiente código. Recuerda usar TABs y no espacios para las indentación.
{% codeblock makefile lang:makefile %}
#Nombre que desees para tu proyecto
PROJECT = test  

#archivos a compilar
FILES = main.o startup_stm32f072xb.o system_stm32f0xx.o 

#directorio con archivos a compilar (*.c, *.s)
VPATH = system 

#directorio con archivos headers (*.h)
INCLUDES = -I system 

# Apartir de aqui no modifiques nada a menos que sepas lo que hases. ;)
# linker script a usar
LINKERFILE = STM32F072RB_FLASH.ld 
CPU = -mcpu=cortex-m0 -mthumb -mlittle-endian

AS = arm-none-eabi-as
CC = arm-none-eabi-gcc
LD = arm-none-eabi-gcc
OD = arm-none-eabi-objdump
OC = arm-none-eabi-objcopy

CCFLAGS = $(CPU) $(INCLUDES) -Wall -fno-common -O0 -fomit-frame-pointer -Wstrict-prototypes -fverbose-asm
ASFLAGS = $(CPU)
LDFLAGS = $(CPU) -Wl,--gc-sections
OCFLAGS = -Oihex
ODFLAGS = -S

all : test

test : $(PROJECT).elf
    $(OC) $(OCFLAGS) $(PROJECT).elf $(PROJECT).hex
    $(OD) $(ODFLAGS) $(PROJECT).elf > $(PROJECT).lst
    mv *.o *.elf *.hex *.lst Output

$(PROJECT).elf : $(FILES)
    $(LD) $(LDFLAGS) -T$(LINKERFILE) -o $@ $^

# Compile C source files
%.o : %.c
    $(CC) $(CCFLAGS) -c -g -o $@ $^

# Compile ASM files
%.o : %.s
    $(AS) $(ASFLAGS) -o $@ $^

clean :
    rm Output/*.*

{% endcodeblock %}

En esta ocasión compilamos tres archivos fuente y tenemos un subfolder el cual hay que indicarle al compilador.

A compilar se ha dicho
```
$ cd ~/test_f072_CMSIS
$ make
```

Si la terminal no arrojo ningun error deberemos tener nuestro archivo **test.hex** en la carpeta Output

A Programar se ha dicho
----------------------

Abre una nueva terminal y conectate con tu tarjeta usando OpenOCD
```
$ cd ~/test_f072_CMSIS  #Recomendacion, siempre estar en el directorio de tu proyecto
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg
```

En la terminal anterior madaremos nuestro programa compilado a nuestra tarjeta usando **telnet**, conectate al puerto **4444** de la siguiente manera
```
$ cd ~/test_f072_CMSIS #Recomendacion, siempre estar en el directorio de tu proyecto
$ telnet localhost 4444
```

Si te acepta la conexion, solo restara mandar el archivo **.hex**. Escribe los siguientes comandos en orden
```
reset halt
flash write_image erase Output/test.hex
reset run
```

El primer comando resetea y detiene al micro, el segundo manda el programa y lo escribe en la memoria y el último lo resetea y pone a correr el programa, asi que ya podras ver un feliz led parpadeando.

Conclusion
----------

Incluir los archivos que nos indica el estándar **CMSIS** nos facilita la interacción con los registros del sistema, pero eso no es lo unico ya que tambien nos permitira de una forma mas sencilla llamar las funciones de interrupción y controlar los registros internos del CPU.

Lo único que no nos exenta es el hecho de configurar los periféricos del micro de manera manual sin la ayuda de ningún framework o librería.

Para terminar te dejamos la estructura del directorio de tu proyecto
```
.
├── main.c
├── makefile
├── Output/
├── STM32F072RB_FLASH.ld
└── System/
    ├── arm_math.h
    ├── core_cm0.h
    ├── core_cmFunc.h
    ├── core_cmInstr.h
    ├── startup_stm32f072xb.s
    ├── stm32f072xb.h
    ├── system_stm32f0xx.c
    └── system_stm32f0xx.h

```
