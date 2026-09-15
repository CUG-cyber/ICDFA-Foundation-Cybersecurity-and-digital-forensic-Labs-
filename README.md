# ICDFA Cybersecurity Labs — Network Reconnaissance, Web Application Security Testing & Bash Automation

This repository documents three hands-on labs completed as part of the ICDFA (Introduction to Cyber Defence & Forensic Analysis) practical coursework. All labs were performed exclusively against instructor-authorised, intentionally vulnerable lab targets (Metasploitable 2, DVWA, Mutillidae) within an isolated VirtualBox lab environment (Kali Linux attacker VM + Metasploitable 2 target VM).

> ⚠️ **Authorisation Notice:** All scanning, exploitation, and automation activity documented here was carried out strictly against instructor-provided, intentionally vulnerable lab machines on a private/host-only network (`192.168.56.0/24`). None of the techniques, tools, or scripts in this repository should be used against systems you do not own or do not have explicit written authorisation to test.

---

## 📁 Repository Contents

| File | Description |
|---|---|
| `Lab_1_Network_Service_Reconnaissance_FINAL.pdf` | Nmap fundamentals, service enumeration, NSE scripting, and WhatWeb fingerprinting against Metasploitable 2 |
| `Lab_2_Web_Application_Security_Testing_FINAL.pdf` | DVWA file-upload testing, Burp Suite traffic analysis, Nikto, DIRB, and manual/automated SQL injection against DVWA and Mutillidae |
| `Lab_3_Bash_Reconnaissance_Automation_FINAL.pdf` | Custom Bash reconnaissance automation tool (`recon_tool.sh`) wrapping WhatWeb, Nmap, and DIRB |
| `recon_tool.sh` | Executable Bash script deliverable from Lab 3 |

---

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

## 🌐 Lab 2 — Web Application Security Testing

**Objective:** Move from network reconnaissance to application-layer testing against DVWA and Mutillidae, focusing on request/response inspection, file-upload validation weaknesses, and SQL injection.

**Tools used:** Browser DevTools, Burp Suite Community Edition, Nikto, DIRB, manual SQL injection, SQLMap (metadata enumeration only)

**Key activities performed**
- Inspected raw HTTP requests/responses and session cookies via DevTools and Burp
- Tested DVWA's file-upload feature with harmless files, comparing extension handling vs. client-supplied `Content-Type` spoofing
- Confirmed uploaded files are stored in a web-accessible, executable directory
- Ran Nikto against the target and analysed 5 key configuration findings (version disclosure, `phpinfo.php` exposure, HTTP TRACE/XST risk, directory indexing, exposed phpMyAdmin)
- Ran DIRB for content/path discovery and correlated results with the upload-storage risk
- Mapped user-controlled input parameters in Mutillidae
- Performed manual SQL injection testing (`'`, `' OR '1'='1`, `' AND '1'='2`) and explained the resulting query logic
- Verified findings using SQLMap, restricted strictly to database/table metadata enumeration (no data exfiltration or modification)
- Documented a defensive-controls table mapping each weak practice to its recommended fix

**Highlights from findings**
- File upload accepts spoofed `Content-Type` headers, enabling MIME-based bypass of client-side validation
- Boolean-based SQL injection confirmed manually in Mutillidae's login form and cross-verified with SQLMap (MySQL back end, 7 databases enumerated)
- Multiple unauthenticated, web-accessible administrative and diagnostic endpoints identified (`/phpMyAdmin/`, `/phpinfo.php`, `/dav/`, `/twiki/`)

📄 Full evidence, request/response tables, Nikto/DIRB output, and defensive recommendations: see `Lab_2_Web_Application_Security_Testing_FINAL.pdf`

---

## ⚙️ Lab 3 — Bash Reconnaissance Automation

**Objective:** Build a menu-driven Bash tool that automates the WhatWeb, Nmap, and DIRB workflows from Labs 1–2 against a user-supplied (non-hardcoded) target.

**Script:** [`recon_tool.sh`](./recon_tool.sh)

**Features**
- Prompts for an authorised target IP/domain (no hardcoded target)
- Input validation (rejects empty input and targets containing whitespace)
- Interactive menu (`case` statement) offering WhatWeb, Nmap, or DIRB
- `check_tool()` function verifies each required binary is installed before running
- Optional `tee`-based output logging to a `results/` directory
- Executable via `chmod +x` and `./recon_tool.sh`

**Usage**
```bash
chmod +x recon_tool.sh
./recon_tool.sh
```

You will be prompted for a target, then presented with:
```
Select a reconnaissance tool:
1) WhatWeb
2) Nmap
3) DIRB
4) Exit
```

**Verified successful execution against:**
- ✅ WhatWeb (Option 1) — returned full technology fingerprint
- ✅ Nmap `-sV` (Option 2) — returned full service/version inventory
- ✅ DIRB (Option 3) — returned discovered directories and status codes

📄 Full staged build-out, explanations, and Q&A: see `Lab_3_Bash_Reconnaissance_Automation_FINAL.pdf`

---

## 🛠️ Tools & Technologies

`Nmap` · `NSE` · `WhatWeb` · `Burp Suite` · `Nikto` · `DIRB` · `SQLMap` · `Bash` · `Kali Linux` · `Metasploitable 2` · `DVWA` · `Mutillidae`

---

## 📚 Key Takeaways

- Layered reconnaissance (network → service → application) builds a progressively deeper picture of an attack surface.
- Client-supplied metadata (file extensions, `Content-Type` headers) must never be trusted for server-side validation.
- Verbose error messages and default diagnostic scripts (e.g. `phpinfo.php`) leak significant internal detail.
- Manual testing should always precede and validate automated tool output (SQLMap results were cross-checked against manual boolean-based injection).
- Automation (Bash scripting) increases testing efficiency but also increases the importance of strict scope/authorisation controls.

---

## ⚖️ Disclaimer

This repository is submitted for academic/educational purposes only, as part of a supervised cybersecurity lab course. All testing was performed in an isolated virtual lab environment against intentionally vulnerable systems designed for this purpose. Do not run these tools or techniques against any system without explicit, documented authorisation.

