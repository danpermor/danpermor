- Primero instalamos apache

```bash
sudo apt install apache2
```

- Creación apache sitio seguro, para eso vamos a habilitar y crear los certificados autofirmados, como el habilitar el ssl:

```bash

$ vim /etc/apache2/sites-avaiable/default-ssl.conf

      SSLCertificateFile      /etc/ssl/certs/apachecert.cert

      SSLCertificateKeyFile /etc/ssl/private/apachekey.pem.key

$ openssl genrsa -out ./private/apachekey.key.key.pem 2048

$ openssl req -new -key ./private/apachekey.key.pem -out ./certs/apachecert.cert # certificate signing request

$ openssl x509 -req -days 365 -in ./certs/apachecert.cert -signkey ./private/apachekey.pem.key -out apachecert.cert

$ a2enmod ssl

$ systemctl restart apache2

## hacemos un link simbólico, esto es una buena práctica en seguridad.
$ ln -s ../certs/nombre_de_nuestro_certificado .

$ chown root:ssl-cert apachekey.key

```

- Como meter la información?
 	- Añadiremos los .md a /var/www/html/carpeta_que_queramos, añadir la información al directorio.

```
mv carpeta_que_queramos /var/www/html/
chown -R apacheuser:apachegroup /Var/www/html/carpeta_movida
```
