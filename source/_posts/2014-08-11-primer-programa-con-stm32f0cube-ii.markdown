---
layout: post
title: "primer programa con stm32f0cube (pt IV)"
date: 2014-08-11 21:13:25 -0700
comments: true
categories: [stm32f0]
keywords: tutorial,arm,linux,microcontrollers,st,stm32f0,cortex-m0,cmsis,stm32cubef0
description: Creacion de un primer proyecto para un micro stm32f0 utilizadno la tarjeta Nucleo-f072rb, el compilador gnu arm, openocd y la libreria STM32F0Cube de ST 
---

La última parte de la serie **Primer Programa** en la cual abordaremos un poco más de la librería **STM32F0Cube**. Primero que nada una explicación de lo que contiene esta librería al descomprimir el archivo **zip** en el que viene, te encuentras con:
```
.
├── Documentation
├── Drivers
├── _htmresc
├── Middlewares
├── package.xml
├── Projects
├── Release_Notes.html
└── Utilities
```

####Drivers
Es uno de los directorios más importantes, en el encontramos el estándar **CMSIS** que contiene definidos los registros de los periféricos del micro así como los vectores de interrupción. Lo otro que encontramos de gran importancia es la librería de **HAL drivers** con las piezas de código que nos permitirán controlar los periféricos a través de funciones y macros ya bien definidos y probados.
```
.
├── BSP
├── CMSIS
└── STM32F0xx_HAL_Driver
```
<!--more-->
####Middelwares
Este es el segundo directorio de importancia el cual contiene librerias de codigo que nos permitiran realizar cosas mucho mas interesantes y que requieren un nivel mayor de expertis.
```
.
├── ST
│   ├── STM32_TouchSensing_Library
│   └── STM32_USB_Device_Library
└── Third_Party
    ├── FatFs
    └── FreeRTOS
```

####Projects
Aqui no encontrarás piezas de código reusables, solo unos cuantos ejemplos en los que se usa la librería para algunas tarjetas que están en el mercado entre las cuales encontramos la **Nucleo-f072rb**
```
.
├── STM32072B_EVAL
├── STM32F0308-Discovery
├── STM32F030R8-Nucleo
├── STM32F072B-Discovery
└── STM32F072RB-Nucleo
```

El resto de los directorios son documentación y alguna utileria para actualizar la propia librería.


Proyecto Recomendado por ST
-----------------------------

