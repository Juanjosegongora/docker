# DOCKER
## JUAN JOSE GONGORA CONTRERAS

# INSTALACION Y CONFIGURACION DE DOCKER EN LINUX

## OPENSUSE
Para instalar docker en opensuse simplemente buscarlo en yast y que nos instale las dependencias

## UBUNTU
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

# ACCESO A LA BASE DE DATOS.

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

# DETENCION Y ELIMINADO DE CONTENEDORES.

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
Esto se hace con `docker-compose` asi que hay que instalarlo, mediante yast para openSUSE o mediante comando para ubuntu.

Una vez instalado para la utilizacion de docker-compose es necesaria la creacion de archivos `yaml`

Los archivos yaml para docker-compose son como un comando de docker pero separados con cada una de las opciones y nos permiten la gestion de redes y de mas contenedores al mismo tiempo, por ejemplo para el ejemplo de mysql de antes en docker compose el contenido del archivo yaml seria algo asi...
```

```