# Contenedores y cómo usarlos

## Ejercicio 1: Buscar alguna demo interesante de Docker y ejecutarla localmente o, en su defecto, ejecutar la imagen anterior y ver cómo funciona y los procesos que se llevan a cabo la primera vez que se ejecuta y las siguientes ocasiones

Vamos a usar como demo la imagen `jjmerelo/docker-daleksay`. La primera vez que ejecutamos esta imagen con `docker run --rm` obtenemos esta salida:

```
Unable to find image 'jjmerelo/docker-daleksay:latest' locally
latest: Pulling from jjmerelo/docker-daleksay
0a8490d0dfd3: Pull complete 
aac9fee8ebfb: Pull complete 
c951ce7a8ac7: Pull complete 
ab590fdbfbc0: Pull complete 
08ad84f33a7b: Pull complete 
Digest: sha256:9e2b66e742f80dd7422e48d69c0cf9a01afc9c2d50a6c2185fd37c3519a984ff
Status: Downloaded newer image for jjmerelo/docker-daleksay:latest
Use of uninitialized value in sprintf at daleksay/cowsay line 129.
 _ 
<   >
 - 
         \
          \
           \
            \
             \___
      D>=G==='   '.
            |======|
            |======|
        )--/]IIIIII]
           |_______|
           C O O O D
          C O  O  O D
         C  O  O  O  D
         C__O__O__O__D
snd     [_____________]
```

En esta primera ejecución, busca la imagen en la máquina local y, al no encontrarla, la descarga de Docker Hub. Concretamente, descarga cada una de las capas o _layers_ de las que se compone la imagen. A continuación, muestra el SHA de la imagen, finalizando la descarga, y ejecuta el contenedor.

La segunda vez que usamos `docker run --rm`, obtenemos la siguiente salida:

```
Use of uninitialized value in sprintf at daleksay/cowsay line 129.
 _ 
<   >
 - 
         \
          \
           \
            \
             \___
      D>=G==='   '.
            |======|
            |======|
        )--/]IIIIII]
           |_______|
           C O O O D
          C O  O  O D
         C  O  O  O  D
         C__O__O__O__D
snd     [_____________]
```

Al tener ya la imagen descargada, directamente ejecuta el contenedor.

## Ejercicio 2: Tomar algún programa simple, "Hola mundo" impreso desde la línea de órdenes, y comparar el tamaño de las imágenes de diferentes sistemas operativos base, Fedora, CentOS y Alpine, por ejemplo

## Ejercicio 3: Crear a partir del contenedor anterior una imagen persistente con *commit*

## Ejercicio 4: Examinar la estructura de capas que se forma al crear imágenes nuevas a partir de contenedores que se hayan estado ejecutando

## Ejercicio 5: Crear un volumen y usarlo, por ejemplo, para escribir la salida de un programa determinado

## Ejercicio 6: Usar un *miniframework* REST para crear un servicio web e introducirlo en un contenedor, y componerlo con un cliente REST que sea el que finalmente se ejecuta y sirve como *"frontend"*

## Ejercicio 7: Reproducir los contenedores creados anteriormente usando un ``Dockerfile``

## Ejercicio 8: Crear con ``docker-machine`` una máquina virtual local que permita desplegar contenedores y ejecutar en él contenedores creados con antelación
