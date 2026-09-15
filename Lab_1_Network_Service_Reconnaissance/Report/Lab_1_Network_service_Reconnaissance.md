# Network & Service Reconnaissance — Nmap, NSE & WhatWeb

A guided, hands-on penetration-testing lab demonstrating end-to-end **network and service reconnaissance** against an intentionally vulnerable target (Metasploitable 2), using **Nmap**, the **Nmap Scripting Engine (NSE)**, **curl**, and **WhatWeb**, in an isolated Kali Linux lab environment.

> ⚠️ **Scope & Authorisation:** All activity documented here was performed exclusively against an instructor-authorised, isolated lab target (Metasploitable 2) on a private host-only network (`192.168.56.0/24`). No public, third-party, or production systems were scanned. This project is for educational demonstration only.

---

## 🎯 Objective

Build a complete, evidence-backed network and service inventory of a target host by progressing through host discovery, port scanning, service/version fingerprinting, OS detection, safe NSE script enumeration, and web-technology fingerprinting — then interpret and document every result.

## 🧰 Environment & Tools

| Component | Detail |
|---|---|
| Attacker VM | Kali Linux — `192.168.56.102` |
| Target VM | Metasploitable 2 — `192.168.56.101` |
| Core tools | `nmap` (incl. NSE), `curl`, `whatweb` |
| Network | Private VirtualBox host-only adapter |

## 🗂️ Repository Structure

```
lab1-report/
├── README.md
└── screenshots/
    ├── 01-ping-connectivity-test.png
    ├── 02-nmap-host-discovery-sn.png
    ├── 03-nmap-service-version-detection-sV.png
    ├── ...
    └── 36-whatweb-verbose-results-saved-to-file-continued.png
```

All screenshots are named sequentially in the order the commands were run, with descriptive filenames (e.g. `08-nmap-aggressive-scan-A.png`) so each image is self-explanatory and easy to trace back to a specific step below.

---

## 🔍 Methodology & Results

### 1. Environment Setup & Connectivity

Identified the attacker (`192.168.56.102`) and target (`192.168.56.101`) IP addresses via `ip addr` / `ifconfig`, then confirmed bidirectional connectivity with a 4-packet ICMP echo test (0% packet loss, avg RTT 2.7 ms).

![Ping connectivity test](screenshots/01-ping-connectivity-test.png)

### 2. Host Discovery

`nmap -sn 192.168.56.0/24` swept the subnet and identified 4 live hosts: the gateway, the DHCP/VirtualBox interface, the Metasploitable 2 target, and the Kali scanner itself.

![Nmap host discovery](screenshots/02-nmap-host-discovery-sn.png)

### 3. Service & Version Detection

`nmap -sV` fingerprinted the software behind every open port, identifying **vsftpd 2.3.4, OpenSSH 4.7p1, Apache httpd 2.2.8, Samba smbd 3.X, and ProFTPD 1.3.1** among 23 open ports (of the default 1,000 scanned).

![Nmap service/version detection](screenshots/03-nmap-service-version-detection-sV.png)

Increasing probe depth with `--version-intensity 9` confirmed every service signature with no discrepancies, at the cost of scan time (11.83s vs. the default).

![Nmap max version intensity](screenshots/04-nmap-version-intensity-9.png)
![Nmap max version intensity (continued)](screenshots/05-nmap-version-intensity-9-continued.png)

### 4. Operating System Fingerprinting

`sudo nmap -O` analysed TCP/IP stack response characteristics and correctly estimated the target as running **Linux kernel 2.6.9–2.6.33**, 1 network hop away.

![Nmap OS detection](screenshots/06-nmap-os-detection-O.png)
![Nmap OS detection (continued)](screenshots/07-nmap-os-detection-O-continued.png)

### 5. Aggressive Scan (OS + Version + NSE + Traceroute)

`sudo nmap -A` combined multiple discovery techniques in one pass, surfacing three critical additional findings not visible in a basic scan:

- **Anonymous FTP login enabled** (FTP code 230)
- **SSH host keys** exposed (DSA 1024-bit / RSA 2048-bit)
- **SMB domain details** — hostname `METASPLOITABLE`, domain `localdomain`

![Nmap aggressive scan](screenshots/08-nmap-aggressive-scan-A.png)
![Nmap aggressive scan (continued 1)](screenshots/09-nmap-aggressive-scan-A-continued-1.png)
![Nmap aggressive scan (continued 2)](screenshots/10-nmap-aggressive-scan-A-continued-2.png)
![Nmap aggressive scan (continued 3)](screenshots/11-nmap-aggressive-scan-A-continued-3.png)
![Nmap aggressive scan (continued 4)](screenshots/12-nmap-aggressive-scan-A-continued-4.png)

