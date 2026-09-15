## 🧪 Lab 1 — Network & Service Reconnaissance

**Objective:** Build a complete network and service inventory of the Metasploitable 2 target using Nmap and WhatWeb.

**Environment**
- Kali Linux (attacker): `192.168.56.102`
- Metasploitable 2 (target): `192.168.56.101`

**Key activities performed**
- Host discovery (`nmap -sn`) across the `/24` subnet
- Default, version (`-sV`), OS (`-O`), and aggressive (`-A`) scans
- Full TCP port sweep (`-p-`) and UDP scanning (`-sU`)
- Timing template comparison (`-T4`) for scan-speed optimisation
- Safe NSE script enumeration (`-sC`, `http-title`, `http-headers`, `smb-protocols`, `ssh-hostkey`, `banner`)
- Manual HTTP confirmation with `curl -I`
- WhatWeb fingerprinting at aggression levels 1, 3, and 4, including redirect-following and result persistence

**Highlights from findings**
- 30 open TCP ports identified across the full port range, including legacy/high-risk services (FTP, Telnet, rsh/rlogin, NFS, distccd, IRC bind shell on port 1524)
- Anonymous FTP access enabled (vsftpd 2.3.4)
- SMBv1 in use with message signing disabled
- Web stack fingerprinted as Apache/2.2.8 (Ubuntu) with WebDAV and PHP 5.2.4
- OS fingerprinted as Linux kernel 2.6.9–2.6.33

📄 Full command log, screenshots, service inventory table, and question responses: see `Lab_1_Network_Service_Reconnaissance_FINAL.pdf`

---
