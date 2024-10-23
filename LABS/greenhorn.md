# GreenHorn - HTB Easy - Active machine

- Machine is active so hints only

### Enumeration

- Open ports - 22,80,3000 
- Port 80 runs Pluck CMS, exploit out but need creds.
- Gitea running on port 3000

### Initial Foothold

- Gitea - password for PluckCMS admin
- Running Pluck CMS exploit - We have access as www-data

### Lateral movement

- password reuse

### Privilege escalation

- poppler-utils - pdftohtml to extract data from a pdf which contains root password\n
- depix.py -> unpixelate images extracted from pdf
