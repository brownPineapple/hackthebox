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
wpscan --url http://metapress.htb/events/
```

