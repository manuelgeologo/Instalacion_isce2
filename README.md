## Instalacion isce2

Estas instrucciones fueron creadas siguiendo el procedimiento descrito en https://github.com/lijun99/isce2-install y agregando algunos pasos extras para instalar el linux (ubuntu 20.04 en mi caso).

## 1. Descargar anaconda

Lo primero que recomiendo es instalar anaconda para obtener los paquetes necesarios que usan programas como isce. Lo pueden descargar directamente de https://repo.anaconda.com/archive/Anaconda3-2020.11-Linux-x86_64.sh

En los cuadros de codigo el simbolo $ indica que se ingresa codigo en la terminal. Primero nos dirigimos a descargas donde se encuentra el archivo de instalación de anaconda.

```

$cd Downloads/
 
```

y luego corremos el instalador

```

$bash Anaconda3-2020.11-Linux-x86_64.sh

```

El nombre anterior lo pueden cambiar dependiendo de la versión que descarguen. Luego seguir las instrucciones que aparecen en la terminal. Cuando se haya installado deben agregar python al PATH para que puedan abrir los programas sin llamar la ruta completa. Para eso tienen que editar ~/.bashrc (dependiendo del shell que tengan podría tener otro nombre) y agregar la siguiente linea al final del archivo.

```
sudo gedit ~/.bashrc
export PATH="/home/nombreusuario/anaconda3/bin:$PATH"

```
Después tienen que cerrar y abrir la terminal o bien correr el siguiente comando:

```
$ source ~/.bashrc

```

Si tienes anaconda instalado de antes y ocupas otros programas que puedan verse alterados por actualizar paquetes o por conflictos entre estos. Recomiendo hacer un entorno nuevo para instalar isce2. Para crear un entorno pueden correr el siguiente comando, en mi caso use isce2 como nombre para el entorno. 

```
$ conda create -n isce2 python=3.8

```
Luego para activar el entorno solo escriben 

```
$ conda activate isce2

```
## 2. Instalar paquetes para isce2

