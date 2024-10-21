HTB Seasonal machine - Chemistry

Machine is still active, dropping hints - 

Initial foothold 

- Port 5000 running Python server with a website, upload and exploit cif file - 
- Create a malicious file using the info in the link below
- !(pymatgen - exploit)[https://github.com/materialsproject/pymatgen/security/advisories/GHSA-vgv8-5cpj-qj2f]

Lateral movement 

- there is a file database.db with a bunch of md5 hashes, check if you can crack them at https://crackstation.net

ROOT

- check localhost:8080 
- aiohttp 3.9.1 exploit
