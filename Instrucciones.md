## Instalacion isce2 (cmake)

Estas instrucciones fueron escritas siguiendo el procedimiento descrito en https://github.com/lijun99/isce2-install y agregando algunos pasos extras para instalar en linux (ubuntu 20.04 en mi caso).

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
$ sudo gedit ~/.bashrc
export PATH="/home/nombreusuario/anaconda3/bin:$PATH"

```
Después tienen que cerrar y abrir la terminal o bien correr el siguiente comando:

```
$ source ~/.bashrc

```

Si tienes anaconda instalado de antes y ocupas otros programas que puedan verse alterados por actualizar paquetes o por conflictos entre estos. Recomiendo hacer un entorno nuevo para instalar isce2. Para crear un entorno pueden correr el siguiente comando, en mi caso use isce2 como nombre para el entorno. Segun las instrucciones originales de isce se recomienda tener python 3.8.

```
$ conda create -n isce2 python=3.8

```
Luego para activar el entorno solo escriben 

```
$ conda activate isce2

```
Al usar este procedimiento se instalará casi todo en la ruta default de conda. Esto se puede ver usando el siguiente comando en la terminal.

            $ echo $CONDA_PREFIX

La ruta que les saldrá será algo como 

            home/username/anaconda3
            
Esta ruta la ocuparemos para dar la instrucción a la instalación. Me parece que también funciona usando simplemente `$CONDA_PREFIX` pero yo copié la ruta completa por si acaso. 
## 2. Instalar paquetes para isce2

acá hay varios paquetes que se deben instalar, tienen que tener ojo que se instalen todos, y si alguno no se instala buscar un equivalente en google.

```
conda install git cmake cython gdal h5py libgdal pytest numpy fftw scipy basemap opencv 
```
y además, para que podamos usar correctamente el mdx debemos instalar los siguientes paquetes. Especial cuidado con estos que dependiendo de la versión de linux que tengan pueden variar de nombre levemente. 

```
   conda install openmotif openmotif-dev xorg-libx11 xorg-libxt xorg-libxmu xorg-libxft libiconv xorg-libxrender xorg-libxau xorg-libxdmcp 