### 6. Full-Range TCP Port Scanning

A full sweep of all 65,535 TCP ports (`-p-`) uncovered **7 additional high-order ports** invisible to a default scan: `3632` (distccd), `6697` (ircs-u), `8787` (drb), `34360`/`38057`/`57272` (RPC services), and `48621` (Java RMI) — bringing the total to 30 open TCP ports.

![Nmap full TCP port scan](screenshots/13-nmap-full-tcp-port-scan.png)
![Nmap full TCP port scan (continued)](screenshots/14-nmap-full-tcp-port-scan-continued.png)

Combining the full port range with version detection (`-p- -sV`) produced the definitive service inventory used for final reporting (completed in 130.55s).

![Nmap full TCP scan with versions](screenshots/15-nmap-full-tcp-scan-with-versions.png)
![Nmap full TCP scan with versions (continued)](screenshots/16-nmap-full-tcp-scan-with-versions-continued.png)

### 7. Scan Performance Tuning

Applying the `-T4` timing template to the full-port scan returned the identical 30 open ports in **2.85 seconds** — roughly **46x faster** than the equivalent version-detection scan — demonstrating the trade-off between scan depth and speed.

![Nmap T4 timing template](screenshots/17-nmap-timing-template-T4.png)
![Nmap T4 timing template (continued)](screenshots/18-nmap-timing-template-T4-continued.png)

### 8. Targeted & Range-Based Scanning

Focused scans against a curated port list (`-p 21,22,23,25,80,139,445`) and a numeric range (`-p 1-1024`) demonstrated how scan scope can be narrowed for efficiency once a shortlist of services is known.

![Nmap selected ports scan](screenshots/19-nmap-selected-ports-scan.png)
![Nmap port range scan](screenshots/20-nmap-port-range-scan.png)

### 9. UDP Service Discovery

A full UDP sweep (`-sU`) — significantly slower than TCP due to the lack of a handshake — identified `dns`, `dhcp`, `tftp`, `rpcbind`, `netbios-ns`, `netbios-dgm`, and `nfs`, correctly distinguishing `open` from the ambiguous `open|filtered` state.

![Nmap UDP scan](screenshots/21-nmap-udp-scan.png)

A reduced, faster top-20 UDP scan with version detection validated the same findings in a fraction of the time.

![Nmap top-20 UDP scan with versions](screenshots/22-nmap-top-udp-ports-with-versions.png)

### 10. Safe NSE Script Enumeration

`nmap -sC -sV` ran Nmap's default script set, surfacing high-value findings including anonymous FTP access, plaintext FTP control channel, SMTP/SSLv2 capability data, and SMB security posture (message signing disabled).

![Nmap default NSE scripts](screenshots/23-nmap-nse-default-scripts-sC-sV.png)
![Nmap default NSE scripts (continued 1)](screenshots/24-nmap-nse-default-scripts-continued-1.png)
![Nmap default NSE scripts (continued 2)](screenshots/25-nmap-nse-default-scripts-continued-2.png)

Targeted NSE scripts confirmed specific findings individually:

| Script | Finding |
|---|---|
| `http-title` | Confirmed page title: `Metasploitable2 - Linux` |
| `smb-protocols` | Confirmed obsolete **SMBv1** (NT LM 0.12) in use |
| `ssh-hostkey` | Enumerated DSA & RSA host-key algorithms without authenticating |

![Nmap http-title script](screenshots/26-nmap-http-title-script.png)
![Nmap smb-protocols script](screenshots/27-nmap-smb-protocols-script.png)
![Nmap ssh-hostkey script](screenshots/28-nmap-ssh-hostkey-script.png)

### 11. Manual HTTP Confirmation

Independently verified the web service with `curl -I`, confirming the `Apache/2.2.8 (Ubuntu) DAV/2` server header and discovering links to exposed applications (`/twiki/`, `/phpMyAdmin/`, `/mutillidae/`, `/dvwa/`, `/dav/`) without relying on any automated scanner.

![curl manual HTTP header check](screenshots/29-curl-manual-http-header-check.png)

### 12. WhatWeb Web Technology Fingerprinting

Ran WhatWeb across four aggression levels (1, 3, 4) plus verbose and redirect-following modes. The basic fingerprint (level 1) was already sufficient to identify **Apache 2.2.8, Ubuntu Linux, PHP 5.2.4, and WebDAV 2**; the most aggressive mode (level 4) additionally revealed a hidden **Matomo** analytics installation — showing how deeper fingerprinting can surface technologies invisible at lower aggression.

