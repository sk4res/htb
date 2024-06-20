## <code style="color : Aquamarine">POV</code>

<style>
    .codenobutton button {
        display: none;
}
</style>

```shell
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