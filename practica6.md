# Hardening Apache2

## Introducción

- El Hardening de Apache2 es una práctica esencial para reforzar la seguridad del servidor web Apache, minimizando las vulnerabilidades y mitigando amenazas potenciales. Al implementar medidas como la restricción de privilegios, la configuración de cortafuegos y la desactivación de servicios innecesarios, se busca reducir la superficie de ataque y optimizar la resistencia del servidor ante posibles intentos de explotación. Desde la configuración de SSL/TLS para asegurar la comunicación hasta la aplicación de actualizaciones de seguridad, estas prácticas son cruciales para construir un entorno web robusto y protegido contra amenazas cibernéticas.
- En esta práctica se harán ciertas acciones en base al hardening en apache2 vistas en la siguente página web:
  - [Tecmint](https://www.tecmint.com/apache-security-tips/)
  - 1,2,4,6,8,9,11,14,16,17(Dos)

## Desarrollo

### 1. Eliminar la versión de apache2

- Se hará añadiendo en /etc/apache2/apache2.conf las siguientes líneas al final del documento

```bash
$ sudo vim /etc/apache2/apache2.conf
    ServerTokens Prod
    ServerSignature Off
```

### 2. Eliminación de indexación

- Eliminamos la indexación de apache2 para que no se puedan ver archivos ocultos, se tiene que modificar el apache2.conf

```bash
$ vim /etc/apache2/apache2.conf

    <Directory /var/www/html/>
            Options -Indexes 
    </Directory>
$ systemctl restart apache2
```

### 4. Añadir https

- [5. Práctica - Certificado autofirmado para Apache](practica5.md)

### 5. Habilitar HSTS estrictamente

- HSTS es una política que protege las webs de man-in-the-middle. asi que hace que sólo se pueda interactuar con https

```bash
$ sudo a2enmod headers
$ sudo systemctl restar apache2
$ sudo vim /etc/apache2/sites-available/default_ssl.conf
        Header always set Strict-Transport-Security "max-age=31536000; includeSubDomains" # añadimos esta línea

```

### 6. Habilitar HTTP/2 en apache

- Es la siguente versión de HTTP/1.1 que quita algunos errores, da protección y privacidad

```bash
sudo a2enmod http2
systemctl restart apache2
```

### 8. Deshabilitar la directiva de la firma del servidor en apache

- La directiva ServerSignature en el archivo de configuración de Apache agrega un pie de página a los documentos generados por el servidor que contienen información sobre la configuración de su servidor web, como la versión y el sistema operativo en el que se ejecuta, por eso mejor quitarlo en /etc/apache2/apache2.conf, está hecho en el apartado nº1, es exactamente la misma configuración.

### 9. ServerTokens Directiva

- Esta directiva hace que da también información sobre la versión de apache que estamos usando, se quita de la misma manera que el 1. poniendole "prod"

### 11. Deshabilitar módulos innecesarios

- Los módulos son para extender las funcionalidades de apache, sino se usan es mejor desactivarlos. En este caso desactivaremos el de autenticación, ya que no necesitamos ese módulo debido a que no vamos a loggearnos

```bash
$ sudo apache2ctl -M |grep auth
    auth_basic_module (shared) #este módulo 
    authn_core_module (shared)
    authn_file_module (shared)
    authz_core_module (shared)
    authz_host_module (shared)
    authz_user_module (shared)
$ sudo a2dismod auth_basic -f
    Module auth_basic disabled.
    To activate the new configuration, you need to run:
    systemctl sudo restart apache2
$ sudo systemctl restart apache2
```

### 14. Limitar el peso de los archivos al subirlos

- Para más seguridad, está bien que no se puedan subir demasiado grandes que puedan relantizar nuestro servidor, se mide en Bytes ex. 8388608 = 8MB, vamos a "imaginar" que tenemos un directorio para subidas

```bash
$ mkdir -p /var/www/html/uploads
$ vim /etc/apache2/apache2.conf
    <Directory /var/www/html/uploads>
            LimitRequestBody 8388608
    </Directory>
$ :wq
$ systemctl restarta apache2
```

### 16. Apache2 diferente usuario y grupo

- Separar apache2 es una buena práctica de seguridad y se hace de la siguiente forma y manera:

```bash
$ sudo groupadd apachegroup
$ sudo useradd -g apachegroup apacheuser
$ chown -R apacheuser:apachegroup /var/www/html

$ vim /etc/apache2/apache2.conf #añadimos lo siguiente
    User apacheuser
    Group apachegroup

```

### 16 Anti DDos

- La configuración de anti DDos no está muy bien explicada en la página web por lo que instalaremos los siguientes paquetes.
- ModSecurity: Módulo de seguridad de aplicaciones web (WAF) que puedes integrar con Apache. Puedes usar reglas personalizadas para detectar y bloquear patrones de tráfico maliciosos.
  
- Mod_evasive: Módulo para Apache que proporciona protección contra ataques de denegación de servicio, incluidos algunos ataques DDoS. Este módulo evita que una sola IP realice demasiadas solicitudes en un corto período de tiempo.

```bash
sudo apt-get install libapache2-mod-security2 -y
sudo apt-get install libapache2-mod-evasive -y
```

## Comprobación

### 1. 8. y 9

- Antes

```bash
$ curl http://localhost
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>404 Not Found</title>
    </head><body>
    <h1>Not Found</h1>
    <p>The requested URL was not found on this server.</p>
    <hr>
    <address>Apache/2.4.52 (Ubuntu) Server at localhost Port 80</address>
    </body></html>
```

- Después

```bash
$ systemctl restart apache2
$ http://localhost/test
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>404 Not Found</title>
    </head><body>
    <h1>Not Found</h1>
    <p>The requested URL was not found on this server.</p>
    </body></html>
```

### 2

- Antes:

```bash
$curl http://localhost/test/
    <!DOCTYPE HTML PUBLIC "-//W3C//DTD HTML 3.2 Final//EN">
    <html>
    <head>
    <title>Index of /test</title>
    </head>
    <body>
    <h1>Index of /test</h1>
    ...output ommited...
    prueba.py
    hola.txt
```

- Después
-

```bash

$ curl http://localhost/test/
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>403 Forbidden</title>
    </head><body>
    <h1>Forbidden</h1>
    <p>You don't have permission to access this resource.</p>
    </body></html>
```

### 6

- Antes:

```bash
$ curl -k -I --http2 https://localhost
    HTTP/1.1 200 OK
    Date: Fri, 10 Nov 2023 10:57:06 GMT
    Server: Apache
    Strict-Transport-Security: max-age=31536000; includeSubDomains
    Last-Modified: Mon, 30 Oct 2023 11:39:30 GMT
```

- Después:
  
```bash
$ curl -k -I --http2 https://localhost
    HTTP/2 200 
    strict-transport-security: max-age=31536000; includeSubDomains
    last-modified: Mon, 30 Oct 2023 11:39:30 GMT

```

### 14

- Probamos si funciona

```bash
$ dd if=/dev/zero of=prueba bs=1M count=9
    9+0 records in
    9+0 records out
    9437184 bytes (9,4 MB, 9,0 MiB) copied, 0,00507164 s, 1,9 GB/s
$ curl -T prueba http://localhost/uploads/
    <!DOCTYPE HTML PUBLIC "-//IETF//DTD HTML 2.0//EN">
    <html><head>
    <title>413 Request Entity Too Large</title>
    </head><body>
    <h1>Request Entity Too Large</h1>
    The requested resource does not allow request data with PUT requests, or the amount of data provided in
    the request exceeds the capacity limit.
    </body></html>
```

- Como vemos dice que es demasiado largo el archivo.

### 16

- Vemos si funciona apache2 con los nuevos grupos accediendo a la web

```bash
$ curl -I -k  https://localhost
    HTTP/2 200 
    strict-transport-security: max-age=31536000; includeSubDomains
    last-modified: Mon, 30 Oct 2023 11:39:30 GMT
    etag: "29af-608ed7dc56a20"
    accept-ranges: bytes
    content-length: 10671
    vary: Accept-Encoding
    content-type: text/html
    date: Fri, 10 Nov 2023 11:30:26 GMT
    server: Apache
```

- Vemos que tiene longuitud por lo que deja leerlo y nos da una respuesta 200 ok
