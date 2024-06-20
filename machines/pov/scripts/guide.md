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

```
=====================================================================
ID           Response   Lines    Word       Chars       Payload
=====================================================================

000000019:   302        1 L      10 W       152 Ch      "dev"                                                               
000009532:   400        6 L      26 W       334 Ch      "#www"                                                              
000010581:   400        6 L      26 W       334 Ch      "#mail"                                                             
000047706:   400        6 L      26 W       334 Ch      "#smtp" 
```
```
```
```
```
```
```
