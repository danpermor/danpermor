1. Instalamos freeradius y las dependencias

```bash
apt install freeradius freeradius-ldap
```

2. Vamos al directorio de freeradius y modificamos el archivo ldap.

```bash
$  cd /etc/freeradius/3.0/mods-available
$  cp ldap ldap.original
$  vim ldap

server = 'ldap.admin.es'
identity = 'cn=admin,dc=admin,dc=es' # hay que descomentarlo
password = administrador #hay que descomentarlo
base_dn = 'dc=admin,dc=es'
```

3. Ahora vamos a mods-enabled y hacemos un link simbólico

```bash
cd ../mods-enabled

$  ln -s ../mods-available/ldap ldap
$  ls -l ldap
 lrwxrwxrwx 1 root root 22 feb 21 19:01 ldap -> ../mods-available/ldap
```

4. Finalmente vamos a inner-tunnel en sites-enabled y modificamos lo siguiente:

```bash
$  cd /etc/freeradius/3.0/sites-avaiable/
$  cp inner-tunnel inner-tunnel.original

$  cd /etc/freeradius/3.0/sites-avaiable/
$  cp default default.original

$  vim default
  
  # Uncomment it if you want to use ldap for authentication
        #
        # Note that this means "check plain-text password against
        # the ldap database", which means that EAP won't work,
        # as it does not supply a plain-text password.
        #
        #  We do NOT recommend using this.  LDAP servers are databases.
        #  They are NOT authentication servers.  FreeRADIUS is an
        #  authentication server, and knows what to do with authentication.
        #  LDAP servers do not.
        #
        Auth-Type LDAP {
                ldap
            }
$  cd ../sites-enabled
$  ln -s ../sites-avaiable/inner-tunnel inner-tunnel
- Hay que descomentar el apartado de auth-type LDAP
```

- Finalmente tenemos nuestro servidor instalado de freeradius, vamos a hacer algunas pruebas.
- Añadimos un cliente en clients.conf para hacer pruebas, en este caso voy a usar el de ldap y añadimos la siguiente configuración

```bash
$  vim /etc/freeradius/3.0/clients.conf
client ldap-sv{
        ipaddr=172.16.82.21
        secret=prueba123
}
$  systemctl stop freeradius
$  freeradius -X # el modo debug de freeradius
```

- Ahora iremos al servidor ldap a hacer pruebas

```bash
$  apt install freeradius-utils
$  radtest marc marc 172.16.82.20 1812 prueba123 -t ldap
Sent Access-Request Id 222 from 0.0.0.0:57766 to 172.16.82.20:1812 length 80
        User-Name = "marc"
        User-Password = "marc"
        NAS-IP-Address = 172.16.82.21
        NAS-Port = 1812
        Message-Authenticator = 0x00
        Framed-Protocol = PPP
        Cleartext-Password = "marc"
Received Access-Accept Id 222 from 172.16.82.20:1812 to 172.16.82.21:57766 length 32
        Framed-Protocol = PPP
        Framed-Compression = Van-Jacobson-TCP-IP

```

- Y con esto sabemos que funciona correctamente.
