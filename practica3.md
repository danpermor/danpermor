# Listas de control de acceso

## Introducción 
El trabajo de un administrador de sistemas, también tienen la obligación de mantener los archivos con la integridad y autenticación necesaria para poder llevar un sistema seguro, en esta práctica se editarán las listas ACL(acces control list) de forma controlada con tal de en un entorno seguro practicar.

## Desarrollo

### Ejercicio
- La estructura de directorios se basará en la siguiente: 
- Compartido de grupos 
  - Teachers 
  - ESO1
  - ESO2
  - Estudiantes

- Los usuarios y grupos asignados serán
  - Grupos: 
    - Students
    - Users
    - ESO1
      - Matemáticas
    - ESO2
  - Usuarios:
    - S1
    - S2
    - S3 
    - T1
    - T2

### - Condiciones
  - El grupo student estarán todos los estudiantes
  - En el grupo teachers estarán todos los T/Profesores
  - En el grupo ESO1 y ESO2, los estudiantes S1 y S2 estarán unidos a ellos a su respectivo grupo
  - S3 no tiene otro grupo a parte de student
  - A la carpeta ESO1 tendrá acceso: Teachers con permisos rw y eso1 con permiso r
  - A la carpeta eso2 tendrá acceso teachers:rw y eso2:r
  - A la carpeta Teachers tendrá acceso t1:rw y teachers:r
  - A la carpeta students tendrá acceso Teachers:rw Students:r

### Desarrollo

- Crearemos un mkdir para los archivos, al saber que se usa con mkdir, no se pondrá el comando, solo el tree
```bash
Compartido de grupos/
├── eso1
│   └── matematicas
├── eso2
├── students
└── Teachers
```

- Ahora crearemos los grupos y usuarios con el siguiente bash script, le damos permisos de ejecución y lo hacemos correr:
```bash
#!/bin/bash
addgroup teachers
for i in 1 2; do
        adduser s$i
        adduser t$1
        addgroup eso$i
        adgroup t$1 teachers
        adduser s$1 students
done
addgroup s1 eso1
addgroup s2 eso2
adduser s3
```

- Una vez creados los grupos y usuarios comenzamos con los permisos:
  
```bash
$ vim creacionejercicio.sh
#!/bin/bash
setfacl -Rb *
setfacl -Rdm u:t1:rw,g:teachers:r Teachers
setfacl -Rdm g:teachers:rwX,g:eso1:rX eso1
setfacl -Rdm g:teachers:rwX,g:eso2:rX eso2
setfacl -Rdm g:students:rX,g:teachers:rwX students

setfacl -Rm u:t1:rw,g:teachers:r Teachers
setfacl -Rm g:teachers:rwX,g:eso1:rX eso1
setfacl -Rm g:teachers:rwX,g:eso2:rX eso2
setfacl -Rm g:students:rX,g:teachers:rw students
$ :wq
$ chmod +x creacionejercicio.sh
$ sudo bash creacionejercicio.sh
```

- Los permisos quedarían así:

```bash
$ getfacl *
# file: eso1
# owner: root
# group: eso1
user::rwx
group::r-x
group:teachers:rwx
group:eso1:r-x
mask::rwx
other::---
default:user::rwx
default:group::r-x
default:group:teachers:rwx
default:group:eso1:r-x
default:mask::rwx
default:other::---

# file: eso2
# owner: root
# group: eso2
user::rwx
group::r-x
group:teachers:rwx
group:eso2:r-x
mask::rwx
other::---
default:user::rwx
default:group::r-x
default:group:teachers:rwx
default:group:eso2:r-x
default:mask::rwx
default:other::---

# file: students
# owner: root
# group: students
user::rwx
group::r-x
group:teachers:rw-
group:students:r-x
mask::rwx
other::---
default:user::rwx
default:group::r-x
default:group:teachers:rwx
default:group:students:r-x
default:mask::rwx
default:other::---

# file: Teachers
# owner: root
# group: teachers
user::rwx
user:t1:rw-
group::r-x
group:teachers:r--
mask::rwx
other::---
default:user::rwx
default:user:t1:rw-
default:group::r-x
default:group:teachers:r--
default:mask::rwx
default:other::---
```
## Finalización / Verificación

- S1 no puede leer eso2
  
```bash
$ root - echo "Prueba Lectura" > eso2/lectura.md
$ root@daniel-virtualbox:/Compartido de grupos# su s1
$ s1@daniel-virtualbox:/Compartido de grupos$ cd eso2
bash: cd: eso2: Permiso denegado
```

- Heredación de permisos en eso1/matematicas y creacion de carpeta de física
  
```bash
$ll
drwxrwx---+ 4 root eso1 4096 oct  2 13:07 ./
drwxr-xr-x  6 root root 4096 oct  2 13:11 ../
drwxrwxr-x+ 2 root root 4096 oct  2 13:09 matematicas/
$ cd matematicas/
$ pwd
/Compartido de grupos/eso1/matematicas
```

- t2 no escribe en teachers
  
```bash
$ su t2
$ cd /Compartido\ de\ grupos/Teachers/
t2@daniel-virtualbox:/Compartido de grupos/Teachers$ touch hola
touch: no se puede efectuar `touch' sobre 'hola': Permiso denegado
```