# Redundancia de discos

## Introducción
- Un administrador de sistemas, tiene que tener en cuenta el posible fallo y pérdida de los equipos, para este caso se utilizará e implementará el sistema RAID para que en caso de corrupción o discos rotos, no se pierda información y nuestro supuesto servidor será operativo 24/7 y redundante, en este caso usaremos mdadm

## Desarrollo
### 1. Creamos el grupo los 4 volúmenes de 2g. 
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
### 3. Creamos los discos para crear un ride 10. 
- Los nombres serán
    - **md0** - Con los discos sdb y sdc
    - **md1** - Con los discos sdd y sde
    - **md3** - Raid 0 con la creación de *md0 y md1*
```bash
    $ mdadm --create /dev/md1 --level=1 --raid-disks 2 /dev/sdb /dev/sdc --metadata=0.90 # creación de raid1
    $ mdadm --create /dev/md2 --level=1 --raid-disks 2 /dev/sdd /dev/sde --metadata=0.90 # creación del otro raid1
    $ mdadm --create /dev/md3 --level=0 --raid-disks=2 /dev/md1 /dev/md2 #creación del raid 0 (10)
```
### 4. Comprobamos que funciona bién los archivos
```bash
    $ mdadm --detail /dev/mdx # comprobación de que esté creado cada raid
    $ mdadm --detail --scan # comprobación de que funciona bien, vamos a añadirlo a >> etc/mdadm/mdadm.conf
        ARRAY /dev/md1 metadata=0.90 UUID=6f18c268:ba73acac:c37d8dea:c4e7c855
        ARRAY /dev/md2 metadata=0.90 UUID=d370b869:64454064:c37d8dea:c4e7c855
        ARRAY /dev/md3 metadata=1.2 name=daniel-virtualbox:3 UUID=9ae66fc0:d7e027e4:3c713da2:8f8b182e
    $ mdadm --detail --scan >> /etc/mdadm/mdadm.conf # líneas añadidas al archivo
```

### 5. Creación de fichero en mdadm 3
- Primero montamos el disco duro mdadm3 para poder añadir archivos en ext4
```bash
    $ mkfs.ext4 /dev/md3 # le decimos que tiene label de ext4
    $ mkdir /archivos_raid # creamos la carpeta archivos_raid
    $ mount /dev/md3 /archivos_raid # montamos el disco duro en la carpeta
    $ touch "Prueba" /archivos_raid # añadimos información a la carpeta montada, simplemente de ejemplo
```
- Creamos en /etc/fstab la carpeta para que sea persistente
```bash
$ vim /etc/fstab
    /dev/md3                /archivos_raid  ext4    defaults        0 1 # añadimos para que se quede de forma persistente
```
### Creamos nuevos discos duros de hot spare

- Los añadimos con virtualBox
- Una vez creados, ponemos los hot spares



---- 
creamos con dd archivos, despues con md5 los miramos, eliminamos el disco metemos el otro y volvemos a mirar el hash
md5sum p.txt > checksum.txt
añadimos  echo "Lo q queramos " >> al archivo dd
md5 -c checksum .txt
volvemos a calcular el checksum con el echo
mdadm /dev/md126 --fail mdadm /dev/sdb --remove
----