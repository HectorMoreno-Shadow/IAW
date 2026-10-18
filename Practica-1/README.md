# Práctica 1 · Servidores, Proxiesy Certificados

## Dominios ficticios elegidos

### Marca 1: "marca1-i.practica"

- **Justificación del nombre:** Utilizo la extensión ".practica" para reflejar de que es un entorno de pruebas y el nombre "marca1-i" para identificar de que es la web interna.

### Marca 2: "marca2-p.practica"

- **Justificación del nombre:** Utilizo la extensión ".practica" para reflejar de que es un entorno de pruebas y el nombre "marca2-p" para representar la web pública.

## Eleccion de MPM

**MPM seleccionado:** event

**Justificación:** He elegido event ya que crea varios procesos y de cada uno de ellos genera múltiples hilos donde cada hilo atiende a un cliente (cosa que prefork no tiene, con lo cual al tener muchas peticiones simultáneas, el rendimiento cae). Es mucho más ligero que prefork ya que los hilos del mismo proceso comparten memoria y eso hace que consuma menos RAM y soporte más tráfico.

**¿Y por qué no worker?**

Bueno pues porque event tiene una ventaja más, y es que añade para cada proceso un hilo (aparte de los demás hilos) dedicado exclusivamente a escuchar eventos. Este hilo resuelve el problema del Keep-Alive que es cuando un cliente no envia datos, el hilo se queda bloqueado e inactivo esperando que el cliente envie datos, desaprovechando memoria.

Entonces este hilo nuevo que se añade al proceso resuelve esto, cuando el cliente no envia datos, el hilo se libera inmediatamente para atender a otros usuarios, dejando a esa conexión inactiva el hilo *listener*. Esto optimiza a la CPU y a la memoria RAM.

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

`AllowOverride All` -> Esto hace que apache lea los archivos del directorio y así lea el archivo .htaccess

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

## Comprobación con curl

### Marca1-i.practica

`curl -i -H "Host: marca1-i.practica" http://localhost:8080/`

**Resultado:**

```
HTTP/1.1 200 OK
Date: Thu, 15 Oct 2026 10:03:00 GMT
Server: Apache/2.4.69 (Unix)
Last-Modified: Wed, 07 Oct 2026 09:58:33 GMT
ETag: "f1-65d3d28724fa5"
Accept-Ranges: bytes
Content-Length: 241
Content-Type: text/html

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Marca 1</title>
</head>
<body>
    
    <h1>Esta es la web marca 1</h1>

</body>
</html>
```
### Marca2-p.practica

`curl -i -H "Host: marca2-p.practica" http://localhost:8080/`

**Resultado:**

```
HTTP/1.1 200 OK
Date: Thu, 15 Oct 2026 10:02:16 GMT
Server: Apache/2.4.69 (Unix)
Last-Modified: Wed, 07 Oct 2026 09:58:12 GMT
ETag: "f1-65d3d273d55e8"
Accept-Ranges: bytes
Content-Length: 241
Content-Type: text/html

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Marca 2</title>
</head>
<body>

    <h1>Esta es la web marca 2</h1>
    
</body>
</html>
```
### www.marca1-i.practica (prueba del ServerAlias)

`curl -i -H "Host: www.marca1-i.practica" http://localhost:8080/`

**Resultado:**

```
HTTP/1.1 200 OK
Date: Thu, 15 Oct 2026 10:04:39 GMT
Server: Apache/2.4.69 (Unix)
Last-Modified: Wed, 07 Oct 2026 09:58:33 GMT
ETag: "f1-65d3d28724fa5"
Accept-Ranges: bytes
Content-Length: 241
Content-Type: text/html

<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Marca 1</title>
</head>
<body>
    
    <h1>Esta es la web marca 1</h1>

</body>
</html>
```

### Fichero sensible (sensible.txt)

`curl -i -H "Host: marca1-i.practica" http://localhost:8080/sensible.txt`

**Resultado:**

```
HTTP/1.1 403 Forbidden
Date: Thu, 15 Oct 2026 10:08:02 GMT
Server: Apache/2.4.69 (Unix)
Content-Length: 239
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>

```

### Dominio desconocido

`curl -i -H "Host: otro.dominio" http://localhost:8080/`

**Resultado:**

```
HTTP/1.1 403 Forbidden
Date: Thu, 15 Oct 2026 10:09:43 GMT
Server: Apache/2.4.69 (Unix)
Content-Length: 239
Content-Type: text/html; charset=iso-8859-1

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
```

# Fase 2

## Reutilización de la Fase 1

La Fase 2 se construye sobre el Apache de la Fase 1, que pasa a ser el backend detrás de Nginx. Para que cada fase sea autocontenida (se puede arrancar desde cero sin depender de la otra), copié a `fase2/` los ficheros que necesita el Apache:

```
cp -r fase1/vhosts fase1/web_marca1_i fase1/web_marca2_p fase1/Dockerfile fase2/
```

- `Dockerfile` y `vhosts/`: construyen la imagen de Apache con los tres Virtual Hosts, sin cambios respecto a la Fase 1.
- `web_marca1_i/` y `web_marca2_p/`: contenido de las dos marcas, incluido el `.htaccess` y `sensible.txt` de la marca interna.

No copié el `docker-compose.yml` ni el `README.md` de la Fase 1, porque en esta fase el compose cambia.

Además en fase 3 tengo que provocar un fallo del lado de Apache, si lo hago en fase 2, fase 1 sigue intacta como referencia de como funcionaba antes de romperlo.

