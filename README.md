# Escalator (PrivSec)

## Project Overview
Privilege escalation vulns are a big deal—they let attackers climb from low access to full control. This project helps you "think like an attacker" on the Escalator VM, so you can build stronger defenses, run solid pentests, respond faster to incidents, and write killer reports. All for good—protecting systems, never harming them! Dive into walkthroughs for my findings.

## System Setup and Configuration
1. UTM on Apple Silicon Mac (M-series) installed
2. Imported `01-Local1.utm.zip` (SHA1: d4a40ca50044778ddc01a57ac16382e4140000e0).
3. Config: 
* Kali host (4 cores, 4GB RAM, 20GB disk) 
* VM network **Host-Only** (not NAT—enables host scans).

## Exploitation Walkthrough
Check the [detailed version](01/Walkthrough.md) or [short version](01/shortWalk.md).

## Vulnerability Remediation

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

#### Summary

In summary, these vulnerabilities pose severe risks.

Interpreters such as Python, Perl, or Ruby must never receive unrestricted sudo access to root. Arbitrary code execution at root level grants attackers full system control, enabling unauthorized data access, configuration changes, backdoor installation, or data destruction.


## Vulnerability Report E-Mail

This is the link to the [email](/01/email.md)

## Ethical Hacking Report

1. **The Necessity of Proper Authorization**

    Before any security testing begins, obtaining explicit, written authorization from the system owner is the most critical step.  Without this permission, any attempt to probe or exploit a system is considered an unauthorized intrusion, which can lead to severe legal consequences under laws such as the Computer Fraud and Abuse Act (CFAA).  In the context of this project, testing was strictly limited to the provided Local#1 VM environment, which was authorized specifically for educational purposes.

2. **Legal and Ethical Boundaries**

    Ethical hacking requires staying strictly within the "scope" defined by the client or organization.  This means only testing the specific systems and services that have been approved. 
    * Penetration testers must respect these boundaries by:
        - Limiting Testing to Authorized Environments: 
        As emphasized in the sources, these techniques must never be used on unauthorized systems.

        * Avoiding Harmful Actions: Testing should not intentionally disrupt services (Do-S), corrupt data, or compromise the privacy of innocent users.

        * Distinguishing Intent: The goal is to identify weaknesses to improve security posture, not for personal gain, curiosity, or malice.

3. **Responsible Disclosure and Avoiding Harm**

    Reporting vulnerabilities is a professional responsibility that must be handled with care to prevent causing unintended harm to the organization.

    * Direct Reporting: Findings should be reported promptly to the designated security team or through an official bug bounty program using professional templates.
    * Confidentiality: A tester must maintain the confidentiality of their findings, ensuring that exploit details are not leaked to the public before a patch is available, as this could assist malicious actors.
    * Actionable Advice: A responsible report does more than just show a "break-in"; it provides clear remediation steps to help the organization fix the vulnerability and secure their infrastructure.