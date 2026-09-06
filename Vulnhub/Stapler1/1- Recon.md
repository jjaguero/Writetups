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