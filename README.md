---
title: "DOCKER, DOCKER-COMPOSE Y DOCKERFILE"
author: [Juan José Góngora Contreras]
subject: "Markdown"
keywords: [Markdown, README]
lang: "en"
toc-own-page: "true"
---
# DOCKER

## QUE ES UN CONTENDOR.

Un contenedor es un proceso que ha sido aislado de todos los demás procesos en la máquina anfitriona. Ese aislamiento aprovecha características de Linux como los namespaces del kernel y cgroups. Aunque es posible tener más de un proceso en un contenedor las buenas prácticas nos recomiendan ejecutar sólo un proceso por contenedor (PID 1).

Algunas ventajas son que podemos tener bastantes contenedores en máquinas puesto que comparten el mismo núcleo el contenedor que la máquina anfitriona, levantar un contenedor es rápido y también nos proporcionan que son portátiles puesto que tenemos la imagen.

## QUE ES UNA IMAGEN DE DOCKER HUB
Una imagen de docker es una instantanea del contenedor, las puedes descargar o tambien se pueden crear mediante un archivo DockerFile

## INSTALACION Y CONFIGURACION DE DOCKER EN LINUX

### OPENSUSE
Para instalar docker en opensuse simplemente buscarlo en yast y que nos instale las dependencias

### UBUNTU
Para instalar docker en ubuntu con el siguiente comando.
```
apt install docker.io
```

## PERMISO PARA USAR DOCKER AL USUARIO PRINCIPAL(IMPORTANTE)
Para hacer posible el uso de docker con un usuario de nuestra maquina simplemente con meterlo al grupo nos basta

IMPORTANTE: RECOMIENDO GESTIONAR LOS CONTENEDORES Y TRABAJAR CON DOCKER DE ESTA FORMA, SIN HACERLO CON ROOT.
```
usermod -G docker tote
```
Docker es el grupo que crea docker por defecto y tote es mi usuario que debeis poner el vuestro.
Hay y reiniciar la maquina, con el servicio deberia de bastar pero mejor maquina completa.

## CREACION DE UN CONTENDOR MYSQL PARA EJEMPLO.
Descargaremos la ultima imagen de mysql con el siguiente comando.
```
docker pull mysql
```

Y se puede visualizar nuestras imagenes.
```
docker image ls
```

Esto seria un contenedor de mysql de ejemplo.

```
docker run -d --name mysql-latest -e MYSQL_ROOT_PASSWORD=root -v /home/tote/.conf:/var/lib/mysql -p 3306:3306 mysql
```
- `-d`
    - Comando que indica que el contenedor se lance como en segundo plano.
- `--name`
    - Con esto le asignamos un nombre al contenedor para en un futuro saber cual es.
- `-e`
    - Por defecto las imagenes pueden tener variables que se pueden usar para algunas cosas, en este caso mysql tiene una para la clave del root `-e` utilizamos ese comando para utililzar una de esas variables y decirle a la imagen que queremos una password personalizada podemos ver que varibales tiene cada imagen en https://hub.docker.com/
- `-v`
    - Comando que nos sirve para indicar que el contenido de una carpeta del contenedor este montada en una de la maquina, util para siempre tener la informacion que necesitamos que no se borre. Como los datos de una base de datos, un ejemplo seria el siguiente... `/home/tote/.conf:/var/lib/mysql` RECOMIENDO CREAR LA CARPETA ANTES
- `-p`
    - Comando que sirve para enlazar los puertos del contenedor con los de la maquina real se pone primero el puerto de la maquina y luego el puerto del contenedor, una cosa asi... `80:80`

- Y por ultimo donde pone mysql es la imagen que se utilizara para la creacion del contenedor, si   ponemos solo mysql cogera la ultima imagen de dockerhub, sino la tienes en maquina la descargara.

Para visualizar los contenedores que tenemos activos en este momento utilizamos el siguiente comando.
```
docker ps
```

Sale algo parecido a esto.
```
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS                               NAMES
b91ce825abd4        mysql               "docker-entrypoint.s…"   8 seconds ago       Up 6 seconds        0.0.0.0:3306->3306/tcp, 33060/tcp   mysql-latest
```

## ACCESO A LA BASE DE DATOS.

Podemos probar la conexion a la base de datos con mysql-workbench por ejemplo, para instalarlo o desde yast o desde comandos en ubbuntu, nombre del paquete `mysql-workbench`

O tambien podemos acceder a la base de datos mediante el contenedor, para acceder al contendor, su interior se hace asi...
```
docker exec -i -t mysql-latest /bin/bash
```

- `-i`
    - Con este comando ordenamos que pasa a ser interactivo ese contenedor.
- `-t`
    - Con este elegimos que el modo sera en forma de terminal(comandos).

- Acto seguido tenemos el nombre del contenedor al que quermos entrar

- Y por ultimo tenemos el interprete de comandos a utilizar, casi todos utilizan este, mysql lo hace, pero recomiendo mirar dockerhub para ver cual es.

De esta forma entramos a un terminal en el contenedor podemos gestionarlo por ejemplo entrando en mysql con el comando.

```
mysql -u root -p
```

Para salir del modo interactivo de docker con poner un simple `exit` nos vale.

## DETENCION Y ELIMINADO DE CONTENEDORES.

Para parar un contendor, pararlo, no eliminarlo, usamos la orden.
```
docker stop mysql-latest
```

