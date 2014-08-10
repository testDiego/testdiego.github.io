---
layout: post
title: "Instalando el compilador GNU ARM"
date: 2014-08-05 21:52:45 -0700
comments: true
categories: [ARM tools]
keywords: tutorial,GNU,ARM,microcontrollers
description: Pequeño tutorial para instalar el compilador GNU ARM en linux en su distribucion tarball del sitio official en launchpad
---

Vamos a empezar con nuestro primer post, y consistira en la instalacion y puesta a punto de nuestra herramienta de desarrollo favorita para microcontroladores **ARM**. El compilador **GCC** es totalemnte libre y lo mejor, no presenta limitacion alguna =).

Primero lo primero, Deberas descargar el toolchain de su pagina oficial [**aqui**](https://launchpad.net/gcc-arm-embedded). Si usas linux, como yo =) descarga el archivo **Linux Installation Tarball**.

Habre la terminal y crea un nuevo folder en tu directorio de usuario. Es en este directorio donde quedara instalado nuestro compilador.
```
$ mkdir ~/ARM
```

Descomprime el archivo descargado en el directorio recien creado (sustituye las x por la version que hayas descargado).
```
$ tar -xvjf gcc-arm-none-eabi-X_X-xxxxxx-xxxxxxxx-linux.tar.bz2 -C ~/ARM/
```
<!--more-->
El siguiente paso es agregar el directorio bin a la variable global **PATH** para que sea ejecutable por la terminal en cualquier directorio.
```
$ nano ~/.bashrc
```

Agrega la siguiente linea al final del archivo, sustituye las **x** con el numero de la version que tengas, fijate en el nombre de la carpeta que descomprimiste.
```
export PATH=$PATH:$HOME/ARM/gcc-arm-none-eabi-X_X-xxxxxx/bin
```

Reinicia la terminal y escribe el siguiente comando que te dira la version del toolchain instalado.
```
$ arm-none-eabi-gcc -v
```

Listo ya tienes tu compilador instalado.
