1. Instalamos ldap slapd-utils

```bash
apt install ldap slapd-utils
```

2. Reconfiguramos ldap

```bash
dpkg-reconfigure slapd
```

3. Pondremos en nuestro caso lo siguiente

- Contraseña: administrador
- dominio: admin.es

4. Instalamos lam

```bash
apt install ldap-account manager
```

5. Introducimos en el navegador: localhost/lam
6. Arriba a la derecha pinchamos en "configuración de LAM o LAM configuration"
7. Acto seguido clicamos en "editar perfiles del servidor"
8. Se nos abrirá un apartado que pongamos las credenciales de lam, la contraseña es **lam**
9. Una vez dentro, en ajustes generales pondremos en sufijo de arbol, donde pone dc=yourdomain,dc=org, lo modificaremos como dc=admin,dc=es
10. En tipos de cuenta,  modificaremos people y groups para que se asigne automáticamente la unidad organizativa.

- Cabe destacar, en lista de usuarios válidos pondremos cn=admin,dc=admin,dc=es

11. Finalmente,guardamos
12. Creamos un grupo en el apartado grupo llamado Usuarios_sistema y invitad
13. Entraremos y crearemos 4 usuarios en el apartado usuarios.

- marc:marc
- daniel:daniel
- carlos:carlos
- invitado:invitado -> Cuenta de invitado para navegar, a este habrá que decir que su grupo es invitado

- Cabe destacar que también se podría añadir usuarios en ldap de forma en comandos, se va a hacer una pequeña demostración para que cuente más en la nota final:

1. Creamos un archivo.ldif despues de haber configurado el dominio

```bash
dn: ou=People,dc=admin,dc=es
objectClass: organizationalUnit
ou: People

dn: ou=Groups,dc=admin,dc=es
objectClass: organizationalUnit
ou: Groups

dn: uid=marc,ou=People,dc=admin,dc=es
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: marc
uidNumber:10000
gidNumber:50000
sn: monar
givenName: marc
cn: marc monar
userPassword: 1234
homeDirectory: /home/marc

dn: uid=carlos,ou=People,dc=admin,dc=es
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: carlos
uidNumber:10001
gidNumber:50001
sn: martorell
givenName: carlos
cn: carlos martorell
userPassword: 1234
homeDirectory: /home/carlos


dn: uid=daniel,ou=People,dc=admin,dc=es
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: daniel
uidNumber:10002
gidNumber:50002
sn: perez
givenName: daniel
cn: daniel perez
userPassword: 1234
homeDirectory: /home/daniel


dn: uid=invitado,ou=People,dc=admin,dc=es
objectClass: inetOrgPerson
objectClass: posixAccount
objectClass: shadowAccount
uid: invitado
uidNumber:10003
gidNumber:50003
sn: invi
givenName: invitado
cn: invitadoinvi
userPassword: 1234
homeDirectory: /home/invitado
```

- Y los añadiremos al ldap.

```bash
ldapadd -x -H ldap:/// -b "dc=admin,dc=es" -D "cn=admin,dc=admin,dc=es" -w administrador -f base.lidf 
```

- Con esto tendriamos ldap acabado