```
Me parece que para que el programa pueda usar la tarjeta de video para ayudar en los procesos se debe tener un compilador CUDA (usualmente se encuentra en /usr/loca/cuda). No sé bien como funciona esto pero en caso de llegar a ocuparlo se puede cargar con `$ module load cuda`

También se necesitan los compiladores de lenguajes C/C++/Fortran. Estos se pueden instalar facilmente con conda:

               conda install gcc_linux-64 gxx_linux-64 gfortran_linux-64

Dependiendo de la versión de CUDA que tengan, los compiladores deberán instalarse en cierta versión. Para ver que versión tienen, tienen que correr:


Si su CUDA es por ejemplo 10.1 recomiendan usar los compiladores en versión <= 7.3.

              conda install gcc_linux-64=7.3.0 gxx_linux-64=7.3.0 gfortran_linux-64=7.3.0

## 3. Descargar los archivos de ISCE2

Cuando tenemos ya los paquetes instalados debemos descargar los archivos del repositorio de isce y dejarlos en una carpeta (la que queramos)

            $ mkdir -p $HOME/username/isce_version__nn/src
            $cd $HOME/username/isce_version_nn/src
            $git clone https://github.com/isce-framework/isce2.git

Se descargará la carpeta de isce y entramos a la carpeta isce2. Luego creamos una carpeta donde se compilará por decir algo isce (en este caso 'build') y entramos a ella. Luego debemos decirle cual es la dirección donde se instalarán los paquetes `DCMAKE_INSTALL_PREFIX`, si lo dejamos por default cuando corramos los modulos de isce2 podemos hacerlo directamente sin llamar la ruta completa. En esta parte creo que se puede dejar como está el código más abajo o cambian el `CONDA_PREFIX` por la ruta que vimos al comienzo `$/home/username/anaconda3`. 

También debemos decirle donde instalará los scrips de python, y esta carpeta será relativa a la anterior que dimos a `DCMAKE_INSTALL_PREFIX`. Entonces para ver esta dirección y corroborar que esté bien, corremos: 

            python3 -c 'import site; print(site.getsitepackages())'

Nos debería dar algo como `'home/username/anaconda3/envs/isce2/lib/python3.8/site-packages'` Entonces en `DPYTHON_MODULE_DIR` ponemos la ultima parte de esta linea anterior (desde lib).

Ahora `DCMAKE_CUDA_FLAGS` tiene relación a la arquitectura de la tarjeta de video que tengamos. Me parece que solo para tarjetas nvidia funciona esto. lo que se cambia es el valor "sm_60", debería salir en google el valor que puede corresponder a su tarjeta, yo el mío lo encontré acá: https://arnon.dk/matching-sm-architectures-arch-and-gencode-for-various-nvidia-cards/

No sé qué tan relevante sea, pero si no lo encuentran dejen el que está en el código de abajo. 
`DCMAKE_PREFIX_PATH` es la ruta donde isce buscará los paquetes, como todo se instaló con conda, le damos el default. 
`DCMAKE_BUILD_TYPE=(None,Debug,Release)` esta flag tiene tres opciones none,debug,release, y según las instrucciones para usuarios comunes y corrientes recomiendan dejar la opción = Release. Entonces cuando cuando corramos las dos últimas lineas del código de abajo se instalará isce2.  


            $cd isce2
        $mkdir build  && cd build
        $cmake .. -DCMAKE_INSTALL_PREFIX=$CONDA_PREFIX -DPYTHON_MODULE_DIR=lib/python3.8/site-packages -DCMAKE_CUDA_FLAGS="-arch=sm_60" -DCMAKE_PREFIX_PATH=${CONDA_PREFIX} -DCMAKE_BUILD_TYPE=Release 
        $make -j 16 # to use multiple threads
        $make install 


Si aparecé algún error (el programa muestra el progreso en % a medida que avanza) con respecto a los compiladores. Podemos darle las rutas directas (que tenemos que buscar en nuestro computador manualmente) para que isce las use. Esta linea de abajo se agrega a la linea `$cmake ..` del paso anterior.


            -DCMAKE_C_COMPILER=/path/to/gcc -DCMAKE_CXX_COMPILER=/path/to/g++ -DCMAKE_Fortran_COMPILER=/path/to/gfortran

Solo tienen que cambiar donde dice /path/to/ por las carpetas que correspondan en su computador. Con esto debería instalar bien (al menos a mi me funcionó). Si les sigue saliendo algún error corran el siguiente comando:

            make VERBOSE=1
Esto les mostrará más detalle de los errores que vayan teniendo. Puede que les aparezcan errores al 30% de instalar y si lo arreglan puede que les salga otro error distinto más adelante (a mi me salió un error a los 98%, y fue porque no se instaló un package en el comienzo de todo el proceso). 

## 5. Corroborar que la instalación haya funcionado

si van a la carpeta de isce2, deberian estar los módulos instalados. Por ejemplo: mdx.py, alos2App.py, etc.

              cd $CONDA_PREFIX/bin
              ls -ltr
              cd ../lib/python3.x/site-packages
              ls -ltr
              
Cuando vayan a la segunda carpeta (site-packages) deberian ver que aparece isce --> path/to/isce2

Ahora para probar que se instaló bien. Podemos correr:

            python3 -c 'import isce'
            topsApp.py -h

Acá nos aparece el mensaje "This is the open source version of ISCE" y muchas lineas de información respecto a topsApp y las opciones que trae. Después cuando queramos volver a usar isce simplemente activamos el entorno donde los instalamos. En este caso isce2. 

            conda activate isce2

Como mencioné al comienzo, estas instrucciones fueron adaptadas de https://github.com/lijun99/isce2-install por lo que algún detalle extra pueden encontrarlo allí. Incluso otra forma de instalación con scons e instrucciones para instalar en mac. Yo adapté la primera que fue la que me funcionó sin mucho problema. Suerte con la instalación. 

