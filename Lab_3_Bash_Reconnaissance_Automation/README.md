## ⚙️ Lab 3 - Bash Reconnaissance Automation

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