![WhatWeb basic fingerprint](screenshots/30-whatweb-basic-fingerprint.png)
![WhatWeb basic fingerprint (continued 1)](screenshots/31-whatweb-basic-fingerprint-continued-1.png)
![WhatWeb basic fingerprint (continued 2)](screenshots/32-whatweb-basic-fingerprint-continued-2.png)
![WhatWeb basic fingerprint (continued 3)](screenshots/33-whatweb-basic-fingerprint-continued-3.png)
![WhatWeb basic fingerprint (continued 4)](screenshots/34-whatweb-basic-fingerprint-continued-4.png)

Results were persisted to disk with `whatweb -v ... > whatweb-results.txt` for evidence retention.

![WhatWeb verbose results saved to file](screenshots/35-whatweb-verbose-results-saved-to-file.png)
![WhatWeb verbose results saved to file (continued)](screenshots/36-whatweb-verbose-results-saved-to-file-continued.png)

---

## 📋 Final Service Inventory (Summary)

| Port | Protocol | Service | Version / Evidence |
|---|---|---|---|
| 21 | TCP | FTP | vsftpd 2.3.4 (anonymous login enabled) |
| 22 | TCP | SSH | OpenSSH 4.7p1 Debian 8ubuntu1 |
| 23 | TCP | Telnet | Linux telnetd |
| 25 | TCP | SMTP | Postfix smtpd |
| 53 | TCP | DNS | ISC BIND 9.4.2 |
| 80 | TCP | HTTP | Apache httpd 2.2.8 (Ubuntu) DAV/2 |
| 111 | TCP | RPCbind | rpcbind 2 (RPC #100000) |
| 139 / 445 | TCP | SMB | Samba smbd 3.X–4.X (SMBv1, signing disabled) |
| 512–514 | TCP | rexec / rlogin / rsh | Legacy Netkit r-services |
| 1099 / 48621 | TCP | Java RMI | GNU Classpath grmiregistry |
| 1524 | TCP | Bind shell | **Metasploitable root shell** |
| 2049 | TCP | NFS | NFS 2–4 (RPC #100003) |
| 2121 | TCP | FTP | ProFTPD 1.3.1 |
| 3306 | TCP | MySQL | MySQL 5.0.51a-3ubuntu5 |
| 3632 | TCP | distccd | distccd v1 / GNU 4.2.4 |
| 5432 | TCP | PostgreSQL | PostgreSQL DB 8.3.0–8.3.7 |
| 5900 | TCP | VNC | VNC protocol 3.3 |
| 6667 / 6697 | TCP | IRC | UnrealIRCd |
| 8009 | TCP | AJP13 | Apache Jserv v1.3 |
| 8180 | TCP | HTTP | Apache Tomcat/Coyote JSP engine 1.1 |
| 8787 | TCP | DRb/RMI | Ruby DRb RMI (Ruby 1.8) |
| 34360 / 38057 / 57272 | TCP | RPC (status/nlockmgr/mountd) | RPC #100024 / #100021 / #100005 |
| 53 / 68 / 69 / 111 / 137 / 138 / 2049 | UDP | DNS / DHCP / TFTP / RPC / NetBIOS / NFS | See UDP scan evidence |

**30 open TCP ports** and **7 UDP services** were catalogued in total, several of which (anonymous FTP, plaintext FTP control, SMBv1, an open bind shell on port 1524) represent high-severity misconfigurations characteristic of Metasploitable 2's intentionally vulnerable design.

---

## 💡 Key Skills Demonstrated

- Structuring a reconnaissance workflow from host discovery → port scanning → service fingerprinting → OS detection → script-based enumeration → application fingerprinting
- Interpreting Nmap port states (`open`, `closed`, `filtered`, `open|filtered`) and reasoning about UDP scanning limitations
- Selecting the right scan type for the objective (full-range vs. targeted vs. timing-optimised)
- Using NSE scripts surgically (`http-title`, `smb-protocols`, `ssh-hostkey`, `banner`) to extract specific evidence
- Cross-verifying automated tool output with manual techniques (`curl -I`)
- Multi-tool fingerprinting (Nmap + WhatWeb) and reconciling overlapping evidence
- Producing a clear, defensible security inventory suitable for a professional report

---

## ⚖️ Disclaimer

This project was completed as part of a supervised cybersecurity training course, entirely within an isolated virtual lab. It is shared for portfolio/educational purposes to demonstrate practical reconnaissance methodology. None of the techniques described should be applied to systems without explicit, documented authorisation.

