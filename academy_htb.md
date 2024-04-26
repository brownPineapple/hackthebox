# Hack The Box

## Academy writeup

### Enumeration

+ Found open ports 22,80,33060
+ checking the website on port 80, redirects to academy.htb
+ Checking the website 
  + We can register and login
  + Creating new user and capturing in burp
  + interesting roleid feild set to 0 when logging in as normal user
  + Changing roleid to 1 sets us as admin
  + logging on /admin.php gives us a second sub domain dev-staging-01.academy.htb

+ Checking dev-staging-01.academy.htb shows a development webpage with some errors
  + Found an API key
  + Server is running Laravel
  + There is a RCE for laravel if we have the API key **Jackpot**
  + CVE-2018-15133

### Initial foothold

+ Checking google for working POC for this CVE and we get a pwn_laravel.py
  + This POC works only if we have the API key which we obtained from the error message on dev-staging-01.academy.htb
  + The exploit works and we get shell as www-data

### Lateral movement

+ We get out of the current directory to go into /var/www/html/academy/ there is a .env file with a password
+ ``` mysql -u <user> -p ``` returns ERROR, we cannot get into mysql using these creds.
+ Checking /home to see multiple users running this box
+ Trying hydra to brute force the password to see if we can ssh into the box using the obtained password
  + **SUCCESS** We can now login as _cry0l1t3_ user
 
+ Read the ***user.txt*** on the machine

### Privelege Escalation

+
