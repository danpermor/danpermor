- Para el Access Point utilizaremos DD-WRT, la configuración es sencilla, estos serán los pasos a seguir:

1. Instalaremos DD-WRT en nuestro Router, que hará de validador.
2. Añadimos en freeradius un nuevo cliente, este será el router.

```bash
$  vim /etc/freeradius/3.0/clients.conf

client freeradius-cliente{
        ipaddr=172.16.82.100
        secret=router_radius
$  systemctl restart freeradius
```

3. Una vez en dd-wrt, vamos a services -> Freeradius/Radius
4. Añadimos la configuración de radius

```txt
Radius Auth Server Adress: 172.16.82.20
Radius Auth Server Port: 1812
Password Format: Shared Key
Radius Auth Shared Secret: router_radius
```

5. Finalmente vamos a wirless security y seleccionamos las siguientes opciones, esto puede variar en función del access point:

```bash
Wirless Security

Security Mode: WPA2 RADIUS Only
WPA Algorithms: AES
Radius Auth Shared  Secret: router_radius
Apply
```

6. Ahora tendríamos que poder loggearnos con nuestro usuario y contraseña de LDAP en el Acess Point

- Cabe añadir, que si queremos podríamos hacer DHCP en el mismo accesspoint-router, para no tener que modificar nada.
- Tambié se puede usar con chillispot pero es un poco más complicado así que no lo haremos
