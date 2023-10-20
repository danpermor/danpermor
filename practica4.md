# Redundancia de discos

## Introducción

- Un administrador de sistemas, tiene que tener en cuenta el posible fallo y pérdida de los equipos,se utilizará e implementará el sistema RAID para que en caso de corrupción o discos rotos, no se pierda información y nuestro supuesto servidor será operativo 24/7 y redundante, en este caso usaremos mdadm
- La redundancia aporta un nivel más de seguriadd en caso de la pérdida de datos, Esta práctica la llevaremos a cabo implementando un raid 10 que gestionaremos con volumenes lógicos.

## Desarrollo

### 1. Creamos el grupo los 4 volúmenes de 2g

- Comprobamos que estén y sean detectados en la máquina, vemos si están creados el "b","c","d" y "e".

```bash
    $ root@daniel-virtualbox:/home/daniel# fdisk -l |grep -E "/dev/sd?"
        Disco /dev/sda: 25 GiB, 26843545600 bytes, 52428800 sectores
        /dev/sda1   *          2048 52420094 52418047    25G 83 Linux
        Disco /dev/sdb: 2 GiB, 2147483648 bytes, 4194304 sectores
        Disco /dev/sdc: 2 GiB, 2147483648 bytes, 4194304 sectores
        Disco /dev/sdd: 2 GiB, 2147483648 bytes, 4194304 sectores
        Disco /dev/sde: 2 GiB, 2147483648 bytes, 4194304 sectores
```

### 2. Instalamos el mdadm

```bash
    $ sudo apt install mdadm -y
        #...output ommited...
```

### 3. Creamos los discos para crear un ride 10

- Los nombres serán
  - **md0** - Con los discos sdb y sdc
  
  - **md1** - Con los discos sdd y sde

  - **md3** - Raid 0 con la creación de *md0 y md1*

```bash
    mdadm --create /dev/md1 --level=1 --raid-disks 2 /dev/sdb /dev/sdc  # creación de raid1
    mdadm --create /dev/md2 --level=1 --raid-disks 2 /dev/sdd /dev/sde  # creación del otro raid1
    mdadm --create /dev/md3 --level=0 --raid-disks=2 /dev/md1 /dev/md2  # creación del raid 0 (10)
```

### 4. Comprobamos que funciona bién los archivos

```bash
    $ mdadm --detail /dev/mdx # comprobación de que esté creado cada raid
    $ mdadm --detail --scan # comprobación de que funciona bien, vamos a añadirlo a >> etc/mdadm/mdadm.conf
        ARRAY /dev/md1 metadata=1.2 name=daniel-virtualbox:1 UUID=02212486:37e46264:ed41344b:f4227a84
        ARRAY /dev/md2 metadata=1.2 name=daniel-virtualbox:2 UUID=3bc4b572:8f9264c7:19986c9c:2dbcaf05
        ARRAY /dev/md3 metadata=1.2 name=daniel-virtualbox:3 UUID=d842854b:93c82938:d62863b8:d452e35f
    $ mdadm --detail --scan >> /etc/mdadm/mdadm.conf # líneas añadidas al archivo
```

### 5. Creación de fichero en mdadm 3

- Primero montamos el disco duro mdadm3 para poder añadir archivos en ext4

```bash
    $ mkfs.ext4 /dev/md3 # le decimos que tiene label de ext4
    $ mkdir /archivos_raid # creamos la carpeta archivos_raid
    $ mount /dev/md3 /archivos_raid # montamos el disco duro en la carpeta
        /dev/md3 on /archivos_raid type ext4 (rw,relatime,stripe=256)
    $ touch "Prueba" /archivos_raid # añadimos información a la carpeta montada, simplemente de ejemplo
```

- Creamos en /etc/fstab la carpeta para que sea persistente

```bash
$ vim /etc/fstab
    /dev/md3                /archivos_raid  ext4    defaults        0 1 # añadimos para que se quede de forma persistente
```

### Creamos nuevos discos duros de hot spare

- Primariamente los revisamos que estén creados.
```bash
    $ fdisk -l
        Disco /dev/sde: 2 GiB, 2147483648 bytes, 4194304 sectores
        Disk model: VBOX HARDDISK   
        Unidades: sectores de 1 * 512 = 512 bytes
        Tamaño de sector (lógico/físico): 512 bytes / 512 bytes
        Tamaño de E/S (mínimo/óptimo): 512 bytes / 512 bytes


        Disco /dev/sdf: 2 GiB, 2147483648 bytes, 4194304 sectores
        Disk model: VBOX HARDDISK   
        Unidades: sectores de 1 * 512 = 512 bytes
        Tamaño de sector (lógico/físico): 512 bytes / 512 bytes
        Tamaño de E/S (mínimo/óptimo): 512 bytes / 512 bytes
```

- Al tener 2 discos extras vamos a hacer una pequeña comprobación, como vemos que en el disco md1 está sdc y en md2 sde, comprobaremos si al cambiar los discos "corruptos" cambiará la información o no.

1. Comprobamos que no haya cambiado nada
```bash

$ mdadm --detail /dev/md{1,2}
    /dev/md1:

        Number   Major   Minor   RaidDevice State
        0       8       16        0      active sync   /dev/sdb
        1       8       32        1      active sync   /dev/sdc
    /dev/md2:

        Number   Major   Minor   RaidDevice State
        0       8       48        0      active sync   /dev/sdd
        1       8       64        1      active sync   /dev/sde
```

2. Ahora creamos un archivo en la carpeta "archivos_raid" de 500 mb y comprobamos con md5sum

```bash
$ cd /archivos_raid/
$ sudo dd if=/dev/zero of=archivo bs=1M count=500
    500+0 registros leídos
    500+0 registros escritos
    524288000 bytes (524 MB, 500 MiB) copied, 0,273997 s, 1,9 GB/s
$ ls
    archivo
$ md5sum archivo 
    d8b61b2c0025919d5321461045c8226f  archivo
$ md5sum archivo > signature.md5sum 

```













- Los añadimos con virtualBox   
- Una vez creados, ponemos los hot spares

creamos con dd archivos, despues con md5 los miramos, eliminamos el disco metemos el otro y volvemos a mirar el hash
md5sum p.txt > checksum.txt
añadimos  echo "Lo q queramos " >> al archivo dd
md5 -c checksum .txt
volvemos a calcular el checksum con el echo
mdadm /dev/md126 --fail mdadm /dev/sdb --remove
----

lvm 
    - Crear particion d 95% 
    - Redimensionar a 60 40
    - Gui que vea nuestro raid con particions o lsblk