En la [**tercera parte**](http://testdiego.github.io/blog/2014/08/10/primer-programa-con-stm32f0cube/) de esta serie de post utilizamos las funciones que vienen con esta librería, si bien nuestro programa corrió exitosamente el codigo no esta ordenado de la forma que **ST** nos recomendaría si estamos usando **STM32F0Cube**. Así que lo correcto seria lo siguiente.

De nueva cuenta crea una nueva carpeta para nuestro proyecto
```
$ mkdir ~/test_f072_cubeII
```

Y al igual que en los anteriores deberás copiar a la carpeta de tu proyecto el archivo **linker**. Recuerda lo obtienes de la librería de **ST** en la siguiente ruta
```
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/TrueSTUDIO/STM32F072RB-Nucleo/STM32F072RB_FLASH.ld
```

Bien, **ST** nos sugiere que ordenemos nuestro proyecto en dos directorios, uno donde iran los archivos headers ( _*.h_ ) **Inc** y otro con los archivos fuente ( _*.s y *.c_ ) **Src**, asi que creamos estos directorios en nuestro proyecto.
```
$ mkdir ~/test_f072_cubeII/Inc
$ mkdir ~/test_f072_cubeII/Src
```

Los archivos que deberán ir en cada uno de los directorios los podremos crear desde cero o copiarlos de un lugar donde ya están escritos, como un template que viene en la librería.

Copia los siguientes archivos de la siguiente ruta en el directorio **Inc** de tu proyecto
```
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/Inc/main.h
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/Inc/stm32f0xx_hal_conf.h
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/Inc/stm32f0xx_it.h
```

Y aqui la descripcion de cada uno de esos archivos

- **main.h.-**Aquí colocaras las funciones, variables y/o macros que tengan carácter global y que deben referenciar otros archivos de tu programa
- **stm32f0xx_hal_conf.h.-** Viejo conocido en donde configuras lo que se use y lo que no se use de la librería STM32F0Cube
- **stm32f0xx_it.h.-** Aquí colocaras los prototipos de las funciones que actúan como vectores de interrupción

Copia los siguientes archivos de la siguiente ruta en el directorio Src de tu proyecto
```
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/Src/main.c
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/Inc/stm32f0xx_hal_msp.c
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/Inc/stm32f0xx_it.c
```

Y aqui la descripcion de cada uno de los archivos

- **main.c.-** Aqui va tu aplicación. Ya contiene algunas líneas de código que ST te sugiere que tenga por default
- **stm32f0xx_hal_msp.c.-** En este archivo colocaras las funciones callback que ocupes de la librería con código específico de tu aplicación
- **stm32f0xx_it.c.-** Aqui van los vectores de interrupción y donde típicamente mandas llamar las funciones que se necesiten ejecutar en una interrupción
- **system_stm32f0xx.c.-** Aqui se encuentra las configuraciones inciales de los reloj del sistema ( _ya no ocuparas hacerle cambios_ )

Tenemos que hacer una pequeña modificación en el archivo **main.h** para que nuestro proyecto no quede tan dependiente de la tarjeta **Nucleo-f072rb** y que quede más genérico. Abre el archivo con tu editor de texto favorito y sustituyes la **línea 44** por la siguiente
{% codeblock main.h lang:c%}
#include "stm32f0xx.h"
{% endcodeblock %}

Abre el archivo **main.c** con tu editor de texto, podras observar bastante código ya escrito así como unos voluminosos comentarios escritos por los ingenieros de **ST**. No te asustes, si observamos la función main podras ver que se manda llamar la funcion ya conocida `HAL_Init()` y una nueva `SystemClock_Config()`, esta funcion cuyo cuerpo encuentras lineas mas abajo configura el reloj del sistema para uqe corra a 48MHz ( _velocidad maxima del micro_ ) usando las funciones del periferico **RCC**.

Inserta la siguiente declaracion en la **linea 53**
{% codeblock main.c %}
/* Private variables ---------------------------------------------------------*/
static GPIO_InitTypeDef  GPIO_InitStruct;
/* Private function prototypes -----------------------------------------------*/
{% endcodeblock %}


A partir de la línea  **84** colocaremos nuestro código y dentro del ciclo while infinito también
{% codeblock main.c %}
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
{% endcodeblock %}

Si eres observador notarás que la función `void SysTick_Handler(void)` ya no se encuentra en el archivo main.c. Como esta función es un vector de interrupción ahora se encuentra en el archivo **stm32f0xx_it.c** ( _linea 109_ ).

Al directorio de tu proyecto le hace falta algo ( _no es el makefile_ ) y es la libreria **STM32F0Cube** podemos copiarla al folder de tu proyecto como en el anterior post, nada mas te recuerdo que pesa **82.3 MB**. 

La librería no está diseñada para la copies en cuanto proyecto trabajes, sino más bien la dejes en un lugar fijo y referencies sus directorios desde ese mismo lugar para todos tus proyectos.

Puedes dejarla en donde sea que la hayas descomprimido, pero que tal si las dejas en un lugar muy estándar de linux, tu home directory ( **~ ** ), de tal manera que tu libreria tendría el siguiente destino. Esto ayudará mucho a la portabilidad de tus proyectos.
```
~/STM32Cube_FW_F0_V1.0.0
```

Por último, el archivo makefile para que make realice la compilación.
```
$ touch test_f072_cubeII/makefile
```

Abre el archivo en tu editor de texto y escribe el siguiente código. Recuerda usar TABs y no espacios para las indentación.
{% codeblock makefile lang:makefile %}
PROJECT = test

# archivos a compilar
FILES = main.o startup_stm32f072xb.o system_stm32f0xx.o stm32f0xx_it.o\
stm32f0xx_hal.o \
stm32f0xx_hal_rcc.o stm32f0xx_hal_rcc_ex.o \
stm32f0xx_hal_gpio.o \
stm32f0xx_hal_cortex.o \
stm32f0xx_hal_flash.o

# definiciones de control
DEFINES = -DSTM32F072xB -DUSE_HAL_DRIVER

# directorios con archivos fuente (*.c y *.s)
VPATH = Src \
/home/diego/STM32Cube_FW_F0_V1.0.0/Drivers/STM32F0xx_HAL_Driver/Src

# directorios con archivos headers (*.h)
INCLUDE = -I ./ \
-I ./Inc \
-I ~/STM32Cube_FW_F0_V1.0.0/Drivers/STM32F0xx_HAL_Driver/Inc \
-I ~/STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Include \
-I ~/STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Include

# a partir de aquí no modifiques nada a menos que sepas lo que haces
# linker script to be use
LINKERFILE = STM32F072RB_FLASH.ld
CPU = -mcpu=cortex-m0 -mthumb -mlittle-endian

AS = arm-none-eabi-as
CC = arm-none-eabi-gcc
LD = arm-none-eabi-gcc
OD = arm-none-eabi-objdump
OC = arm-none-eabi-objcopy

CCFLAGS = $(CPU) $(DEFINES) $(INCLUDE) -Wall -fno-common -O0 -fomit-frame-pointer -Wstrict-prototypes -fverbose-asm
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

Fijate que ahora direccionamos los directorios de la librería desde su nuevo destino que seria tu directorio de usuario **ST**.

A compilar se ha dicho
```
$ cd ~/test_f072_cubeII
$ make
```

Si la terminal no arrojo ningun error deberemos tener nuestro archivo **test.hex** en la carpeta Output

A Programar se ha dicho
-----------------------

Abre una nueva terminal y conectate con tu tarjeta usando OpenOCD
```
$ cd ~/test_f072_cubeII  #Recomendacion, siempre estar en el directorio de tu proyecto
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg
```

En la terminal anterior madaremos nuestro programa compilado a nuestra tarjeta usando **telnet**, conectate al puerto **4444** de la siguiente manera
```
$ cd ~/test_f072_cubeII #Recomendacion, siempre estar en el directorio de tu proyecto
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

Bastantes archivos y codigo para un simple parpadeo de un led te estarás diciendo. Tienes razon pero piensa que la librería y sobre todos el ordenar tu proyecto de esta manera esta pensando para que realices desde pequeños hasta grandes proyectos con una gran cantidad de código y archivos fuente y es en estos último cuando adquiere gran importancia ordenar bien tu código.

Un punto muy importante que **ST** maneja una herramienta que permite configurar la librería **STM32F0Cube** y generar código a partir de una [**interfaz gráfica**](http://www.st.com/web/en/catalog/tools/PF259242). dicha interfaz gráfica necesitará que tu proyecto lo tengas ordenado de esta manera.

Para terminar te dejamos la estructura del directorio de tu proyecto
```
.
├── Inc
│   ├── main.h
│   ├── stm32f0xx_hal_conf.h
│   └── stm32f0xx_it.h
├── makefile
├── Output/
├── Src
│   ├── main.c
│   ├── stm32f0xx_hal_msp.c
│   ├── stm32f0xx_it.c
│   └── system_stm32f0xx.c
├── startup_stm32f072xb.s
└── STM32F072RB_FLASH.ld
```