## Creamos la estructura en fase 2 (dentro de la carpeta fase2)

`mkdir -p nginx/cert_clave nginx/configuracion`

## 1. Generar el certificado

```
openssl req -x509 -newkey ec -pkeyopt ec_paramgen_curve:prime256v1 -nodes \
  -keyout nginx/cert_clave/marca2-p.key \
  -out nginx/cert_clave/marca2-p.crt \
  -days 365 \
  -subj "/CN=marca2-p.practica" \
  -addext "subjectAltName=DNS:marca2-p.practica,DNS:www.marca2-p.practica"
```

- `-newkey ec` con `prime256v1` genera la clave curva.
- `-nodes` hace que la clave no tenga contraseña, para que Nginx arranque sin que nadie la teclee
- `-x509` crea el certificado autofirmado, sin pasar po una CSR.
- El `subjectAltName` es para los navegadores modernos, ya que ignoran el CN y validan solo contra el SAN.

### Verificar el certificado

Para verificar el certificado ponemos esto:

`openssl x509 -in nginx/cert_clave/marca2-p.crt -noout -text | grep -E "Subject:|Not After|Public Key Algorithm|ASN1 OID|NIST CURVE|DNS:"`

**Resultado:**

```
            Not After : Oct 15 11:20:58 2027 GMT
        Subject: CN = marca2-p.practica
            Public Key Algorithm: id-ecPublicKey
                ASN1 OID: prime256v1
                NIST CURVE: P-256
                DNS:marca2-p.practica, DNS:www.marca2-p.practica
```

## 2. Configurar bloque server

### ninbusdocs.conf

```
server {
    listen 443 ssl;
    server_name marca2-p.practica www.marca2-p.practica marca1-i.practica www.marca1-i.practica;

    ssl_certificate     /etc/nginx/certs/marca2-p.crt;
    ssl_certificate_key /etc/nginx/certs/marca2-p.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://servidor_hector_apache:80;
        # SIN proxy_set_header Host todavía
    }
}
```
### docerk-compose.yml

```
services:
  servidor_hector_apache:
    build: .
    container_name: contenedor_nimbusdocs
    volumes:
      - ./web_marca1_i:/usr/local/apache2/htdocs/marca1
      - ./web_marca2_p:/usr/local/apache2/htdocs/marca2
    restart: always

  nginx-proxy:
    image: nginx:stable
    container_name: nginx_nimbusdocs
    ports:
      - "8119:80"
      - "8120:443"
    volumes:
      - ./nginx/configuracion:/etc/nginx/conf.d:ro
      - ./nginx/cert_clave:/etc/nginx/certs:ro
    depends_on:
      - servidor_hector_apache
    restart: always
```

Creamos ***nimbusdocs.conf*** con ***listen 443 ssl***, el certicado y la clave, ***ssl_protocols TLSv1.2 TLSv1.3***,***ssl_ciphers*** solo con suites ***ECDHE-ECDSA*** y ***ssl_prefer_server_ciphers on***. Lo montamos en Nginx con el compose y ***nginx -t*** dio ***syntax is ok***

### 4. Reenviar el host

Al tener en el archivo ***nimbusdocs.conf***:

```
location / {
        proxy_pass http://servidor_hector_apache:80;
        # SIN proxy_set_header Host todavía
    }
}
```

Al hacer curl a ***marca-p.practica***, nos devolcerá un 403

**Salida sin proxy_set_header línea:**

```
HTTP/1.1 403 Forbidden
Server: nginx/1.30.6
Date: Sun, 18 Oct 2026 14:16:08 GMT
Content-Type: text/html; charset=iso-8859-1
Content-Length: 239
Connection: keep-alive

<!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 4.01//EN" "http://www.w3.org/TR/html4/strict.dtd">
<html><head>
<title>403 Forbidden</title>
</head><body>
<h1>Forbidden</h1>
<p>You don't have permission to access this resource.</p>
</body></html>
```

Añadimos el Host

```
server {
    listen 443 ssl;
    server_name marca2-p.practica www.marca2-p.practica marca1-i.practica www.marca1-i.practica;

    ssl_certificate     /etc/nginx/certs/marca2-p.crt;
    ssl_certificate_key /etc/nginx/certs/marca2-p.key;

    ssl_protocols TLSv1.2 TLSv1.3;
    ssl_ciphers ECDHE-ECDSA-AES256-GCM-SHA384:ECDHE-ECDSA-AES128-GCM-SHA256:ECDHE-ECDSA-CHACHA20-POLY1305;
    ssl_prefer_server_ciphers on;

    location / {
        proxy_pass http://servidor_hector_apache:80;
        proxy_set_header Host $host;
    }
}
```










### Estructura resultante

```
fase2/
├── Dockerfile
├── docker-compose.yml
├── vhosts/            (00-default.conf, marca1-i.conf, marca2-p.conf)
├── web_marca1_i/      (.htaccess, index.html, sensible.txt)
├── web_marca2_p/      (index.html)
└── nginx/
    ├── cert_clave/    (marca2-p.crt, marca2-p.key)
    └── configuracion/ (nimbusdocs.conf)
```

# Fase 3

En esta fase tengo que provocar un fallo del lado de Apache. Lo haré sobre un copia en ***fase3/***, de modo que ***fase2/*** siga siendo la referencia de cómo funcionaba antes de romperlo.