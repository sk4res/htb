ping 10.129.108.177
#Linux (TTL -> 64) | Windows (TTL->128)
ping -c 1 10.129.108.177 -R   

nmap -p- --open -sS --min-rate 5000 -vvv -n -Pn 