De este modo si visualizamos los contendores con `docker ps` ya no nos saldria ese contendor porque no esta activo, pero si queremos ver TODOS los contenedores incluidos los parados usamos `docker ps -a`

Si queremos volver a inciarlo usamos la inversa
```
docker start mysql-latest
```
Y d este modo volveria a estar corriendo.

Para la eliminacion completa de contenedores lo debemos de parar u luego usar la orden...
```
docker rm mysql-latest
```
Si comprobamos con `docker ps -a` vemos que se ha eliminado.

IMPORTANTE: El bind mount que se ha creado al principio,
```
-v /home/tote/.conf:/var/lib/mysql
```
Esos archivos no estan borrados y siguen ahi. De esta forma podemos parar el contenedor eliminarlo y si al cabo del tiempo quieres volver a acceder creamos otra vez el contendor
```
docker run -d --name mysql-latest -e MYSQL_ROOT_PASSWORD=root -v /home/tote/.conf:/var/lib/mysql -p 3306:3306 mysql
```
Y como el bind mount es el mismo los archivos estaran ahi y tendremos el mysql como se quedo.

# DOCKER-COMPOSE

## QUE ES DOCKER-COMPOSE
Docker-compose es lo que nos va a permitir iniciar varios contenedores al mismo tiempo mediante un archivo `docker-compose.yaml` podemos los contenedores que vamos a ejecutar con sus preferencias de esta forma tenemos la ventaja de poder crear redes y que los contenedores esten conectados unos con otros mediante la red de docker.

## INSTALACION DE DOCKER-COMPOSE EN LINUX

### OPENSUSE
Para instalar docker en opensuse simplemente buscarlo en yast y que nos instale las dependencias

### UBUNTU
Para instalar docker-compose en ubuntu con el siguiente comando.
```
apt install docker-compose
```

Una vez instalado para la utilizacion de docker-compose es necesaria la creacion de archivos `yaml`

Los archivos yaml para docker-compose son como un comando de docker pero separados con cada una de las opciones y nos permiten la gestion de redes y de mas contenedores al mismo tiempo, por ejemplo para el ejemplo de mysql de antes en docker compose el contenido del archivo yaml seria algo asi...
```
version: '3'
services:
    mysql-latest:
        image: mysql
        environment:
            - MYSQL_ROOT_PASSWORD=root
        ports:
            - 3306:3306
        volumes:
            - /home/tote/.conf:/var/lib/mysql
```

El fichero yaml tiene que llamarse `docker-compose.yaml` y para ejecutarlo tenemos que usar
```
docker-compose up
```
Esto de forma interactiva, si queremos de forma en la que se ejecute de fondo seria con...
```
docker-compose up -d
```

## CREACION DE UN MYSQL Y UN PHPMYADMIN DE EJEMPLO

Para la creacion de dos contenedores conectados como en este ejemplo podria ser un servidor mysql y un cliente phpmyadmin tendriamos que crear una red virtual de docker, en el archivo yaml se puede hacer todo, para la creacion de lo que seria phpmyadmin el archivo seria el siguiente

```
version: '3'
services:
    phpmyadmin:
        image: phpmyadmin/phpmyadmin
        environment:
            - PMA_HOST=mysql-latest
        ports:
            - 80:80
```

Con la environment `PMA_HOST` le decimos cual sera nuestra base de datos, ponemos ponerle la ip que le asigna docker o mas facilmente, docker tiene instalado como un dns entre sus redes y con el nombre del contenedor SIEMPRE que esten en la misma red de docker.

Pues para ejecutar los dos habria que ponerlos en un docker-compose juntos y tocar algunas cosas, seria algo asi...

```
version: '3'
services:
    phpmyadmin:
        image: phpmyadmin/phpmyadmin
        environment:
            - PMA_HOST=mysql-latest
        ports:
            - 80:80
        networks:
            - default-net
        depends_on: 
            - mysql-latest
    mysql-latest:
        image: mysql
        environment:
            - MYSQL_ROOT_PASSWORD=root
        volumes:
            - /home/tote/.conf:/var/lib/mysql
        networks:
            - default-net
networks:
    default-net:
```

El apartado `networks` es nuevo, aqui lo que estamos diciendo es a que red estara conectado este contendor (Es una red que yo me he inventado su nombre) Si os fijais los dos tienen la misma porque sino no tendrian conexion entre ellos.

El apartado `depends_on` lo que hace simplemente es que ese contenedor depende de que otro se incie primero para la configuracion entonces primero docker-compose inciara mysql porque phpmyadmin depende de el.

Si nos fijamos ahora mysql no tiene un puerto desde fuera, porque no hace falta para phpmyadmin, estan en la misma red interna entonces no hace falta que se le habra un puerto en la maquina real.(Si quisieramos acceder con workbench si tendriamos que ponerlo, pero como tenemos phpmyadmin es un poco inutil tener dos gestores.)

Por ultimo tenemos el apartado `networks` que es un apartado importante, para que las redes que hemos dicho antes funcionen hay que crearlas, y este apartado es el que se encarga de ello, aqui simplemente debemos poner todas las redes que vayamos a utilizar en un archivo docker-compose.yaml

Ya simplemente faltaria lanzar el docker-compose, nos situamos en la carpeta en la que esta y ejecutamos la orden.
```
docker-compose up -d
```
Si nos vamos a un navegador y ponemos localhost, al estar en el puerto 80 nos saltara el phpmyadmin que si ponemos las credenciales del root nos dara acceso a la base de datos.

# DOCKERFILE
