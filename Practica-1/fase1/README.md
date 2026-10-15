# Fase 1

## DockerFile

```
FROM httpd:2.4

COPY ./vhosts/00-default.conf /usr/local/apache2/conf/00-default.conf
COPY ./vhosts/marca1-i.conf /usr/local/apache2/conf/marca1-i.conf
COPY ./vhosts/marca2-p.conf /usr/local/apache2/conf/marca2-p.conf

RUN echo "Include conf/00-default.conf" >> /usr/local/apache2/conf/httpd.conf
RUN echo "Include conf/marca1-i.conf" >> /usr/local/apache2/conf/httpd.conf
RUN echo "Include conf/marca2-p.conf" >> /usr/local/apache2/conf/httpd.conf

EXPOSE 80

CMD ["httpd", "-D", "FOREGROUND"]
```

`FROM httpd:2.4`: Define la imagen base de **Apache HTTP Server (versión 2.4)** sobre la que se construirá nuestro contenedor.

`COPY ./vhosts/00-default.conf /usr/local/apache2/conf/00-default.conf`: Copia la regla por defecto desde nuestra máquina local al directorio de configuración interno de Apache dentro del contenedor.

`COPY ./vhosts/marca1-i.conf /usr/local/apache2/conf/marca1-i.conf`: Copia la configuración del Virtual Host de **Marca 1** al contenedor.

`COPY ./vhosts/marca2-p.conf /usr/local/apache2/conf/marca2-p.conf`: Copia la configuración del Virtual Host de **Marca 2** al contenedor.

`RUN echo "Include conf/00-default.conf" >> /usr/local/apache2/conf/httpd.conf`: Añade al final del archivo principal de Apache (`httpd.conf`) la directiva para incluir la regla por defecto.

`RUN echo "Include conf/marca1-i.conf" >> /usr/local/apache2/conf/httpd.conf`: Vincula el archivo de **Marca 1** en el `httpd.conf` principal para que Apache reconozca el dominio `marca1-i.practica`.

`RUN echo "Include conf/marca2-p.conf" >> /usr/local/apache2/conf/httpd.conf`: Vincula el archivo de **Marca 2** en el `httpd.conf` principal para que Apache reconozca el dominio `marca2-p.practica`.

`EXPOSE 80`: El contenedor escuchará peticiones en el puerto HTTP estándar (puerto 80).

`CMD ["httpd", "-D", "FOREGROUND"]`: Comando principal para arrancar el proceso de Apache en primer plano (*foreground*), evitando que el contenedor Docker se apague tras iniciarse.

## Docker-compose.yml

```
version: '3.8'

services:
  servidor_hector_apache:
    build: .
    container_name: contenedor_nimbusdocs
    ports:
      - "8080:80"
    volumes:
      - ./web_marca1_i:/usr/local/apache2/htdocs/marca1
      - ./web_marca2_p:/usr/local/apache2/htdocs/marca2
    restart: always
```

`version: '3.8'`: Especifica la versión del formato del archivo de Docker Compose.

`services:`: Define la lista de contenedores y servicios que forman parte de la aplicación.

`servidor_hector_apache:`: Nombre interno del servicio dentro de la red de Docker Compose.

`build: .`: Indica a Docker Compose que debe construir la imagen utilizando el `Dockerfile` ubicado en el directorio actual (`.`).

`container_name: contenedor_nimbusdocs`: Asigna un nombre fijo e identificable al contenedor en el sistema.

`ports:`: Mapea los puertos entre la máquina anfitriona y el contenedor.

  `"- 8080:80"`: Redirige el puerto `8080` de la máquina local al puerto `80` interno de Apache en el contenedor.

`volumes:`: Mapea carpetas de la máquina local hacia el contenedor para permitir la edición de la web en tiempo real sin reiniciar el servicio.

  `- ./web_marca1_i:/usr/local/apache2/htdocs/marca1`: Monta el contenido del sitio web de Marca 1.

  `- ./web_marca2_p:/usr/local/apache2/htdocs/marca2`: Monta el contenido del sitio web de Marca 2.

