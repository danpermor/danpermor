- Pasos a seguir para poder usar el enrutador de forma correcta.
- Configuración de Netplan

```bash
# This is the network config written by 'subiquity'
network:
  ethernets:
      addresses:
      - 192.168.82.223/24
      nameservers:
        addresses:
        - 192.168.82.100
        - 8.8.8.8
        search: []
   routes:
   - to: default
   via: 192.168.10.100
    ens34:
      addresses:
      - 172.16.82.1/24
      nameservers:
        addresses:
        - 172.16.82.1
        search:
        - admin.es
    ens35:
      addresses:
      - 172.16.0.1/24
      nameservers:
        addresses:
        - 172.16.0.1
        search:
        - admin.es
  version: 2
```

# DHCP

- El servicio DHCP y DNS se utilizará mediante dnsmasq, ya que su configuración es sencilla y funcionará en la red interna, las demás, serán estáticas. Es decir, se usará en 172.16.82.0/24

1. Instalar dnsmasq

```bash
apt install dnsmasq
```

2. Modificar el /etc/hosts

```bash
$ vim /etc/hosts
127.0.0.1 localhost
172.16.82.1 enrutador.admin.es enrutador
172.16.82.20 radius.admin.es radius
172.16.82.21 ldap.admin.es squid
172.16.82.22 squid.admin.es squid
```

3. Modificar el /etc/dnsmasq.conf

```bash
interface=ens34
dhcp-range=172.16.82.100,172.16.82.150,255.255.255.0,12h
dhcp-option=option:router,172.16.82.1
dhcp-option=option:dns-server,172.16.82.1
dhcp-option=option:domain-name,admin.es
dhcp-host=00:50:56:2E:A5:21,172.16.82.20,radius #radius
dhcp-host=00:0C:29:15:56:CF,172.16.82.21,ldap #ldap
dhcp-host=00:50:56:29:51:33,172.16.82.22,squid #squid
```

1. Creando nuestro propio servidor dns

```bash
sudo systemctl disable --now systemd-resolved.service 
sudo unlink /etc/resolv.conf sudo vi /etc/resolv.conf
nameserver 8.8.8.8
nameserver 8.8.4.4
nameserver 192.16.0.1 # nuestro dns local
 search admin.es # nuestro dominio
## Reiniciamos dnsmasq
systemctl restart dnsmasq
```

# Firewall - Iptables

- ATENCIÓN, es recomendable, después de instalar todo comenzar a modificar iptables, es mejor simplemente hacer 1 en ip_forwarding y las siguientes opciones:

```bash
iptables -F
iptables -t nat -F
iptables -P FORWARD DROP
iptables -A FORWARD -s 172.16.82.0/24 -j ACCEPT
iptables -t nat -A POSTROUTING -s 172.16.82.0/24 -o ens33 -j MASQUERADE
```

1. Activamos el forwarding en el servidor para comenzar a enrutar paquetes

```bash
echo "1" > /proc/sys/net/ipv4/ip_forwarding
```

2. Nos instalamos iptables_persistent.

```bash
apt install iptables-persistent
```

3. Hacemos que la red 172.16.82.0/24 pueda salir a internet y que se redirijan al servidor squid

```bash
iptables -F
iptables -t nat -F
iptables -P FORWARD DROP
iptables -A FORWARD -s 172.16.82.0/24 -j ACCEPT
iptables -t nat -A POSTROUTING -s 172.16.82.0/24 -o ens33 -j MASQUERADE
```

### Forwarding de APACHE2 + DMZ

```bash
iptables -t nat -A PREROUTING -i ens33 -p tcp --dport 80 -j DNAT --to-destination 172.16.1.100:80
iptables -t nat -A PREROUTING -i ens33 -p tcp --dport 443 -j DNAT --to-destination 172.16.1.100:443
iptables -A FORWARD -i ens33 -o ens35 -p tcp --dport 80 -d 172.16.1.100 -j ACCEPT
iptables -A FORWARD -i ens33 -o ens35 -p tcp --dport 443 -d 172.16.1.100 -j ACCEPT
iptables -A FORWARD -s 172.16.1.0/24 -d 172.16.82.0/24 -j DROP
iptables -A FORWARD -s 172.16.1.0/24 -o ens33 -j ACCEPT
iptables -A FORWARD -d 172.16.1.0/24 -i ens33 -j ACCEPT
```

### Forwarding zona inalámbrica + conexion radius permitida

- Esto haremos que el servidor inalámbrico se pueda autenticar con radius, dejaremos pasar toda la información solo entre estos dos servidores.
- También denegaremos el acceso de la red interna a la red DMZ
- La red interna se puede conectar a internet

```bash
iptables -A FORWARD -s 172.16.0.0/24 -d 172.16.82.0/24 -j DROP
iptables -A FORWARD -s 172.16.0.100/24 -d 172.16.82.20/24 -j ACCEPT
iptables -A FORWARD -s 172.16.0.0.20/24 -d 172.16.82.100/24 -j ACCEPT
iptables -A INPUT -s 172.16.0.0/24 -d 172.16.82.0/24 -j DROP
iptables -t nat -A POSTROUTING -s 172.16.0.0/24 -o ens33 -j MASQUERADE
iptables -A FORWARD -s 172.16.0.0/24 -j ACCEPT
```

### Redirección a SQUID

- Esto hará que todo lo que salga a internet, primero pase por el puerto 172.16.82.0:3128
- Una vez que se active esto SOLO se podrá acceder a través del firewall, entonces está bien saber que si modificamos esto, tendremos que declarar que estamos a través de un proxy en los demás pcs para poder salir a internet.

```bash
# creamos ya la política de reject en iptables, esto hará que todo lo que no ponga que sea forwarding lo eliminará.
iptables -P FORWARDING REJECT
# Red interna a squid
iptables -t nat -A PREROUTING -s 172.16.82.0/24 -p tcp --dport 443 -j DNAT --to-destination 172.16.82.22:3126
iptables -t nat -A PREROUTING -s 172.16.82.0/24 -p tcp --dport 80 -j DNAT --to-destination 172.16.82.22:3126

# wifi a squid
iptables -t nat -A PREROUTING -s 172.16.0.0/24 -p tcp --dport 443 -j DNAT --to-destination 172.16.82.22:3126
iptables -t nat -A PREROUTING -s 172.16.0.0/24 -p tcp --dport 80 -j DNAT --to-destination 172.16.82.22:3126

# Aceptar tráfico a ese puerto, todo lo demás está bloquedo
iptables -A INPUT -s 172.16.0.0/24 -d 172.16.82:22:3126 -j ACCEPT

```
