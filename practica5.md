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

```bash
root@daniel-virtualbox:/home/daniel/Desktop# openssl req -new -key clave_privada_apache.key -out solicitud.csr
You are about to be asked to enter information that will be incorporated
into your certificate request.
What you are about to enter is what is called a Distinguished Name or a DN.
There are quite a few fields but you can leave some blank
For some fields there will be a default value,
If you enter '.', the field will be left blank.
-----
Country Name (2 letter code) [AU]:ES
State or Province Name (full name) [Some-State]:Valencia
Locality Name (eg, city) []:Valencia
Organization Name (eg, company) [Internet Widgits Pty Ltd]:Daniel SL
Organizational Unit Name (eg, section) []:       
Common Name (e.g. server FQDN or YOUR name) []:Daniel
Email Address []:danpermor5@alu.edu.gva.es

Please enter the following 'extra' attributes
to be sent with your certificate request
A challenge password []:daniel
An optional company name []:Daniel2  
root@daniel-virtualbox:/home/daniel/Desktop# openssl x509 -req -days 365 -in solicitud.csr -signkey clave_privada_apache.key -out certificado.crt
Certificate request self-signature ok
subject=C = ES, ST = Valencia, L = Valencia, O = Daniel SL, CN = Daniel, emailAddress = danpermor5@alu.edu.gva.es

```bash
vim /ruta/al/archivo/de/configuración
<VirtualHost *:443>
    ServerName localhost

    SSLEngine on
    SSLCertificateFile /ruta/al/archivo/certificado.crt
    SSLCertificateKeyFile /ruta/al/archivo/clave_privada.key
</VirtualHost>
```