`restart: always` -> El contenedor se reiniciará siempre que se detenga (por un error, un reinicio del sistema o incluso si lo detienes tú manualmente con docker stop y luego reinicias el servicio de Docker)

# VHOSTS

## Dominio desconocido (por defecto)

Cuando Apache recibe una peteción, lee la cabecera para saber que web enseñar.

Si lo que hay en la cabecera no coincide exactamente con ninguna de las directivas `ServerName` o `ServerAlias` definidas en los Virtual Hosts (`marca1-i.practica` ni `marca2-p.practica`), **Apache selecciona por defecto el primer Virtual Host que haya sido cargado en memoria** (por ejemplo, tu marca interna marca1-i.practica) exponiendo tu web interna a cualquiera que entre por un dominio no configurado.

Entonces creamos otro vhost, en este caso "00-default.conf"

El archivo 00-default.conf actúa como Virtual Host por defecto porque es la primera directiva Include que se añade al archivo httpd.conf en el Dockerfile.

```
RUN echo "Include conf/00-default.conf" >> /usr/local/apache2/conf/httpd.conf
RUN echo "Include conf/marca1-i.conf" >> /usr/local/apache2/conf/httpd.conf
RUN echo "Include conf/marca2-p.conf" >> /usr/local/apache2/conf/httpd.conf
```

 Apache asigna como sitio por defecto el primer bloque <VirtualHost> que procesa en su configuración principal. El prefijo 00- se utiliza como convención de nombrado.

### 00-default.conf

Uso ***_default_*** por claridad, pero lo que lo convierte en el por defecto es que se carga primero

```
<VirtualHost _default_:80>
    DocumentRoot "/usr/local/apache2/htdocs"

    <Location "/">
        Require all denied
    </Location>

    ErrorLog "logs/default_error.log"
    CustomLog "logs/default_access.log" combined
</VirtualHost>
```

Con la regla  `<Location "/"> Require all denied </Location>` cualquier dominio desconocido que se pida mostrará el error "403 forbidden"

## Archivo - marca1-i.conf y marca2-p.conf

```
<VirtualHost *:80>
    ServerName marca1-i.practica
    ServerAlias www.marca1-i.practica
    DocumentRoot "/usr/local/apache2/htdocs/marca1"

    <Directory "/usr/local/apache2/htdocs/marca1">
        Options -Indexes
        Require all granted
        AllowOverride All
    </Directory>

    ErrorLog "logs/marca1_error.log"
    CustomLog "logs/marca1_access.log" combined
</VirtualHost>
```

```
<VirtualHost *:80>
    ServerName marca2-p.practica
    ServerAlias www.marca2-p.practica
    DocumentRoot "/usr/local/apache2/htdocs/marca2"

    <Directory "/usr/local/apache2/htdocs/marca2">
        Options -Indexes
        Require all granted
    </Directory>

    ErrorLog "logs/marca2_error.log"
    CustomLog "logs/marca2_access.log" combined
</VirtualHost>
```

`DocumentRoot "/usr/local/apache2/htdocs/marca1"` -> Carpeta raíz dentro del contenedor donde están los archivos de la web (index.html).

`<Directory "/usr/local/apache2/htdocs/marca1">` -> Aplica la configuración de permisos a la carpeta del sitio web.

`Options -Indexes` -> **-Indexes** desactiva el listado de archivos si no hay index.html (seguridad)

`Require all granted` -> Permite el acceso público a todo el contenido general de esta web.

`ErrorLog "logs/marca1_error.log"` -> Archivo donde se guardarán los registros de errores de la Marca 1.

`CustomLog "logs/marca1_access.log" combined` -> Archivo donde se registrarán los accesos con éxito/fracaso usando el formato combined.

`AllowOverride All` -> Esto hace que apache lea los archivos del directoria y así lea el archivo .htaccess

## Web_marca1_i - .htaccess

### Archivo ".htaccess"

```
<Files "sensible.txt">
    Require all denied
</Files>
```

### Archivo "sensible.txt"

```
Esto es un archivo sensible simulado
```