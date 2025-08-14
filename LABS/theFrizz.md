### HTB Windows Medium

## Enumeration

# Nmap scan

```bash
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)
53/tcp   open  domain        Simple DNS Plus
80/tcp   open  http          Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-title: Did not follow redirect to http://frizzdc.frizz.htb/home/
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-08-14 17:36:54Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb0., Site: Default-First-Site-Name)
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  tcpwrapped
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb0., Site: Default-First-Site-Name)
3269/tcp open  tcpwrapped
Service Info: Hosts: localhost, FRIZZDC; OS: Windows; CPE: cpe:/o:microsoft:windows
```

# Web Enumeration 

Entering the IP in the browser redirects to frizzdc.frizz.htb, browsing around we see a Staff Login button which redirects us to Gibbon-LMS hosted on http://frizzdc.frizz.htb/Gibbon-LMS/

Gibbon-LMS is vulnerable to Arbitraty File Write - CVE-2023-45878

Found a working exploit at - https://github.com/davidzzo23/CVE-2023-45878/blob/main/CVE-2023-45878.py
POC
<img width="1092" height="211" alt="image" src="https://github.com/user-attachments/assets/79f7b8a3-b246-4b51-94b0-bc282e42eaa4" />

# Exploitation and lateral movement

using the -s flag for reverse shell we get code execution

<img width="915" height="291" alt="image" src="https://github.com/user-attachments/assets/8c700346-6953-4feb-bb4c-1dc5732b30c1" />

We find a config.php in the Gibbon-LMS directory, we get DB credentials, lets try to find some hashes

```bash
$databaseServer = 'localhost';
$databaseUsername = 'MrGibbonsDB';
$databasePassword = 'MisterGibbs!Parrot!?1';
$databaseName = 'gibbon';
```
Uploading a ConPtyShell - https://github.com/antonioCoco/ConPtyShell to get a fully interactive reverse shell

Checking the gibbonperson table we find Fiona's password hash

<img width="587" height="88" alt="image" src="https://github.com/user-attachments/assets/6326710f-5e6d-4737-a37c-26cb8b091134" />

After a lot of googling and searching the web for Gibbon-LMS password cracking techniques, I found that it is a sha256 hash with dynamic salt.

Final hash should look something like (Do not copy, i've added *****)- 

```bash
$dynamic_82$067f746faca44f170c6cd9d7c4bdac6bc342c60*****33f80ff784242b0b0c03$/aACFhikmNopqrRTVz2489
```

Let us enumerate further to see what access Fiona has


