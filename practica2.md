# Política de contraseñas

## Introducción
- La seguridad de nuestro sistema puede verse amenazada, una buena política de herramientas puede ayudar en esto.
- El objetivo de la práctica es aprender y asignar las directrices de seguridad, para ello usaremos el módulo pam, más concretamente pw_password.

## Desarrollo
### 1. Instalación 
- Primariamente, para comenzar instalaremos el paquete  "_libpwquality-tools_"
    ```bash
        $ sudo apt install libpwquality-tools
    ```
- Para ver si ha sido correctamente instalado, usaremos el comando pwscore, esto hará que podamos puntuar una contraseña.
    ```bash
        $ pwscore
        daniel1234 #contraseña a probar
        50 #puntuación de la contraseña
    ```
### 2. Configuración/Desarrollo
- Para configurar el módulo PAM de pwconfig, necesitaremos entrar a la configuración de pwconfic, esta se aloja en ***"/etc/security/pwquality.conf"***, esto tiene varías configuraciones.
  - difok: Número de caracteres en una nueva contraseña que no deben estar presentes en la contraseña anterior.
  - minlen: Tamaño mínimo aceptable para la nueva contraseña.
  - dcredit: Crédito máximo por tener dígitos en la nueva contraseña.
  - ucredit: Crédito máximo por tener letras mayúsculas en la nueva contraseña.
  - lcredit: Crédito máximo por tener letras minúsculas en la nueva contraseña.
  - ocredit: Crédito máximo por tener otros caracteres en la nueva contraseña.
  - minclass: Número mínimo de clases de caracteres requeridas para la nueva contraseña
  - maxrepeat: Número máximo de caracteres repetidos.
  - maxclassrepeat: Número máximo de caracteres consecutivos en la misma clase.
  - gecoscheck: Verifica si las palabras individuales de más de 3 caracteres del campo passwd GECOS (campo de comentarios) del usuario están contenidas en la nueva contraseña.
  - dictpath: Ruta a los diccionarios de clacklib.
  - badwords: Lista de palabras separadas por espacios que no deben incluirse en la contraseña

- #### ¿Como funcionan los sistemas de créditos?
  - Los creditos funcionan por la complejidad de la contraseña, es decir, de forma predeterminada, de forma explicada arriba si cambiamos dcredit a 2, cada número contaría como 2 en el pwscore.
  - #### Valores Negativos
  - En caso de poner números negativos, significaría que debe incluir al menos -x carácteres, ejemplo: -1 en dcredit, se necesitará que la contraseña contenga el menos un crédito.

- #### Edición del /etc/security/pwquality.conf
- En mi caso voy a poner las siguientes configuraciones:
    ```bash
        $ vim /etc/security/pwquality.conf
         minlen = 12
        dcredit = 3 #Crédito máximo por dígito en la nueva cuenta
        <esc> :wq #Guardar la configuración
    ```
### 3. Finalización / Verificación

- #### A continuación, veremos si se han aplicado las directivas con el pwscore
  ```bash
    $ pwscore
    hola
    Falló la comprobación de calidad de la contraseña:
    La contraseña tiene menos de 12 caracteres
  ```
  - Como se puede observar, al no cumplir la política establecida de 12 carácteres, vemos que no funciona ni el pwscore.

## Comprobación

- #### Finalmente, comprobaremos unas cuantas configuraciones para ver la complejidad de las contraseñas.

  - Comenzaremos a hacer comprobaciones:
  ```bash
      $ pwscore
      apruebamecarlosporfavorconun10
      100
      $ pwscore
      apruebamecon10 
      31
      $ pwscore
      apruebamecon101
      38
      $ pwscore
      holacaracolaa
      4
  ```

  - Ahora editaremos unos parámetros.
    ```bash
        $ vim /etc/security/pwquality.conf
            dcredit = -2
            ucredit = -3
            lcredit = -1
    ```
  - Y comenzaremos a probar
    ```bash
    $ pwscore
        hola
        Falló la comprobación de calidad de la contraseña:
        La contraseña contiene menos de 2 dígitos
    $ pwscore
        hola12
        Falló la comprobación de calidad de la contraseña:
        La contraseña contiene menos de 3 letras en mayúscula
    $ pwscore
        HOL123
        Falló la comprobación de calidad de la contraseña:
        La contraseña contiene menos de 1 letras en minúscula
    $ pwscore
        HOLa123
        Falló la comprobación de calidad de la contraseña:
        La contraseña tiene menos de 12 caracteres
    $ pwscore
        HOLAa12334278
        31
    ```