---
layout: post
title: "Primer Programa Bare Board (pt I)"
date: 2014-08-06 16:17:22 -0700
comments: true
categories: [stm32f0]
keywords: tutorial,arm,linux,microcontrollers,st,stm32f0,cortex-m0,cmsis
description: Creacion de un primer proyecto para un micro stm32f0 utilizadno la tarjeta Nucleo-f072rb, el compilador gnu arm, openocd
---

Con nuestro compilador instalado y **OpenOCD** listo para comunicarse con nuestra tarjeta **Nucleo-F072RB**, llego la hora de crear nuestro programa _hola mundo_. Antes que nada aclararemos que no usaremos ningun IDE en especifico, para escribir el codigo usaremos cualquier editor de texto plano ( _te recomiendo Sublime Text_ ) y la compilacion la realizaremos usando archivos **makefiles** y la **terminal**.

El programa lo realizaremos sin el uso de ninguna libreria de funciones ( _just balls!!!_ ), pero si nesecitaremos un par de archivos de apoyo y los optendremos de la libreria oficial [**STM32F0Cube**](http://www.st.com/web/en/catalog/tools/PF260612) de ST. Descarga el archivo y descomprimelo en tu lugar favorito de la computadora =).

Crea una carpeta donde estara tu flamante y nuevo programa de prueba
```
$ mkdir ~/test_f072
$ cd ~/test_f072
``` 

Copia a la carpeta de tu proyecto el archivo **linker**, el cual permitira a tu programa saber como esta ordenada la memoria del micro. El archivo se encuentra en la libreria de **ST** en la siguiente ruta: 
```
STM32Cube_FW_F0_V1.0.0/Projects/STM32F072RB-Nucleo/Templates/TrueSTUDIO/STM32F072RB-Nucleo/STM32F072RB_FLASH.ld
```
<!--more-->
Copia a la carpeta de tu proyecto el archivo **startup** el cual permitira a tu programa entre otras cosas iniciar desde el vector de reset, setear el valor del stack pointer y acomodar la tabla de vectores de interrupciones del micro. EL archivo se encuentra en la libreria de **ST** en la siguiente ruta:
```
STM32Cube_FW_F0_V1.0.0/Drivers/CMSIS/Device/ST/STM32F0xx/Source/Templates/gcc/startup_stm32f072xb.s
```

Habra que hacer una pequeña modificacion al archivo de **startup_stm32f072xb.s** para que no nos de problema con una funcion que manda llamar ( _y que por lo pronto no tenemos_ ). Abre con tu editor favorito el archivo y comenta la linea numero **101**. _Al archivo le deberas quitar la proteccion contra escritura para realizar lo anterior_.
```
100 /* Call the clock system intitialization function.*/
101    //bl  SystemInit
102 /* Call static constructors */
```

En el directorio de tu proyecto crea una carpeta a la que llamaremos **Output** y sera la carpeta en la que dejaremos los archivos producidos por la compilacion
```
$ mkdir ~/test_f072/Output
```

Bien hora de la accion, crea un nuevo archivo al que llamremos **main.c** y que contendra nuestro codigo
```
$ touch ~/test_f072/main.c
```

Escribiendo el codigo
----------------------

Habre el archivo con tu editor de texto favorito ( _como Gedit_ ) y escribe el siguiente codigo
{% codeblock Hola Mundo - main.c %}
#define LED_PIN 5

int main ( void )
{
    volatile unsigned long i;
    
    /* Enable clock for GPIO port A */
    *((volatile unsigned long*)0x40021014) |= 0x00020000;
    /* Configure GPIOA pin 5 as output */
    *((volatile unsigned long*)0x48000000) |= (1 << (LED_PIN << 1));
    /* Configure GPIOA pin 5 in max speed */
    *((volatile unsigned long*)0x48000008) |= (3 << (LED_PIN << 1));

    for(;;)
    {
        /* Toggle pin 5 from port A */
        *((volatile unsigned long*)0x48000014) ^=(1 << LED_PIN);
        /* simple and practical delay */
        for(i=0;i<100000;i++);
    }
}
{% endcodeblock %}

El anterior codigo solo hara parpadear el led conectado al **puerto A pin 5**, el cual esta presente en la tarjeta. Y te estaras preguntando que son todos esos numeros??, pues son las direcciones en las que se encuntran los registros que deberemos manipular para interactuar con el **pin A5**. Esta informcaion la optienes de la hoja de datos del micro. 

Hora de compilar nuestro programa. La compilacion la realizaremos usando un archivo **makefile** el cual concentrara las ordenes de compilacion que le pasaremos a nuestro toolchain.

Crea un nuevo archivo llamado makefile en la carpeta de tu proeycto
```
$ touch ~/test_f072/makefile
```

Habre este nuevo archivo con tu editor de texto favorito ( _digamos atom_ ) y escribe lo siguiente
{% codeblock makefile lang:makefile %}
PROJECT = test  #Nombre que desees para tu proyecto

FILES = main.o startup_stm32f072xb.o #archivos fuente a compilar

# Apartir de aqui no modifiques nada a menos que sepas lo que hases. ;)
LINKERFILE = STM32F072RB_FLASH.ld # linker script a usar
CPU = -mcpu=cortex-m0 -mthumb -mlittle-endian

AS = arm-none-eabi-as
CC = arm-none-eabi-gcc
LD = arm-none-eabi-gcc
OD = arm-none-eabi-objdump
OC = arm-none-eabi-objcopy

CCFLAGS = $(CPU) -Wall -fno-common -O0 -fomit-frame-pointer -Wstrict-prototypes -fverbose-asm
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

No te apures si no sabes nada de **makefiles**, dentro de poco posteare un par de tutoriales. Otra cosa muy importante cuando escribes lo anterior usa **TABS** en la identacion y no espacios, o tendras errores.

Uff!!. Hora de compilar, solo escribe **make** en tu terminal, recuerda estar en la carpeta de tu proyecto
```
$ cd ~/test_f072
$ make
```

Si la terminal no te arrojo ningun error de compilacion deberas tener un archivo **test.hex** en tu folder **Output**, anda ve y revisa, porque hay que programar la tarjeta

A Programar se ha dicho
-----------------------

Habre una nueva terminal y conectate con tu tarjeta usando OpenOCD
```
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg
```

En la terminal anterior madaremos nuestro programa compilado a nuestra tarjeta usando **telnet**, conectate al puerto **4444** de la siguiente manera
```
$ telnet localhost 4444
```

Si te acepta la coneccion, solo restara mandar el archivo **.hex**. Escribe los siguientes comandos en orden
```
reset halt
flash write_image erase test.hex
reset run
```

El primer comando resetea y detiene al micro, el segundo manda el programa y lo escribe en la memoria y el ultimo lo resetea y pone a correr el programa, asi que ya podras ver un feliz led parpadeando.

Esta manera de programar ( _sin ayuda alguna_ ) te obligara a leer bien la hoja de datos del micro y en consecuencia aprenderas muy bien a utilizar la maquina que estas usando, pero te costara buenas desveladas y canas verdes.

Para terminar te dejamos la estrucutra del directorio de tu proyecto
```
.
├── main.c
├── makefile
├── Output/
├── startup_stm32f072xb.s
├── STM32F072RB_FLASH.ld

```
