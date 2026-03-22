# Vulnerability Remediation

1. **Securing File Transfer (Anon FTP)**  
   Vuln: ProFTPD anon login downloads hints/hash (life.c, important.txt).  
   **Mitigation**: Disable anon FTP (`AnonymousEnable no` in proftpd.conf; restart).  
   **Hardening**: No web-dir writes (/files/ chmod 755, FIM like OSSEC for uploads). <br>
   **Risk**: Initial foothold.

2. **Protecting Credentials**  
   Vuln: World-readable hash in `/home/important.txt` (crackable to shrek pass).  
   **Mitigation**: Purge plaintext/hashes from files; use env vars/secret mgmt (Vault).  
   **Best Practice**: chown 600 sensitive files; no www-data reads. <br>
   **Risk**: Lateral pivot.

3. **Least Privilege (Sudoers Privesc)**  
   Vuln: Shrek runs `/usr/bin/python3.5` root NOPASSWD.  
   **Mitigation**: Edit `/etc/sudoers.d/shrek`: drop NOPASSWD or interpreter. Run `visudo -c`.  
   **Hardening**: Audit sudo (`sudo -l` all users), noexec on interpreters. <br>
   **Risk**: Root takeover.

We can also keep track by making a log.

**Detection** : Cron find /home -readable -type f 2>/dev/null | xargs grep -l pass; sudo logs (/var/log/auth.log).

## Summary

In summary, these vulnerabilities pose severe risks.

Interpreters such as Python, Perl, or Ruby must never receive unrestricted sudo access to root. Arbitrary code execution at root level grants attackers full system control, enabling unauthorized data access, configuration changes, backdoor installation, or data destruction.


[← Back to README](../README.md)