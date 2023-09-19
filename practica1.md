# Logs centralizados

## Introducción 
El trabajo de un administrador de sistemas puede verse claramente simplificando centralizando en un mismo servidor, todos los logs organizandolos y archivandolos en proporción de los servicios que den cada servidor, este ejercicio se basa en el uso del rsyslog para modificar el .conf del demonio **rsyslog** con tal de que los logs sean centralizados en un solo servidor, este ejercicio se va a basar en tcp, pero también se puede pasar en udp por esa razón se abre el puerto udp.

## Desarrollo

- Para el desarrollo de la práctica se basa en la parte del servidor y la parte del cliente, primero, comenzaremos en la parte del servidor.

- ### Servidor

    1. Primero, creamos una regla de firewall abierto en el puerto que queramos para que los logs lleguen a nuestra máquina, en nuestro caso será el 6689.
        ```bash
        $sudo ufw allow 6689/tcp
        $sudo ufw allow 6689/udp
        ```
    2. Comenzamos a editar el archivo /etc/ryslog.conf y añadimos la configuración, descomentaremos las siguientes líneas:
        ```bash
        # provides UDP syslog reception
        module(load="imudp")
        input(type="imudp" port="6689")

        #provides TCP syslog reception
        module(load="imtcp")
        input(type="imtcp" port="6689")
        ```
    3. Haremos una template debajo de la apertura de puertos para redirigir todos los logs a carpetas y así tenerlo todo de forma más ordenada.
        ```bash
            $template RemoteLogs,"/var/log/%HOSTNAME%/%PROGRAMNAME%.log"
            *.* ?RemoteLogs
            & ~

        ``` 
    4. Acto seguido lo reiniciaremos:
        ```bash
            $systemctl restart rsyslog.service
        ```

- ### Cliente
    1. Editaremos el archivo /etc/rsyslog.d/50-default.conf y añadiremos las siguientes líneas, en este caso el 6689 es nuestro puerto, si se usa un solo @ en vez de @@, esto redirigira todo al puerto udp en vez de tcp.:
        ```bash
            $sudo nano /etc/rsyslog.d/50-default.conf
            #  Default rules for rsyslog.
            #For more information see rsyslog.conf(5) and /etc/rsyslog.conf
            *.* @@IP_DEL_SERVIDOR:6689
        ```
    2. Acto seguido lo reiniciaremos:
        ```bash
            $systemctl restart rsyslog.service
        ```



## Finalización 
- Finalmente, en la parte cliente con logger, crearemos un log en el que se pueda reflejar que el servidor acepta los logs de los clientes, se usará de la siguiente forma:
  
    ```bash
            $logger "mimensaje"
    ``` 

- En el servidor, entraremos en el archivo /var/log para saber si se ha guardado la máquina virtual
    ```bash
    $ cat /var/log/jorge-virtualbox/Logger.log |grep prueba
    2023-09-12T14:41:26+02:00 jorge-virtualbox jorge: prueba
    ```