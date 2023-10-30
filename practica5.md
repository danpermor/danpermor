# Generación de certificado autofirmado para página web apache

## Introduccion
Internet es un canal no seguro, ya que sin unas normas de seguridad preventivas para acceder a este, todas las personas podrían acceder y ver nuestros datos, de esta forma se han creado ciertos mecanismos que hace que las comunicaciones sean más seguras, en este caso se va a desarrollar el cifrado de TLS (Transport Layer Security), equivalente a un certificado, que vamos a autofirmar en un servidor web.

- Desarrollo
  - Generar Certificado
  - Autofirma
  - Instalación
  - Comprobación

## Desarrollo 

1. Instalamos Apache para comenzar la práctica
```bash
$ apt install apache2

```
2. Primero, creamos una llave y luego el certificado también llamado CSR (Certificate Signing Request)
```bash
$ openssl genrsa -out apache2.key.pem 2048
$ openssl req -new -key apache2.key.pem -out solicitud.csr
  You are about to be asked to enter information that will be incorporated
  into your certificate request.
  What you are about to enter is what is called a Distinguished Name or a DN.
  There are quite a few fields but you can leave some blank
  For some fields there will be a default value,
  If you enter '.', the field will be left blank.
  -----
  Country Name (2 letter code) [AU]:ES
  State or Province Name (full name) [Some-State]:España
  Locality Name (eg, city) []:Valencia
  Organization Name (eg, company) [Internet Widgits Pty Ltd]:DPM
  Organizational Unit Name (eg, section) []:DP
  Common Name (e.g. server FQDN or YOUR name) []:Daniel
  Email Address []:danpermor5@alu.edu.gva.es

  Please enter the following 'extra' attributes
  to be sent with your certificate request
  A challenge password []:Daniel123
  An optional company name []:daniel2

```

- Una vez creado el certificado, simplemente tenemos que firmarlo, como no vamos a pagar a una empresa certificadora lo hacemos nosostros mismos aunque el ordenador se quejará.
```bash
$ openssl x509 -req -days 365 -in solicitud.csr -signkey apache2.key.pem -out certificadoapache2.crt
  Certificate request self-signature ok
  subject=C = ES, ST = Espa\C3\83\C2\B1a, L = Valencia, O = DPM, OU = DP, CN = Daniel, emailAddress = danpermor5@alu.edu.gva.es

```


- Cabe destacar que la ruta es insegura y simplemente se ha usado para fines académicos, normalmente los certificados se guardan en /etc/ssl/certs y las claves en /etc/ssl/private/keys.
  - - Finalmente, una vez teniendo todo creado añadiremos lo siguiente a /etc/apache2/sites-avaliable/default-ssl.conf y modificaremos las siguientes líneas, si quisieramos crear otro default-site, sería diferente para otro root directory, habría que crear otro archivo de default-sites...
```bash
$ chmod 777 solicitud.csr
$ chown root:ssl-cert apache2.key.pem 
$ chmod 640 apache2.key.pem
    SSLCertificateFile /etc/apache2/llaves/certificadoapache2.crt
    SSLCertificateKeyFile /etc/apache2/llaves/apache2.key.pem
```

- Reiniciamos apache, activamos el módulo ssl y vemos si todo funciona a la perfección
```bash
$ systemctl restart apache2
$ sudo a2enmod ssl
  Considering dependency setenvif for ssl:
  Module setenvif already enabled
  Considering dependency mime for ssl:
  Module mime already enabled
  Considering dependency socache_shmcb for ssl:
  Enabling module socache_shmcb.
  Enabling module ssl.
```

## Comprobación 


