---
layout: post
title: "Primer Programa con STM32F0Cube (pt III)"
date: 2014-08-10 13:30:28 -0700
comments: true
categories: [stm32f0]
keywords: tutorial,arm,linux,microcontrollers,st,stm32f0,cortex-m0,cmsis,stm32cubef0
description: Creacion de un primer proyecto para un micro stm32f0 utilizadno la tarjeta Nucleo-f072rb, el compilador gnu arm, openocd y la libreria STM32F0Cube de ST
---

En los dos anteriores post de la serie **Primer Programa** hemos estado utilizando archivos pertenecientes a la librería **STM32F0Cube**. Esta librería es un banco de código de funciones ya preestablecidas que nos permitirán empezar a codificar nuestras aplicaciones sin conocer tan a detalle el micro que estemos usando.

Llegó la hora de usar a fondo y sacar provecho de esas funciones, vamos a repetir el anterior ejemplo pero esta ocasión sacando provecho a librería **STM32F0Cube**.

Creamos una nueva carpeta para nuestro proyecto
```
$ mkdir ~/test_f072_cube
```

De nueva cuenta deberás copiar a la carpeta de tu proyecto el archivo linker. Recuerda lo obtienes de la librería de **ST** en la siguiente ruta
```
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/TrueSTUDIO/STM32F072RB-Nucleo/STM32F072RB_FLASH.ld
```
<!--more-->

Antes de comenzar, aclararemos que la librería es un conjunto de piezas de código que nos permitirán controlar por completo los periféricos de nuestro microcontrolador y esta diseñada para ser compatible con la totalidad de la línea de micros **STM32F0** de la marca **ST**. Estos codigos no sera necesario que los modifiques o adaptes excepto por un archivo en especial.

Copia el archivo **stm32f0xx_hal_conf_template.h** que encontrarás en la siguiente ruta. Deberas quitarle la proteccion contra escritura y renombrarlo a **stm32f0xx_hal_conf.h**
```
STM32Cube_FW_F0_V1.0.0/Drivers/STM32F0xx_HAL_Driver/Inc/stm32f0xx_hal_conf_template.h
```

Con este archivo podremos controlar varias funciones de los drivers de bajo nivel ( _HAL drivers_ ) de la librería. Una de las cosas que podemos controlar es la cantidad de drivers que vamos a usar  

Para facilitarnos un poco más las cosas copiamos por completo la carpeta de la libreria **STM32F0Cube** al folder de nuestro proyecto

Como en anteriores ocasiones creamos nuestra carpeta **Output** donde guardaremos los archivos que nos arroja la compilación
```
$ mkdir test_f072_cube/Output
```

Creamos el archivo que contendrá nuestro código magico =).
```
$ touch ~/test_f072_cube/main.c
```

Abrimos con nuestro editor de texto favorito el archivo **main.c** y escribimos el siguiente código
{% codeblock Hola Mundo STM32F0Cube - main.c %}
#include "stm32f0xx.h"

static GPIO_InitTypeDef  GPIO_InitStruct;

int main(void)
{
    HAL_Init(); /*init HAL library and tick interrupt to 1 ms*/

    /* -1- Enable each GPIO Clock (to be able to program the configuration registers) */
    __GPIOA_CLK_ENABLE();

    /* -2- Configure IOs in output push-pull mode to drive external LEDs */
    GPIO_InitStruct.Mode  = GPIO_MODE_OUTPUT_PP;
    GPIO_InitStruct.Pull  = GPIO_PULLUP;
    GPIO_InitStruct.Speed = GPIO_SPEED_HIGH;
    GPIO_InitStruct.Pin   = GPIO_PIN_5;

    HAL_GPIO_Init(GPIOA, &GPIO_InitStruct);

    /* -3- Toggle IOs in an infinite loop */
    while (1)
    {
        HAL_GPIO_TogglePin(GPIOA, GPIO_PIN_5);
        /* Insert delay 100 ms */
        HAL_Delay(100);
    }
}

void SysTick_Handler(void)
{
    HAL_IncTick();
}
{% endcodeblock %}

