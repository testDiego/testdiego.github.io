---
layout: post
title: "Instalando OpenOCD"
date: 2014-08-06 11:54:37 -0700
comments: true
categories: [ARM tools]
keywords: tutorial,openocd,gnu,arm,linux
description: Tutorial para compilar, cofigurar e instalar OpenOCD en linux con soporte para jlink, ftdi y stlink 
---

Ya con el compilador instalado, deberemos instalar un programa que nos permita comunicarnos con nuestro hardware y programarlo. Para ello usaremos **OpenOCD**, el cual es un pequeño servidor que se conectará con nuestro programador/debugger y nos permitirá pasar a nuestro micro el programa ya compilado.

**NOTA** Necesitamos instalar unas pequeñas librerias a manera de prerequisito (drivers USB). En Ubuntu escribe
```
$ sudo apt-get install libftdi-dev libusb-1.0-0-dev
```

**OpenOCD** es un software totalmente libre y en constante actualización, descarga la versión **0.8.0** de [**Aquí**](http://sourceforge.net/projects/openocd/files/openocd/0.8.0/).

Descomprime el archivo descargado con el siguiente comando en la terminal.
```
$ tar xvjf openocd-0.8.0.tar.bz2
```
<!--more-->
**OpenOCD** se distribuye usando sus archivos fuente, así que antes de ejecutarlo debemos compilarlo y despues instalarlo. No te preocupes son pasos muy sencillos.

Primero pasate a la carpeta de OpenOCD ya descomprimido
```
$ cd openocd-0.8.0 
```

Ahora lo configuraremos para que trabaje con ciertos debuggers, como **jlink**, **ftdi** y **stlink** ( _que es el que usaremos nosotros_ )
```
$ ./configure --enable-stlink --enable-jlink --enable-ftdi
```

Compilamos
```
$ make
```

Por último instalamos
```
$ sudo make install
```

**OpenOCD** quedó instalado en dos diferentes folders de nuestro sistema linux, el ejecutable ( _binario_ ) quedará en el directorio `/usr/local/bin`.


La otra carpeta importante `/usr/local/share/openocd/scripts` es donde quedarán instalados los scripts que nos servirán para identificar el micro que queremos programar y contiene la información necesaria para decirle a **OpenOCD** como programarlo, estos scripts están agrupados en tres niveles:

> - **target.-** scripts con instrucciones especificas para microcontroladores/procesadores
> - **interface.-** scripts con la informacion del debugger/programador a usar
> - **board.-** scripts con instrucciones que combinan interface y target. Y pertenecen a tarjetas especifcas que ya estan en el mercado

Antes de prenderle fuego a tu board ( _conectarla a OpenOCD_ ) deberemos registrar ciertas reglas, para que el debugger ( _ST-Link-V2-1_ ) sea reconocido por nuestro sistema. Para no entrar mas en detallas simplemente ejecuta lo siguietne en la terminal.
```
$ sudo cp  /usr/local/share/openocd/contrib/openocd.udev  /etc/udev/rules.d/98-openocd.rules
$ sudo udevadm control --reload-rules
```

Bueno basta de Charla, hay que probar si realmente funciona. Para ello usaremos nuestra nueva tarjeta **Nucleo-F072RB** de la marca **ST**, el cual por cierto posee un micro con CPU **Cortex-M0**. Conectala al puerto USB ( _se abrirá una ventana, pues la PC la identificara como un mass storage, no te apures solo cierra la ventana_ )

En la terminal ejecuta OpenOCD de la siguiente manera.
```
$ sudo openocd -f interface/stlink-v2-1.cfg -f target/stm32f0x_stlink.cfg
```

Si todo resulto bien te aparecera la siguiente información.
```
Open On-Chip Debugger 0.8.0 (2014-08-02-17:39)
Licensed under GNU GPL v2
For bug reports, read
    http://openocd.sourceforge.net/doc/doxygen/bugs.html
Info : This adapter doesn't support configurable speed
Info : STLINK v2 JTAG v22 API v2 SWIM v5 VID 0x0483 PID 0x374B
Info : using stlink api v2
Info : Target voltage: 3.246689
Info : stm32f0x.cpu: hardware has 4 breakpoints, 2 watchpoints
```

Con un `Ctrl+C` sales de openocd.

Si requieres mas informacion consulta:

- [openocd.org](openocd.org)
- [http://gnuarmeclipse.livius.net/blog/openocd-install/](http://gnuarmeclipse.livius.net/blog/openocd-install/)
