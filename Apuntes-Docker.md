# Apuntes Docker - Héctor Moreno Tejero

## Docker Run

El comando ***docker run*** comprueba primero si tienes la imagen guardada en tu ordenador. Si no la encuentra en tu equipo local, se conecta automáticamente a Docker Hub, la descarga por ti y justo después la arranca.

***"-d"*** -> Sirve para que el contenedor se ejecute en segundo plano sin secuestrar tu terminal. Si lanzas el comando sin “-d” Tu terminal se quedará "bloqueada" mostrando la consola viva del contenedor. Si en esa terminal pulsas Ctrl + C para volver a escribir, apagarás y detendrás el contenedor automáticamente.

***"-p"*** -> Sirve para realizar el mapeo de puertos entre tu ordenador real y el contenedor. EL puerto de tu ordenador puede ser cualquiera, pero el del contenedor debe ser el que el programa escuche.

> *-p [Puerto de tu ordenador] : [Puerto dentro del contenedor]*

> *-p [Puerto libre que quieras] : [Puerto donde escucha el programa dentro]*

***"--name"*** -> Sirve para poner el nombre al contenedor

### Ejemplo:

`docker run -d -p 8080:80 –name hector_contenedor httpd_alpine`

## Docker Logs

***docker logs*** -> Sirve para ver los registros, todos los eventos que ocurren en el contenedor.

***"-f"*** -> Esta opción es para ver en tiempo real esos eventos.

### Ejemplo:

`docker logs -f`

## Docker Build

***docker build*** -> Sirver para construir una imagen

***"-t"*** -> Sirve para darle un nombre y versión a la imagen que estás construyendo.

***"RUTA"*** -> La ruta donde se encuentra el archivo **Dockerfile**

***"."*** -> Indica que busque en la carpeta actual en la que estoy (en este caso debes estar en la carpeta donde tienes el DockerFile)

**Para uso local únicamente (en tu PC):**

> Docker build -t [cualquier_nombre] / [nombre_de_la_imagen]:[etiqueta] RUTA o un "."

**Para subir la imagen a Docker Hub:**

> Docker build -t [usuario_en_dockerhub] / [nombre_de_la_imagen]:[etiqueta] RUTA o un "."

### Ejemplo:

`docker build -t mi-app:v1.0 .`

`docker build -t juanperez/mi-node-api:1.0 .`

## Docker Push

***docker push*** -> Sirve para subir una imagen local desde tu ordenador a un registro remoto en la nube, como Docker Hub.

> docker push [usuario_docker_hub]/[nombre_imagen]:[etiqueta]

### Ejemplo:

`docker push heectoor91/hector-apache-web:v1.0`

## Docker Rmi

***docker rmi*** -> Sirve para eliminar una imagen. Puedes ejecutarlo desde cualquier directorio o carpeta en tu terminal.

### Ejemplo:

`docker rmi heectoor91/hector-apache-web:v1.0`

## Docker rm

***docker rm*** -> Sirver para eliminar un contenedor. Puedes ejecutarlo desde cualquier directorio o carpeta en tu terminal.

***"-f"*** -> Para su eliminación inmediata.

> docker rm [nombre_del_contenedor]

> docker rm [id_del_contenedor]

### Ejemplo:

`docker rm mi-contenedor`

`docker rm 3a1b2c3d4e5f`

## Docker Stop

***docker stop*** -> Sirve para detener el contenedor.

> docker stop nombre_del_contenedor

### Ejemplo:

`docker stop mi-contenedor`

## Docker Exec

***docker exec*** -> Sirve para inspeccionar el interior de un contenedor.

***"-i"*** -> **(Interactive)**: Mantiene la entrada estándar (STDIN) abierta para que puedas escribir comandos.

***"-t"*** -> **(TTY)**: Asigna una terminal virtual para que veas el prompt (como / # o root@...) y los colores en pantalla.

***"sh"*** -> Ejecuta el intérprete de comandos Shell, también puese ser Bash (la consola de Linux).

> docker exec [OPCIONES] NOMBRE_O_ID_DEL_CONTENEDOR COMANDO

### Ejemplo:

`docker exec -it mi-contenedor bash`