El anterior programa hará parpadear el led de la tarjeta conectado al **puerto A pin 5**. Pero en esta tercera ocasión usando las funciones de la librería **STM32F0Cube**. Como pudes observar en ningún momento accedemos directamente a los registros del microcontrolador.

No entraremos en detalles (_aun_) con las funciones que encontrarás en el código ( _eso lo haremos más adelante_ ). Por cierto por ahora el archivo  **stm32f0xx_hal_conf.h** no necesitara modificaciones.

Solo nos falta crear el archivo makefile para que make realice la compilación.
```
$ mkdir test_f072_cube/makefile
```

Abre el archivo en tu editor de texto y escribe el siguiente código. Recuerda usar TABs y no espacios para las indentación.
{% codeblock makefile lang:makefile %}
PROJECT = test

# archivos a compilar
FILES = main.o startup_stm32f072xb.o system_stm32f0xx.o \
stm32f0xx_hal.o \
stm32f0xx_hal_rcc.o stm32f0xx_hal_rcc_ex.o \
stm32f0xx_hal_gpio.o \
stm32f0xx_hal_cortex.o \
stm32f0xx_hal_flash.o

# definiciones de control
DEFINES = -DSTM32F072xB -DUSE_HAL_DRIVER

# directorios con archivos fuente (*.c y *.s)
VPATH = STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Source/Templates \
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Source/Templates/gcc \
STM32Cube_FW_F0_V1.0.0/Drivers/STM32F0xx_HAL_Driver/Src

# directorios con archivos headers (*.h)
INCLUDE = -I ./ \
-I STM32Cube_FW_F0_V1.0.0/Drivers/STM32F0xx_HAL_Driver/Inc \
-I STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Include \
-I STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Include

# a partir de aquí no modifiques nada a menos que sepas lo que haces
LINKERFILE = STM32F072RB_FLASH.ld # linker script to be use
CPU = -mcpu=cortex-m0 -mthumb -mlittle-endian

AS = arm-none-eabi-as
CC = arm-none-eabi-gcc
LD = arm-none-eabi-gcc
OD = arm-none-eabi-objdump
OC = arm-none-eabi-objcopy

CCFLAGS = $(CPU) $(DEFINES) -Wall -fno-common -O0 -fomit-frame-pointer -Wstrict-prototypes -fverbose-asm $(INCLUDE)
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

En esta ocasión compilamos varios archivos fuente y tenemos un directorio con la librería completa de **ST**.

A compilar se ha dicho
```
$ cd ~/test_f072_cube
$ make
```

Si la terminal no arrojo ningun error deberemos tener nuestro archivo **test.hex** en la carpeta Output

A Programar se ha dicho
-----------------------

Abre una nueva terminal y conéctate con tu tarjeta usando **OpenOCD**
```
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg
```

En la terminal anterior mandaremos nuestro programa compilado a nuestra tarjeta usando **telnet**, conéctate al puerto 4444 de la siguiente manera
```
$ telnet localhost 4444
```

Si te acepta la coneccion, solo resta mandar el archivo .hex, escribe los siguientes comandos en orden
```
reset halt
flash write_image erase test.hex
reset run
```

El primer comando resetea y detiene al micro, el segundo manda el programa y lo escribe en la memoria y el último lo resetea y pone a correr el programa, asi que ya podras ver un feliz led parpadeando.

Conclusion
----------

Incluir la librería completa de **ST** nos ayuda a no tener que conocer a fondo cómo funcionan los periféricos del microcontrolador, ahora solo tendremos que conocer a fondo cómo funciona la librería =). En general tener una librería como la **STM32F0Cube** representa una gran ventaja en especial si tomamos en cuenta que es compatible con toda la linea de micros de la familia **STM32F0**.

Solo hay que poner un poco de esfuerzo en como configurar los códigos e indicarle de manera correcta los directorios de los archivos fuente al compilador, costará un poco de trabajo al principio pero despues te acostumbras.

Para terminar te dejamos la estrucutra del directorio de tu proyecto
```
.
├── main.c
├── makefile
├── Output/
├── STM32Cube_FW_F0_V1.0.0/
├── STM32F072RB_FLASH.ld
└── stm32f0xx_hal_conf.h

```

