## Guided Mode

1. How many TCP ports are open on the remote host?
> 3

```bash
nmap -sC -sV -oN nmap.txt <ip>
```

2. Which CMS and version is the website using?
> Wordpress 5.6.2

```bash
wpscan --url <ip>
```

3. Which PHP version is the website using?
> 8.0.24

```bash
curl -I http://metapress.htb
```

4. Which CVE is the bookingpress plugin being used on the website vulnerable to?
> CVE-2022-0739

```bash
wpscan --url http://metapress.htb/events/ --wp-content-dir ../wp-content
```

5. Which database table contains the password hash for the Wordpress users?
> wp_users

6. What is the password for the Wordpress user manager?
> Get the hash from exploiting the above CVE -> run john/hashcat against the hash and use rockyou wordlist

 7. Which CVE is the Media library of this Wordpress version vulnerable to?
> CVE-2021-29447

8. What is the FTP password for the user metapress.htb?
> This is stored under the file wp_config.php
> This part can be tricky so a hint -> check the nginx default sites enabled file to find the root of the web app and go from there

9. What is the SSH password for the user jnelson on the remote host?
> ***************
> Download the FTP files and store them in a folder
```bash
grep -i password *
```

10. User flag

11. Which password manager software is present on the remote host?
> passpie

12. What is the passphrase for the Passpie password manager?
> look for a file named .keys in the passpie folder
> copy the private key block and save on local machine
> gpg2john <filename> -> copy output hash to a file and run john against that hash with rockyou wordlist

13. What is the password for the user root on the remote host?
> Now we can run passpie commands
``` passpie export /tmp/passwords```
> This should export the root password to the file /tmp/passwords

14 Root Flag
```su root```
