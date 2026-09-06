# Port 21
```css
Name (192.168.6.134:jjaguero): anonymous
331 Please specify the password.
Password: 
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
550 Permission denied.
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
-rw-r--r--    1 0        0             107 Jun 03  2016 note
226 Directory send OK.
ftp> ls -la
200 PORT command successful. Consider using PASV.
150 Here comes the directory listing.
drwxr-xr-x    2 0        0            4096 Jun 04  2016 .
drwxr-xr-x    2 0        0            4096 Jun 04  2016 ..
-rw-r--r--    1 0        0             107 Jun 03  2016 note
226 Directory send OK.
ftp> get note
local: note remote: note
200 PORT command successful. Consider using PASV.
150 Opening BINARY mode data connection for note (107 bytes).
```

se puede ver que hay una nota en el servicio ftp con el siguiente contenido
```
Elly, make sure you update the payload information. Leave it in your FTP account once your are done, John.
```

Con esta informacion nos dan dos posibles usuarios, John, Elly
# SMB
se puede ver que en el smb aunque no este abierto, se puede ver que uno se puede conectar como usuario invitado vamos a ver que pasa ahi


![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905172207.png)

vemos extrañamente que el disk de kathy tiene un interesante mensaje, que dice que haces aqui vamos a echar un vistazo a eso primero que todo vamos a ver dentro de la carpeta de kathy donde se puede ver lo siguiente

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905172656.png)

continuando checkearemos el contenido de tmp para ver los archivos temporales:
![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905172726.png)

a continuacion con esto vamos a sacar la informacion que tenemos y la exploraremos
```css
smbclient '\\192.168.6.134\tmp' -N -c 'prompt OFF;recurse ON; mget *'
smbclient '\\192.168.6.134\kathy' -N -c 'prompt OFF;recurse ON; mget *'

```
en kathy stuff vemos un archivo txt llamada todo-list.txt con la siguiente informacion:

```css
I'm making sure to backup anything important for Initech, Kathy
```

en la carpeta backup vemos lo siguiente

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905174312.png)

vemos un wordpress que se estrajo esto da acceso a una pagina hecha en wordpress, aun no voy a revisar bien y tambien hay una configuracion de ftp, validando que el usuario anonymous si existe y se puede usar.


# Port 80

el puerto 80 redirige a un http sin nada, es algo raro


# Port 12380

en este puerto encontramos la verdadera pagina de la aplicacion
![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905174959.png)

Con la siguiente informacion, se puede ver un mensaje, esto es una plantilla de bootstrap en la parte de abajo dice  Free Download here que al clickear nos redirije a la pagina de la plantilla

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905175122.png)


veo que no hay nada relevante en la pagina, no encontre nada intente fuzzear nada hasta que cambie el certificado a https y ocurrio lo siguiente:

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905183816.png)

hay una pagina y ya cambio la cosa.
voy a escanear la web con nikto para averiguar si existen directorios:

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905184158.png)


se ve que encontro expuesto un archivo llamado robots.txt y un phpmyadmin, tambien que el dominio se llama red.initech

lo asignamos en el etc/hosts

en robots.txt vemos lo siguiente
```css
User-agent: *
Disallow: /admin112233/
Disallow: /blogblog/

```

dos paginas webs encontrada, vamos a acceder a blogblog ya que admin112233 es un xss

dentro de aca en el apartado de login encontramos un wordpress
![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905184558.png)

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905184607.png)



un panel de login lo que vamos a hacer aca es intentar analizar lo que hay dentro de la pagina wspcan
```css
wpscan --url https://192.168.6.134:12380/blogblog/ -e u --disable-tls-checks

```



