Writeup for Sea HTB machine - easy - active

Dropping hints

### Initial Foothold

- Check how to website handles error pages, why are all directory brute force the pages return a 200 instead of a 404
- What else can i discover using this  technique ? are there any installed themes ?
- Once you find the CMS, check for exploits online
- Modify the exploit to fetch the zip file from your machine instead of the github repo
- Modify the exploit to and create the necessary rev.php file

- run the exploit and start a netcat listener, and if you follow the steps perfectly, you should get a reverse shell

### Lateral movement

- What files usually contain credential information on the web root directory ?
- crack the password
- run ```ls /home``` to get users list and spray the password on the machine

### Privilege Escalation

- There is a web service running on port 8080, local port forward
- check for command injection
- Generate a pub-priv key pair - ```ssh-keygen -f root```
- write a public key in /root/.ssh/authorized_keys and ssh into the box as root

- PWNED
