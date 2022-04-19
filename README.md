# Volumes

Los volumes es el mecanismo que se recomienda usar para que la información generada por in container se mantenga persistente.
A diferencia de los bind mounts que dependen de la estructura de directorios y del sistema operativo en el que corre docker, los *volumes* son administrados automáticamente por Docker.


<a id="orgb5ef7b3"></a>

## Ventajas

Los volumes tienen varias ventajas:

-   Es más fácil hacer un respaldo.
-   Los volumes se pueden adiministrar con comandos de docker y de una API.
-   Al no depender de un sistema operativo, son más fáciles de usar.
-   Es más fácil compartir volumes entre varios containers.
-   Podemos usar volumes desde servicios en la nube y conectarnos remotamente a estos.
-   Podemos agregar contenido a un volume con un container y luego usar este volume con otro container.


<a id="orgd2f255a"></a>

## Administración de volumes

Un volume es independiente a los containers.
Por lo tanto, no es necesario tener un volume relacionado con un container para poder usar ese volume.

Por ejemplo, podemos crear un volume.

    docker volume create volume-prueba

Una vez creado nuestro volume, podemos listarlo con:

    docker volume ls

Podemos ver información de este volume:

    docker volume inspect volume-prueba

Y podemos borrarlo con:

    docker volume rm volume-prueba


<a id="orge7b72e6"></a>

## Diferencias entre `-v` y `--mount`

Cuando estemos manipulando volume con el CLI de docker, vamos a tener que usar algunas de las opciones `-v` y `--mount`.
Su fin es el mismo, crear volumes, pero su funcionamiento es diferente.

`--mount` es para declarar de manera más explícita o detallada las opciones del volume, bind o tmpfs a crear.
`-v` o `--volume` es más conciso y breve, y sólo es para volumes.

En el caso de `-v` la syntax es:

    <nombre del volume>:<path del archivo o directorio en el container>[:<opciones para el volume>]

Son dos campos obligatorios que están separados por un signo dos puntos (:).

Para usar `--mount` lo hacemos dando un lista de tuplas `<key>=<value>`, donde *key* pueden ser:

-   `type` para decir el tipo del mount, puede ser `bind`, `volume`, `tmpfs`. Aquí estamos discutiendo sobre volumes, así que esta tupla siempre será `type=volume`.
-   `source` o `src` para decir el nombre del volume una vez creado. Si lo dejamos vacío Docker le asigna un ID.
-   `destination`, `dst`, `target` para decir donde se encuentra el archivo o directorio en el container.
-   `readonly` o `ro` para indicar que el volume solo tiene permisos para ser leído.
-   `volume-opt` son las opciones relacionadas con el volume.

Un ejemplo general del uso de `--mount` es:

    docker service create \
        --mount 'type=volume,src=<VOLUME-NAME>,dst=<CONTAINER-PATH>,volume-driver=local,volume-opt=type=nfs,volume-opt=device=<nfs-server>:<nfs-path>,"volume-opt=o=addr=<nfs-address>,vers=4,soft,timeo=180,bg,tcp,rw"'
        --name myservice \
        <IMAGE>


<a id="orge2b269d"></a>

## Iniciar un container con un volume

Docker crea automáticamente los volume necesarios para instalar un container en caso de que el volume no exista.

Como ejemplo:

    docker run -d \
      --name neginxdevtest \
      --mount source=nginx-volume,target=/app \
      nginx:latest

Este comando crea un container y también le agrega un volume que se llama `nginx-volume`.

Podemos ver la información de este container con el siguiente comando:

    docker inspect nginxdevtest

Y buscamos en la sección de *Mounts*.

    "Mounts": [
        {
            "Type": "volume",
            "Name": "nginx-volume",
            "Source": "/var/lib/docker/volumes/nginx-volume/_data",
            "Destination": "/app",
            "Driver": "local",
            "Mode": "",
            "RW": true,
            "Propagation": ""
        }
    ],

Para finalizar para detener el container y borrar el container y el volume:

1.  Listamos los containers

    docker container ls

1.  Detenemos el container

    docker container stop nginxdevtest

1.  Vemos el status del container

    docker container ls

1.  Eliminamos el container

    docker container rm nginxdevtest

1.  Vemos que el container está eliminado:

    docker container ls

1.  Vemos que el volume no se eliminó:

    docker volume ls

1.  Eliminamos el volume:

    docker volume rm nginx-volume

1.  Vemos que se eliminó el container:

    docker volume ls

Podemos volver a crear este container usando `-v`:

    docker run -d \
      --name nginxdevtest \
      -v nginx-volume:/app \
      nginx:latest

Como nota, no podemos borrar un volume que está en uso por un container.


<a id="orgb5f5037"></a>

## Usar volumes en docker-compose

Cómo recuerdan, anteriormente vimos la estructura de un `docker-compose.yaml`.

    version: "3.9"
    services:
      frontend:
        image: node:lts
        volumes:
          - myapp:/home/node/app
    volumes:
      myapp:

En la parte de volumes podemos describir los volumes que queremos crear con el multicontainer.
Cuando hagamos `docker-compose up` estos volumes se crearán.
Si volvemos a crear más multicontainers se va usar el mismo volume.

También podemos crear los volumes por fuera de `docker-compose` y luego referenciarlos.

    version: "3.9"
    services:
      frontend:
        image: node:lts
        volumes:
          - myapp:/home/node/app
    volumes:
      myapp:
        external: true


<a id="org53beb10"></a>

## Popular un volume

Si creamos un container que tiene un volume, los archivos del container se van a copiar a este volume, el volume se va a montar y el container va empezar a usarlo.
Pero este mísmo volume puede ser usado por otros containers.

Para ejemplificar, realicemos el siguiente ejercicio:

1.  Vamos a usar VSCode para editar los archivos por lo que deben instalar la siguiente extensión:
    
    ![img](Volumes/2022-04-19_13-38-53_screenshot.png)

2.  Una vez instalado el plugin, vamos a crear un container con un volume referenciado al container:

    docker run -d -p 8081:80\
      --name=nginxtest \
      --mount source=nginx-vol,destination=/usr/share/nginx/html \
      nginx:latest

Este es un servidor web nginx que incluye una pequeña página en HTML dando la bienvenida.

1.  Vamos a dirección `localhost/8082` y veremos la página de inicio.
    
    ![img](Volumes/2022-04-19_13-47-02_screenshot.png)

2.  Hagamos un pequeño cambio.
    Tenemos que ir a la extensión de Docker dentro de VSCode.
    Dentro de la pestaña **CONTAINERS** vamos a buscar a buscar el container que acabamos de crear.
    
    ![img](Volumes/2022-04-19_13-51-51_screenshot.png)

3.  Dentro de la estructura del container tenemos que ingresar hasta `/usr/share/nginx/html`, donde encontraremos el archivo `index.html` que es la página que estamos viendo, damos click derecho y en "Abrir".
4.  Editamos este archivo, lo guardamos y recargamos la página web para confirmar los cambios.
5.  Ahora, podemos crear otro container usando el mismo volume:

    docker run -d -p 8082:80\
      --name=nginxtest2 \
      --mount source=nginx-vol,destination=/usr/share/nginx/html \
      nginx:lates

1.  Si visitamos la dirección que le dimos a este container veremos que los cambios hechos en el container anterior se mantienen.


<a id="org5cace47"></a>
