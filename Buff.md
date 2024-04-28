# Hack The Box
## Buff Writeup (Guided Mode)

### On which TCP port is Buff hosting a website?
> ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/c0eac422-e007-44cf-91d7-404cca15ccbe)
> Port 8080 we have httpd

### Which framework is the website using? 
> ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/52036b39-9f45-4a3f-8f3e-a29b584ecbac)
> Gym Management Software

+ There is an exploit for Gym Manangement Software on Exploit-DB
+ ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/1315d846-72db-48a5-8160-af7ed9600a7d)
+ This exploit works with python2, which has been discontinued, but if you wish to install it here is the link on how to do it - https://ubuntuforums.org/showthread.php?t=2486174
+ Note: Installing python2 might change some configs for python3 so please do not do it unless you know what you are doing. If you're a newbie, create a temporary user for running the exploit.

### Which user is the website running as?
> ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/99f7ddb2-931d-4fc6-97d6-e19a05bc550c)
> Shaun
