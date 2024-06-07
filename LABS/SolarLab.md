### SolarLab challenge - I need those points

Difficulty:Medium

* __Nmap Scan__ 
  - Discovered unique port 6791 running ReportLab.
  - Discovered SMB shares, interesting file with a lot of PWs -> details-file.xlsx
  - Using HYDRA to check if any username/password combo works on login portal on port -> 6791 ***BINGO Blake boy to the rescue***
* Initial Foothold
  - The website uses ReportLab python module to convert the data into a PDF, it has a RCE vulnerability
  - https://github.com/c53elyas/CVE-2023-33733 -> has a POC on exploiting the vulnerability
  - Using a powershell reverse shell and using the POC from above we can get a shell on the box with user blake
# Lateral movement
  - transferring the shell to a meterpreter shell, trying to get hashes or any creds
  - running winpeas
