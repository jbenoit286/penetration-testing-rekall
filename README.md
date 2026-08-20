# penetration-testing-rekall
3-day Red team penetration test achieving full domain compromise via DCSync attack

# Penetration Test Report — Rekall Corporation

## Project Overview

Conducted a structured 3-day red team penetration test against Rekall Corporation's internal and external infrastructure. The assessment covered web application security, Linux and Windows systems, Active Directory, and network-level attacks. Identified 39 vulnerabilities across critical, high, medium, and informational severity levels, achieving full domain compromise via DCSync attack.

---

## Environment

- **Platform:** Kali Linux, Metasploit Framework, Nessus
- **Target:** totalrekall.xyz — Web app, Linux servers, Windows AD environment
- **Network Ranges:** 192.168.13.0/24, 172.22.117.0/24
- **Target Hosts:**
  - 192.168.13.10 — Apache Tomcat server
  - 192.168.13.11 — Shellshock-vulnerable Linux host
  - 192.168.13.12 — Apache Struts / Drupal server
  - 192.168.13.13 — Drupal CMS server
  - 192.168.13.14 — SSH sudo bypass host
  - 172.22.117.10 — Windows Server 2019 (Domain Controller)
  - 172.22.117.20 — Windows 10 workstation

- **Tools Used:** Nmap, Nessus, Metasploit, Meterpreter, Kiwi/Mimikatz, Burp Suite, John the Ripper, cURL, WHOIS/DomainDossier, crt.sh

---

## Vulnerability Summary

| Severity | Count |
|----------|-------|
| Critical | 16 |
| High | 7 |
| Medium | 10 |
| Informational | 6 |

---

## Day 1 — Web Application Testing

### Target: totalrekall.xyz (192.168.14.35)

| # | Vulnerability | Severity | Method |
|---|--------------|----------|--------|
| 1 | XSS Reflected | Medium | `<script>alert("hi")</script>` on welcome.php |
| 2 | XSS Reflected Advanced | Medium | Obfuscated script tag bypass on memory-planner.php |
| 3 | XSS Stored | High | Persistent payload on comments.php |
| 4 | Sensitive Data Exposure | Medium | HTTP response headers exposed via cURL |
| 5 | Local File Inclusion (LFI) | High | PHP file upload and execution on memory-planner.php |
| 6 | LFI Advanced Bypass | High | Double extension bypass `script.jpg.php` |
| 7 | SQL Injection | Critical | `ok' or 1=1--` payload on login.php |
| 8 | Sensitive Data Exposure | Medium | Hardcoded credentials in HTML source (`dougquaid:kuato`) |
| 9 | Sensitive Data Exposure | Informational | robots.txt revealing hidden directories and flag |
| 10 | Command Injection | Critical | OS command execution via networking.php |
| 11 | Command Injection Advanced | Critical | Pipe-based command injection in MX record field |
| 12 | Brute Force Attack | High | Credentials discovered via `/etc/passwd` exposure |
| 13 | PHP Injection | High | `system()` call via URL parameter on souvenirs.php |
| 14 | Session Management | High | Predictable session IDs brute forced with Burp Intruder |
| 15 | Directory Traversal | High | URL manipulation to access old disclaimer files |

---

## Day 2 — Linux Network Penetration

### Target: 192.168.13.0/24

| # | Vulnerability | Severity | CVE / Method |
|---|--------------|----------|--------------|
| 1 | Open Source Exposed Data | Informational | WHOIS lookup via DomainDossier |
| 2 | Open Source Exposed Data | Informational | Ping scan revealing live IP |
| 3 | Open Source Exposed Data | Informational | SSL certificate transparency via crt.sh |
| 4 | Nmap Host Enumeration | Informational | `nmap -sP 192.168.13.0/24` — 6 hosts identified |
| 5 | Nmap Service Detection | Informational | Apache Struts identified on 192.168.13.12 |
| 6 | Apache Struts RCE (Nessus) | Critical | Plugin #97610 — Jakarta Multipart Parser RCE |
| 7 | Apache Tomcat RCE | Critical | CVE-2017-12617 — `tomcat_jsp_upload_bypass` via Metasploit |
| 8 | Shellshock | Critical | `apache_mod_cgi_bash_env_exec` — read `/etc/sudoers` |
| 9 | Unprotected System File | Medium | `cat /etc/passwd` exposing user accounts |
| 10 | Apache Struts RCE | Critical | CVE-2017-5638 — `struts2_content_type_ognl` — extracted flag archive |
| 11 | Drupal RCE | Critical | CVE-2019-6340 — `drupal_restws_unserialize` — Meterpreter shell |
| 12 | Sudo Bypass | Critical | CVE-2019-14287 — SSH into alice@192.168.13.14, `sudo -u#-1` privilege escalation |

---

## Day 3 — Windows Active Directory Compromise

### Target: 172.22.117.0/24

| # | Vulnerability | Severity | Method |
|---|--------------|----------|--------|
| 1 | Credential Disclosure | Medium | Hashed credentials found in GitHub repo — cracked with John the Ripper (`trivera:Tanya4life`) |
| 2 | Exposed Admin Interface | Critical | Nmap scan revealed internal Windows hosts — accessed using recovered credentials |
| 3 | Anonymous FTP Access | Medium | Port 21 open on 172.22.117.20 — anonymous login, downloaded flag3.txt |
| 4 | SLMail Buffer Overflow | Critical | `exploit/windows/pop3/seattlelab_pass` via Metasploit — Meterpreter shell on Win10 |
| 5 | Privilege Escalation via Scheduled Task | Medium | `schtasks` enumeration revealed exploitable task running as SYSTEM |
| 6 | NTLM Credential Dump | Critical | Meterpreter Kiwi module — `lsa_dump_sam` — NTLM hash cracked with John (`Computer!`) |
| 7 | Manual File Discovery | Medium | Meterpreter search function found flag7.txt in public documents |
| 8 | Lateral Movement via PsExec | Critical | Dumped cached credentials with Kiwi — used PsExec to pivot to Server2019 |
| 9 | Manual File Discovery C:\ | Medium | Meterpreter navigation to C:\ root — read flag9.txt |
| 10 | DCSync Attack | Critical | Kiwi `lsa_dump_sam` on Domain Controller — full domain credential dump achieved |

---

## Attack Chain Summary

```
OSINT (WHOIS, crt.sh, GitHub)
        ↓
Credential Discovery (trivera:Tanya4life from public GitHub repo)
        ↓
Network Enumeration (Nmap across 192.168.13.0/24 and 172.22.117.0/24)
        ↓
Linux Exploitation (Apache Struts CVE-2017-5638, Drupal CVE-2019-6340, Shellshock)
        ↓
Windows Initial Access (SLMail buffer overflow — Meterpreter shell)
        ↓
Privilege Escalation (Scheduled task exploitation, sudo bypass CVE-2019-14287)
        ↓
Credential Dumping (Kiwi/Mimikatz — NTLM hashes)
        ↓
Lateral Movement (PsExec to Server2019)
        ↓
Domain Compromise (DCSync attack — full AD credential dump)
```

---

## Key Findings & Recommendations

- **Patch immediately:** Apache Struts CVE-2017-5638, Apache Tomcat CVE-2017-12617, Drupal CVE-2019-6340, Sudo CVE-2019-14287
- **Remove credentials from public repositories** — GitHub exposure led to full network access
- **Disable anonymous FTP** — direct file access with no authentication
- **Implement LSASS protection** — prevent credential dumping via Mimikatz/Kiwi
- **Restrict DCSync permissions** — only Domain Controllers should have replication rights
- **Enforce input validation** across all web applications — SQL injection, XSS, and command injection all present
- **Implement account lockout policies** — brute force attacks succeeded against login endpoints
- **Monitor scheduled tasks** — privilege escalation path via misconfigured task

---

## Skills Demonstrated

- Web application penetration testing (XSS, SQLi, LFI, Command Injection, PHP Injection)
- Linux exploitation (CVE-based attacks, Shellshock, privilege escalation)
- Windows Active Directory attacks (credential dumping, lateral movement, DCSync)
- Metasploit framework — module selection, payload configuration, post-exploitation
- Meterpreter post-exploitation (Kiwi, file discovery, shell spawning)
- Password cracking with John the Ripper (NTLM, md5crypt, mscash2)
- OSINT reconnaissance (WHOIS, crt.sh, certificate transparency logs)
- Nmap network scanning and service enumeration
- Nessus vulnerability scanning and CVE identification
- Professional penetration test report writing