```css
[+] URL: https://192.168.6.134:12380/blogblog/ [192.168.6.134]
[+] Started: Sat Sep  5 18:50:47 2026

Interesting Finding(s):

[+] Headers
 | Interesting Entries:
 |  - Server: Apache/2.4.18 (Ubuntu)
 |  - Dave: Soemthing doesn't look right here
 | Found By: Headers (Passive Detection)
 | Confidence: 100%

[+] XML-RPC seems to be enabled: https://192.168.6.134:12380/blogblog/xmlrpc.php
 | Found By: Headers (Passive Detection)
 | Confidence: 100%
 | Confirmed By:
 |  - Link Tag (Passive Detection), 30% confidence
 |  - Direct Access (Aggressive Detection), 100% confidence
 | References:
 |  - http://codex.wordpress.org/XML-RPC_Pingback_API
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_ghost_scanner/
 |  - https://www.rapid7.com/db/modules/auxiliary/dos/http/wordpress_xmlrpc_dos/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_xmlrpc_login/
 |  - https://www.rapid7.com/db/modules/auxiliary/scanner/http/wordpress_pingback_access/

[+] WordPress readme found: https://192.168.6.134:12380/blogblog/readme.html
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Registration is enabled: https://192.168.6.134:12380/blogblog/wp-login.php?action=register
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] Upload directory has listing enabled: https://192.168.6.134:12380/blogblog/wp-content/uploads/
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 100%

[+] The external WP-Cron seems to be enabled: https://192.168.6.134:12380/blogblog/wp-cron.php
 | Found By: Direct Access (Aggressive Detection)
 | Confidence: 60%
 | References:
 |  - https://www.iplocation.net/defend-wordpress-from-ddos
 |  - https://github.com/wpscanteam/wpscan/issues/1299

[+] WordPress version 4.2.1 identified (Insecure, released on 2015-04-27).
 | Found By: Rss Generator (Passive Detection)
 |  - https://192.168.6.134:12380/blogblog/?feed=rss2, <generator>http://wordpress.org/?v=4.2.1</generator>
 |  - https://192.168.6.134:12380/blogblog/?feed=comments-rss2, <generator>http://wordpress.org/?v=4.2.1</generator>

[+] WordPress theme in use: bhost
 | Location: https://192.168.6.134:12380/blogblog/wp-content/themes/bhost/
 | Last Updated: 2026-02-26T00:00:00.000Z
 | Readme: https://192.168.6.134:12380/blogblog/wp-content/themes/bhost/readme.txt
 | [!] The version is out of date, the latest version is 2.3
 | Style URL: https://192.168.6.134:12380/blogblog/wp-content/themes/bhost/style.css?ver=4.2.1
 | Style Name: BHost
 | Description: Bhost is a nice , clean , beautifull, Responsive and modern design free WordPress Theme. This theme ...
 | Author: Masum Billah
 | Author URI: http://getmasum.net/
 |
 | Found By: Css Style In Homepage (Passive Detection)
 |
 | Version: 1.2.9 (80% confidence)
 | Found By: Style (Passive Detection)
 |  - https://192.168.6.134:12380/blogblog/wp-content/themes/bhost/style.css?ver=4.2.1, Match: 'Version: 1.2.9'

[+] Enumerating Users (via Passive and Aggressive Methods)

```
![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905185155.png)

y unas rutas aparte en este caso esta habilitada la ruta de wp-content/uploads esto es importante:

intentare fuerza bruta con el usuario john:

```css
wpscan --url https://192.168.6.134:12380/blogblog --disable-tls-checks -U john -P /usr/share/wordlists/rockyou.txt --password-attack wp-login
```
usando un diccionario llamado rockyou.txt con mas de 14 millones de parametros
despues de dejar un largo rato el wordpress haciendo fuerza bruta encontramos que la contraseña del usuario john es incorrect.

al entrar con el usuario john tenemos un wordpress:

![](obsidian/03_Cybersecurity/Writetups/Vulnhub/Stapler1/images/Pasted%20image%2020260905190924.png)


de todas las opciones tenemos el de instalar un plugin y cargarlo en wp content: