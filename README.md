# TCM Security CTF @ DEF CON 2026 — Writeups

This repository contains my write-ups and solutions for the TCM Security CTF at DEF CON 2026, played under the alias **krnlbash**.

## 📊 Summary

* **Total Challenges Solved:** 18
* **Categories Covered:** Log Analysis, Network Analysis, Cryptography, Reverse Engineering, OSINT/DNS, Web Reconnaissance, Digital Forensics, Web Exploitation, Git Forensics, Malware Analysis, Password Cracking, Memory Forensics

Each write-up follows the same structure: **Challenge Description → Methodology → Tools Used → Lessons Learned → Flag**, so the reasoning process is as visible as the answer.

## 📁 Navigation

### Log Analysis
- [Anomalous Login Detection](./Log-Analysis/anomalous-login-detection/writeup.md)

### Network Analysis
- [Cleartext Credentials in a PCAP](./Network-Analysis/cleartext-credentials-pcap/writeup.md)
- [Exfiltrated File Recovery](./Network-Analysis/exfiltrated-file-recovery/writeup.md)
- [DNS Exfiltration — Beacon in the Noise](./Network-Analysis/dns-exfiltration-chunks/writeup.md)

### Cryptography
- [ROT13 Decoding](./Cryptography/rot13-decode/writeup.md)
- [Vigenère Cipher with Contextual Key](./Cryptography/vigenere-cipher/writeup.md)

### Reverse Engineering
- [Binary String Extraction](./Reverse-Engineering/binary-string-extraction/writeup.md)

### OSINT / DNS
- [Flag Hidden in a DNS TXT Record](./OSINT-DNS/dns-txt-record-flag/writeup.md)

### OSINT / Web Reconnaissance
- [robots.txt Leaking a Staging Environment](./OSINT-Web/robots-txt-staging-exposure/writeup.md)
- [Recovering a Beta Code via the Wayback Machine](./OSINT-Web/wayback-machine-beta-code/writeup.md)

### Digital Forensics
- [Disk Image File Carving](./Forensics/disk-image-file-carving/writeup.md)
- [PDF Hidden Metadata](./Forensics/pdf-hidden-metadata/writeup.md)

### Web Exploitation
- [Cookie-Based Access Control Bypass](./Web-Exploitation/cookie-based-access-control/writeup.md)
- [JWT Forgery via Leaked Signing Secret](./Web-Exploitation/jwt-secret-disclosure/writeup.md)

### Git Forensics
- [Recovering a Deleted Secret from Git History](./Git-Forensics/deleted-secrets-in-history/writeup.md)

### Malware Analysis
- [Macro Aggression — Deobfuscating a Hidden VBA Payload](./Malware-Analysis/macro-aggression-docx-deobfuscation/writeup.md)

### Password Cracking
- [Cracking a Weak Password from an NTLM Hash Dump](./Password-Cracking/ntlm-hash-dump/writeup.md)

### Memory Forensics
- [Ghost Process Beacon — Identifying a Masquerading Process](./Memory-Forensics/ghost-process-beacon/writeup.md)

## 🛠️ Tools Referenced Across Challenges

`dig` · `curl` · `git` · Python 3 (`scapy`, `pyjwt`, `pikepdf`, `pycryptodome`) · `strings` · `od` · `base64` · `oletools` (`olevba`) · Wayback Machine CDX/Availability APIs

## About Me

I'm **Kernel Bash** (krnlbash), a BS Cybersecurity student at the National Cyber Security Academy (NCSA), Air University, focused on offensive security. I compete in CTFs with team **Air2Shell** and publish additional writeups at [krnlbase.netlify.app](https://krnlbase.netlify.app).

---

*Note: This repository is for educational/portfolio purposes. All challenge instances, target domains, and hosted applications referenced here were purpose-built training environments provided by TCM Security for this CTF and are not live production systems.*
